---
title: Skill 框架源码分析
tags: [Skill, 框架架构, 源码分析, Anthropic, skill_run, 沙箱执行, Token优化]
sources: [raw/兄弟！你真的懂 Skill 吗？/article.md]
updated: 2026-04-13
---

# Skill 框架源码分析

## 定义

Anthropic Skill 框架的源码架构分析。通过逐文件拆解 9 个核心文件（约 2000+ 行代码），梳理了从 SKILL.md 声明到 LLM 发起 function_call 并执行的完整链路，归纳出 5 种执行模式。

## 核心结论

> Anthropic 对 Skill 系统的设计哲学，本质上是对 LLM 能力边界的深刻理解。他们没有把 Skill 做成"给 LLM 注册更多 API"的系统，而是做成了"教 LLM 更多知识"的系统。

### 关键发现：16 个官方 Skill 全部不用 function calling

- 16 个 Skill 中，没有一个使用 `Tools:` 声明来注册 function calling
- 全部通过 SKILL.md 文本内容 + `skill_run` 沙箱命令驱动执行
- 公式：**Skill 的执行力 = SKILL.md body 质量 × (Agent 基础工具 + skill_run 沙箱能力)**

## 核心架构链路

```
SKILL.md → FsSkillRepository（扫描、解析）
         → Skill 对象（name, description, body, tools, resources）
         → SkillToolSet（6 个管理工具 + skill_run）
         → DynamicSkillToolSet（按需加载业务工具）
         → SkillsRequestProcessor（注入 system prompt）
```

### LLM 使用 Skill 的四步流程

1. `skill_list()` → 看到所有技能的 name + description（~30 Token/个）
2. `skill_load("pdf")` → SKILL.md body 注入 system prompt
3. `skill_select_docs(docs=["forms.md"])` → 按需加载详细文档
4. `skill_run(command="python3 scripts/xxx.py")` → 在沙箱中执行命令

### CQRS 架构

所有 `skill_xxx` 工具只修改 `state_delta`（临时状态），真正内容注入在 `SkillsRequestProcessor` 中完成。写入方和读取方通过 state_delta 解耦。

## skill_run：核心执行引擎

`skill_run` 是整个系统最核心的工具，4 种模式都依赖它。

### 接口

- **输入**：技能名 + shell 命令
- **输出**：stdout + stderr + exit_code + 输出文件

### 沙箱执行 6 步流程

1. 定位技能目录（如 `/path/to/skills/pdf`）
2. 创建/获取隔离工作空间（`/tmp/ws_session123/`）
3. 增量暂存技能目录（基于目录哈希，不变则跳过）
4. 处理声明式输入文件
5. 执行命令（`bash -lc "python3 scripts/xxx.py"`，在隔离目录中执行）
6. 收集输出文件

### 自动注入的环境变量

| 变量 | 指向 |
|------|------|
| `$WORKSPACE_DIR` | 工作空间根目录 |
| `$SKILLS_DIR` | `$WORKSPACE_DIR/skills` |
| `$WORK_DIR` | 存放中间文件 |
| `$OUTPUT_DIR` | 存放最终输出 |
| `$RUN_DIR` | 运行记录 |
| `$SKILL_NAME` | 当前技能名 |

### 两种运行时

- **本地模式**（`LocalWorkspaceRuntime`）：开发环境，`bash -lc` 执行，秒开
- **容器模式**（`ContainerWorkspaceRuntime`）：生产环境，Docker 隔离，默认禁用网络

### 增量暂存设计（`_stage_skill`）

- `compute_dir_digest()` 计算技能目录哈希
- 哈希没变 → 直接跳过（零开销）
- 符号链接让脚本"无感知"地访问共享目录
- 只读保护防止篡改技能源文件

## Token 优化：三层信息模型

| 注入层级 | 触发条件 | 典型 Token 消耗 |
|---------|---------|----------------|
| L0: 概览 | 始终注入 | ~30 Token/技能 |
| L1: SKILL.md body | skill_load 后 | ~500-2000 Token |
| L2: 详细文档 | skill_select_docs 后 | ~1000-5000 Token |

### 与 function calling 的 Token 对比

- `skill_run` Schema：~20 Token
- 8 个独立 function calling：~1600-4000 Token
- 10 轮对话差距：1.6 万到 4 万 Token

## 五种执行模式

### 模式一：纯 Prompt 注入型

代表：`frontend-design`、`brand-guidelines`、`algorithmic-art`

- Skill = 一段精心编写的 system prompt
- 没有 scripts、没有 tools
- 全部价值：提供领域知识 + 约束行为 + 引导思维流程
- 框架映射：仅注入 body 到 system prompt，无 skill_run 调用

### 模式二：脚本执行型

代表：`pdf`、`pptx`、`xlsx`、`webapp-testing`

- SKILL.md 当教程，scripts/ 当工具箱
- SKILL.md 中的代码示例教 LLM 写代码，预制脚本给 skill_run 执行
- LLM 角色：调用者（跑预制脚本）
- 适合：复杂、需要验证的工作流

### 模式三：库调用型

代表：`slack-gif-creator`

- Skill 自带 Python 库（`core/` 而非 `scripts/`）
- LLM 自己写脚本 import 库函数
- LLM 角色：开发者（组合库函数写新代码）
- 关键区别：模式二是执行预制脚本，模式三是 LLM 现场编写脚本

### 模式四：参考文档渐进加载型

代表：`pptx`、`mcp-builder`

- SKILL.md 是路由表，详细文档按需加载
- 三层信息漏斗：description → SKILL.md body → 详细文档
- Token 节省约 46%

### 模式五：编排型

代表：`skill-creator`

- SKILL.md 是完整的多阶段流水线（32KB，CI/CD 式）
- 流程：Capture Intent → Interview → Write → Test → Evaluate → Improve → Package
- 同时用到：纯 Prompt + skill_run + skill_select_docs + 子 Agent
- 本质：用 SKILL.md 编排 LLM 执行复杂项目，可做 SOP

### 五种模式统一光谱

```
◄── 轻量 ────────────────────────────────── 重量 ──►
纯 Prompt → 参考文档渐进加载 → 库调用 → 脚本执行 → 编排
```

| 框架机制 | 模式一 | 模式二 | 模式三 | 模式四 | 模式五 |
|---------|-------|-------|-------|-------|-------|
| skill_load → body 注入 | ✅ | ✅ | ✅ | ✅ | ✅ |
| skill_select_docs | ❌ | ✅ | ❌ | ✅ | ✅ |
| skill_run (预制脚本) | ❌ | ✅ | ❌ | ✅ | ✅ |
| skill_run (LLM 写的脚本) | ❌ | ❌ | ✅ | ❌ | ❌ |
| Tools: 声明 | ❌ | ❌ | ❌ | ❌ | ❌ |
| 子 Agent 编排 | ❌ | ❌ | ❌ | ❌ | ✅ |

## 三层信息注入闭环

1. **工具 Schema 注入** → LLM 知道有 skill_run 这个工具
2. **System Prompt 行为指引** → LLM 知道加载技能后用 skill_run 执行
3. **SKILL.md body 注入** → LLM 知道 command 参数具体怎么填

## 框架工程设计亮点

- **CQRS 解耦**：写入方（_tools.py）和读取方（_skill_processor.py）通过 state_delta 解耦
- **漏斗式加载**：三层信息模型，逐步注入，Token 效率最大化
- **声明式绑定**：工具名字符串匹配而非 import 引用，松耦合
- **统一 IR**：FunctionDeclaration 作为中间表示，屏蔽不同 LLM API 的差异

## 给 Skill 开发者的实操建议

| 场景 | 推荐模式 |
|------|---------|
| 教 LLM 遵循某种规范/风格 | 纯 Prompt 注入 |
| 需要 LLM 操作特定文件格式 | 脚本执行型 |
| 需要 LLM 灵活组合 API | 库调用型 |
| 知识量大，不同任务需要不同文档 | 渐进加载型 |
| 需要 LLM 执行复杂多步骤工作流 | 编排型 |

> 万变不离其宗：写好 SKILL.md 是一切的基础。

## 相关概念

- [[Skill定义规范]] — Skill 的定义、SKILL.md 标准写法与安全规范
- [[Skill实战踩坑指南]] — Anthropic 数百个 Skill 实战踩坑经验，9 类分类与九大改进原则
- [[Skills-Github化]] — 将 GitHub 开源项目 Skill 化的实战方法
- [[Skills-多Skill协作]] — 多个 Skill 如何协同工作
- [[代码执行范式]] — Agent 代码执行新范式，Skill 框架是其落地实现
- [[Skill-vs-Prompt]] — Skill 与 Prompt 的详细对比与选型指南
- [[AI编程行为准则]] — 行为约束型 Skill 的实践（CLAUDE.md 模式）

## 来源

- [兄弟！你真的懂 Skill 吗？](../raw/兄弟！你真的懂 Skill 吗？/article.md) — 李小宇，腾讯云开发者，2026-03-17
