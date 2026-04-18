---
title: Agent 记忆系统
tags: [agent, memory, memdir, 长期记忆, 缓存, fork模式, claude-code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Agent 记忆系统

## 定义

Claude Code 的记忆系统（memdir）是一个基于文件的、类型化的、跨会话持久的记忆架构。它对应人类记忆结构中的"长期记忆"——不同于上下文窗口的"工作记忆"，memdir 跨会话持久保存不可从代码推导的关键知识。

## 核心原则

**只保存不可从当前项目状态推导的信息。** 代码模式、架构、文件路径、Git 历史都可以通过工具实时获取，不属于记忆范畴。记忆应专注于：人的偏好、决策背景、外部链接——存在于人的大脑或外部系统中、无法通过读取代码仓库获得的信息。

## 四种记忆类型

闭合四类型系统（不可扩展，约束换来高效的一致性推理）：

| 类型 | 存储内容 | 触发时机 |
|------|---------|---------|
| **user** | 用户角色、目标、知识背景 | 了解到用户画像时 |
| **feedback** | 用户对 Agent 行为的纠正和确认 | 用户说"不要那样"或确认非显而易见的做法成功时 |
| **project** | 项目的非代码状态（决策、截止日期、进行中的工作） | 了解到谁在做什么、为什么、何时完成时 |
| **reference** | 外部系统指针（Linear、Grafana、Slack 频道） | 了解到外部系统的资源和用途时 |

### feedback 记忆的结构要求

不仅记录失败（纠正），也记录成功（确认）。格式：规则本身 + **Why: 原因** + **How to apply: 适用场景**

### project 记忆的特殊规则

相对日期必须转换为绝对日期（"周四" → "2026-03-05"），因为记忆跨会话持久，相对日期会失去意义。

### 明确排除的内容

- 代码模式、架构、文件路径——可通过阅读代码推导
- Git 历史——`git log`/`git blame` 是权威来源
- 调试解决方案——修复已在代码中，上下文在 commit message 中
- CLAUDE.md 中已有的文档
- 临时任务细节

## 记忆文件格式

### Markdown + Frontmatter

每条记忆是独立的 Markdown 文件：

```markdown
---
name: pre-commit-lint-requirement
description: Must run npm run lint before every commit; CI failed for a full day
type: feedback
---

**Rule**: Run `npm run lint` before every code commit.
**Why**: Unlinted code caused CI to fail for an entire day.
**How to apply**: Before git commit, always run lint first and fix errors.
```

**使用文件而非数据库的理由**：可读性、版本控制、可移植性、可调试性、无依赖成本。

### MEMORY.md 索引文件

入口点，不是记忆本身而是索引。每次对话开始时自动加载。

双重容量保护：
- **行数限制（200行）**：保护理解效率，确保索引是"快速浏览"工具
- **字节限制（25KB）**：保护上下文预算

每条条目格式：`- [Title](file.md) -- 一行钩子描述`（不超过 150 字符）

### 目录结构

默认路径：`~/.claude/projects/<sanitized-git-root>/memory/`

路径解析优先级：
1. `CLAUDE_COWORK_MEMORY_PATH_OVERRIDE` 环境变量
2. `autoMemoryDirectory` 设置（**仅限 policySettings/localSettings/userSettings，排除 projectSettings**）
3. 默认路径

路径安全校验拒绝：相对路径、根路径、Windows 盘符根、UNC 路径、Null 字节注入、用户目录外的路径。

## 自动记忆提取（Fork 模式）

### 完美分叉架构

后台 Agent 通过 `runForkedAgent` 在对话结束后异步运行，与主对话共享：
- 完全相同的系统提示（system prompt）
- 完全相同的工具集（保持缓存 key 一致）
- **提示缓存（prompt cache）共享**——节省约 96.7% 的提取成本

### 互斥机制

主 Agent 和后台 Agent 的记忆写入互斥：
- 主 Agent 本次对话已写入记忆 → 后台 Agent 跳过提取
- 主 Agent 未写入记忆 → 后台 Agent 正常执行提取

防止重复提取导致的冗余记忆。

### 工具权限白名单（最小权限原则）

| 工具 | 权限 |
|------|------|
| Read / Grep / Glob | 不受限制（只读） |
| Bash | 仅限只读命令（ls、find 等） |
| Edit / Write | **仅限记忆目录内的路径** |
| 其他所有工具 | 拒绝 |

### 节流与尾随提取（Trailing Extraction）

- **节流**：每经过若干轮次后才触发一次提取，提高每次提取的信息密度
- **尾随提取**：提取进行中又有新对话完成时，新上下文被暂存，当前提取完成后执行尾随提取（跳过节流计数器），确保不遗漏任何对话

### 启用条件

1. `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 环境变量不为 true
2. 非 `--bare` 模式
3. 有持久存储（CCR 场景需要 `CLAUDE_CODE_REMOTE_MEMORY_DIR`）
4. `settings.json` 中 `autoMemoryEnabled` 不为 false
5. 通过 GrowthBook 功能门控

## 缓存感知架构

### 工具列表一致性

缓存共享的前提：工具列表是 API 缓存 key 的一部分。forked Agent 使用与主 Agent **相同的工具列表**，权限通过 `canUseTool` 运行时回调过滤，而非编译时不同的工具列表。

**架构原则：接口一致，行为可变。**

### 缓存节省计算

```
无缓存：发送 80,000 tokens → $0.24
有缓存：复用缓存前缀 → $0.008
节省：96.7%
```

## 记忆的信任层次

**记忆是快照而非事实**，推荐使用 Level 1（线索信任）：

- Level 0（完全不信任）：过于保守，记忆失去价值
- **Level 1（线索信任）**：记忆指出方向，独立验证当前状态 ← Claude Code 采用此层次
- Level 2（事实信任）：过于激进，可能被过时记忆误导

验证规则：
- 记忆命名了文件路径 → 检查文件是否存在
- 记忆命名了函数或标志 → grep 查找
- 用户即将根据建议行动 → 先验证

## 常见误区

- **误区一：记忆越多越好** → 定期审查，自问"删除后行为会有实质性不同吗？"
- **误区二：把记忆当作文档系统** → 技术文档放 `docs/`，只有决策的"为什么"放记忆
- **误区三：忽略相对日期问题** → 所有时间记忆必须使用绝对日期

## 相关概念

- [[Agent-配置系统]] — 记忆路径的安全策略与配置系统的 projectSettings 排除机制一脉相承
- [[Context-压缩]] — 压缩与记忆共享"只保留不可重新获取的信息"的哲学
- [[上下文管理策略]] — 记忆系统是上下文管理的重要补充，压缩后通过记忆恢复关键上下文
- [[Harness-Engineering]] — 持久化记忆是 Harness 四大支柱之三
