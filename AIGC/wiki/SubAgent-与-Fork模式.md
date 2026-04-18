---
title: SubAgent 与 Fork 模式
tags: [SubAgent, Fork, AgentTool, 并行执行, PromptCache, Claude Code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# SubAgent 与 Fork 模式

## 定义

SubAgent（子智能体）是主智能体通过 AgentTool 派生的专用执行单元，拥有独立的工具集、系统提示和生命周期。Fork 模式是一种特殊的子智能体并行执行策略，通过字节级共享父对话上下文来最大化 Prompt Cache 命中率，从而大幅降低并行任务的 token 消耗。

## 核心观点

- Fork 模式的核心是**字节级继承**：子智能体必须与父智能体共享完全相同的 API 请求前缀才能命中 prompt cache（来源：raw/御舆：解码 Agent Harness/article.md）
- 内置智能体体系覆盖软件工程四种工作模式：探索（Explore）、规划（Plan）、执行（General）、验证（Verification）
- Verification Agent 采用**对抗性验证哲学**：目标是尽可能破坏被验证的代码，而非确认它能工作
- 工具隔离三层防线：全局禁止列表 → 异步白名单 → filterToolsForAgent 过滤

## AgentTool 架构

### BaseAgentDefinition 四大分类

| 类别 | 字段 |
|------|------|
| 身份与描述 | agentType, whenToUse, color |
| 工具与能力 | allowedTools/disallowedTools, skills, mcpServers |
| 执行控制 | model/effort, maxTurns, permissionMode/background |
| 上下文管理 | omitClaudeMd/isolationMode, hooks, memory |

### 三种智能体来源（优先级升序）

1. **BuiltInAgentDefinition**（内置）：通过 `getSystemPrompt()` 动态生成系统提示
2. **CustomAgentDefinition**（自定义）：用户通过 `.claude/agents/` 下的 Markdown 文件声明式定义
3. **PluginAgentDefinition**（插件）：插件提供，携带版本信息和依赖声明

加载优先级：内置 < 插件 < 用户设置 < 项目设置 < Flag 设置 < **策略设置**（企业最高优先级）

## 四种内置智能体

### Explore Agent（只读代码探索）

- 严格只读约束：双重锁——提示层面的软约束 + 工具层面（disallowedTools）的硬约束
- `omitClaudeMd: true`：省略 CLAUDE.md 降低噪声，节省约 5-15 Gtoken/周
- 可选 haiku 模型降低成本
- 适用场景：代码考古、依赖分析、架构探索、知识检索

### Plan Agent（结构化规划）

- 同样禁止修改类工具，但承担软件架构师角色
- 省略 CLAUDE.md 是为了让 Plan Agent 关注"做什么"而非"怎么做"
- 输出要素：问题分析、实施步骤、关键文件、风险评估、依赖关系

### General Purpose Agent（通用执行）

- 使用通配符 `'*'` 允许所有工具（实际受全局过滤函数约束）
- 设计哲学：**默认信任，边界后移**，灵活性最大化

### Verification Agent（对抗性验证）

- 使用红色 UI 标识，始终后台运行，禁止修改项目文件
- 警惕两种失败模式：
  1. **验证回避**：找借口不运行测试
  2. **前 80% 的表面正确性陷阱**：忽略边界条件和并发场景
- 借鉴软件工程**红队测试（Red Teaming）**理念
- 后台运行原因：验证可以等待，释放主线程，强制独立工作

## Fork 模式详解

### 类比 Unix fork()

| 维度 | Unix fork() | Claude Code Fork 模式 |
|------|-------------|----------------------|
| 继承内容 | 内存映像、文件描述符 | 系统提示、工具定义、对话历史 |
| 资源节省 | Copy-on-Write 内存共享 | Prompt Cache token 共享 |
| 递归防护 | 进程层级限制 | querySource 检查 |

### 激活条件

Feature gate 开启 + 非 Coordinator 模式 + 非非交互模式（Coordinator 激活时自动禁用 Fork）

### CacheSafeParams 五个维度

缓存命中需要以下五维度完全一致：

1. `systemPrompt`：系统提示必须字节一致
2. `userContext`：用户上下文（CLAUDE.md 等）
3. `systemContext`：系统注入的上下文信息
4. `toolUseContext`：工具定义 + 模型选择
5. `forkContextMessages`：对话历史前缀消息

> ⚠️ 任何一个维度的偏离都会导致整个缓存前缀失效。这就是为什么 Fork 路径优先传递已渲染的原始字节而非重新构建。

### buildForkedMessages 核心机制

- 所有 Fork 子智能体共享固定占位符：`"Fork started -- processing in background"`
- 消息结构：`[...history, assistant(all_tool_uses), user(placeholder_results..., directive)]`
- 只有最后的指令文本不同 → 最大化缓存命中面积

### 缓存效率示例

假设父对话 50,000 token，assistant 消息 2,000 token，系统提示 10,000 token：

- **不使用 Fork**：3 个子智能体 × 62,000 = 186,000 input token
- **使用 Fork**：62,600 input token（其中 62,000 命中缓存）
- **节省比例：约 66%**

### 递归 Fork 防护（双重检测）

1. **querySource 检查**（主防线）：设置在上下文选项上，不受自动压缩影响
2. **消息扫描**（后备）：检测特定标签，应对边界情况

## 自定义智能体

### 目录结构

`.claude/agents/` 下的 Markdown 文件，使用 YAML frontmatter 配置：

```markdown
---
name: security-auditor
description: 分析代码中的安全漏洞
tools: [Bash, Read, Grep, Glob]
disallowedTools: [Write]
model: haiku
effort: high
permissionMode: default
maxTurns: 30
color: orange
background: true
memory: project
skills: [/commit]
mcpServers: [slack]
hooks:
  PreToolUse:
    - matcher: Bash
      command: "audit-log.sh"
---
系统提示正文...
```

### maxTurns 经验值

| 任务类型 | 推荐轮次 |
|---------|---------|
| 只读搜索 | 10-15 |
| 代码生成 | 20-30 |
| 复杂重构 | 30-50 |
| 探索性任务 | 15-20 |

## 工具隔离三层防线

| 层级 | 规则 |
|------|------|
| 第一层：全局禁止 | Agent, ExitPlanMode, TaskOutput, TaskStop, AskUserQuestion |
| 第二层：异步白名单 | Read/Write/Edit/Bash/Grep/Glob/WebSearch 等核心工具 |
| 第三层：filterToolsForAgent | MCP 工具始终可用；全局禁止一律排除；进程内队友部分豁免 |

> MCP 工具始终可用的原因：MCP 工具经过用户授权，安全控制交给 MCP 权限系统处理。

## 相关概念

- [[Coordinator模式]] — Fork 的替代编排模式，两者互斥
- [[Tool-vs-Skill-vs-SubAgent选型]] — 选型决策树
- [[MCP]] — MCP 工具在子智能体中的使用
- [[Skill框架源码分析]] — Skill 在智能体中的预加载机制
