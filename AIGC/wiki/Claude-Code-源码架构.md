---
title: Claude Code 源码架构
tags: [claude-code, architecture, agent-harness, 源码分析]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Claude Code 源码架构

## 定义

Claude Code 是基于 Agent Harness 模式构建的 AI 编程助手，其核心是一个将 LLM 推理能力与工具执行结合为迭代循环的运行时框架。架构分为四层，通过数据驱动的消息数组模型驱动整体执行。

## 四层分层架构

```
表现层 → 编排层 → 能力层 → 基础设施层
```

| 层次 | 核心模块 | 职责 |
|------|---------|------|
| 表现层（Presentation） | CLI 入口、REPL 渲染（Ink/React） | 用户输入捕获、终端输出渲染、快捷键处理 |
| 编排层（Orchestration） | 对话主循环、状态管理（AppState） | turn 调度枢纽、响应式状态订阅 |
| 能力层（Capability） | 工具实现、子智能体系统、技能系统 | 具体能力单元、递归组合、领域扩展 |
| 基础设施层（Infrastructure） | 查询引擎、Shell 执行、MCP 集成、IDE 桥接、设置系统、记忆系统、钩子系统 | 最底层技术能力，不含业务逻辑 |

## 核心模块索引

| 模块名称 | 职责描述 | 关键数据结构 |
|---------|---------|------------|
| CLI 入口与 REPL | 命令行入口，参数解析、REPL 交互循环初始化 | 命令行参数对象、REPL 渲染状态树 |
| 对话主循环 | turn-based 对话循环，消息收发与工具调度核心调度器 | messages 数组（Message[]）、turn 计数器、stop reason |
| 查询引擎 | 封装 API 调用、流式响应处理、重试逻辑 | API 请求配置、流式响应块（ContentBlock[]）、缓存断点标记 |
| 工具类型系统 | 工具泛型接口定义、执行上下文与工具工厂函数 | Tool interface、Zod 输入 schema |
| 工具注册表 | 全局工具注册、发现与组装，内置工具与 MCP 工具合并 | 工具映射表（Map<name, Tool>） |
| 子智能体系统 | 子智能体生成、恢复、fork 及内置智能体定义 | 子智能体配置、fork 状态快照 |
| Shell 执行引擎 | Shell 命令权限校验、只读检测与安全沙箱执行 | 命令执行请求、stdout/stderr、退出状态码 |
| 上下文压缩 | auto-compact、micro-compact、session memory 等多层压缩策略 | 压缩摘要消息、token 使用量计数 |
| 钩子系统 | Pre/Post tool use hooks、session hooks 注册与执行 | 钩子注册表、钩子执行上下文 |
| 设置系统 | 全局/项目/本地三级配置、权限规则与 schema 验证 | 三级配置对象（Settings）、权限规则数组 |
| 记忆系统 | CLAUDE.md 发现、嵌套记忆、团队记忆与相关性匹配 | 记忆文件、记忆层级树、相关性匹配评分 |
| 技能系统 | 内置技能管理、动态加载与 slash command 注册 | 技能注册表、slash command 映射 |
| MCP 集成 | MCP 连接管理、协议适配、资源读写与权限通道 | MCP 连接配置、资源描述符、工具能力声明 |
| IDE 桥接 | VSCode/JetBrains 双向通信、JWT 认证与远程会话管理 | 桥接消息队列、JWT token |
| 协调器模式 | 多智能体协调场景下的 worker 分配与结果汇总 | worker 注册表、任务队列 |
| 状态管理 | 全局应用状态 store、React 集成与 selector 模式 | 全局状态树（AppState）、selector 函数 |

## 核心执行循环（数据驱动模型）

Claude Code 的执行本质是**消息数组不断追加**驱动的循环，而非命令式流程控制：

```
用户输入
  → 追加到 messages[]
  → 查询引擎 API 调用（组装 system prompt + messages + tools）
  → 流式响应（text / tool_use）
  → 工具执行（PreToolUse 钩子 → 实际执行 → PostToolUse 钩子）
  → tool_result 追加到 messages[]
  → 检查 stop_reason：end_turn 则结束，tool_use 则继续循环
```

**横切关注点**：
- **钩子系统**：工具调用前后插入拦截逻辑，不影响核心路径
- **权限系统**：贯穿所有工具执行的安全护栏
- **压缩系统**：上下文接近 token 上限时自动触发
- **记忆系统**：每轮开始时扫描 CLAUDE.md 并注入系统提示

## 关键数据流路径

### 标准工具调用流程（10 步）
1. 用户输入
2. processUserMessage() — 追加消息 + 注入记忆
3. queryAPI() — 组装请求，调用 Anthropic Messages API（流式）
4. 流式响应处理 — 渲染到 REPL
5. 工具路由 — 查找工具实例 / Zod 验证 / 权限检查
6. 工具执行 — Pre 钩子 → 实际执行 → Post 钩子
7. 结果回填 — tool_result 追加到 messages[]，大结果持久化到磁盘
8. 循环判断 — stop_reason === end_turn?
9. turn 结束 — 输出文本，检查是否需要 auto-compact
10. 等待下一轮用户输入

### 权限判定路径（5 层）
1. Zod schema 输入验证
2. 工具级 checkPermissions() — 返回 allow/deny/ask
3. PreToolUse 钩子 — 可修改参数或阻止
4. 通用权限系统 — alwaysAllow / alwaysDeny / alwaysAsk
5. 分类器辅助决策（Transcript Classifier / Bash Classifier）

**权限模式**：

| 模式 | 策略 | 适用场景 |
|------|------|---------|
| ask（默认） | 所有写操作需用户确认 | 最安全 |
| auto-edit | 文件编辑自动放行 | 平衡安全与效率 |
| full-auto | 所有操作自动执行 | 最流畅，风险最高 |
| plan | 仅允许只读工具 | 安全规划模式 |

### 上下文压缩触发路径

压缩策略从轻到重：

| 策略 | 标志 | 压缩力度 | 缓存友好 |
|------|------|---------|---------|
| micro-compact | `CACHED_MICROCOMPACT` | 轻度 | 高 |
| auto-compact | `REACTIVE_COMPACT` | 中度 | 中 |
| history snip | `HISTORY_SNIP` | 中高度 | 低 |
| context collapse | `CONTEXT_COLLAPSE` | 极高 | 低 |

### 子智能体 Fork 执行路径

1. 父智能体通过 AgentTool 触发 fork
2. 复制消息历史 + 继承权限规则，子智能体获得独立 messages 数组
3. 递归调用对话主循环，支持嵌套 fork（受深度限制）
4. 子智能体结果回填到父消息数组
5. 父智能体从 fork 点继续执行

## 模块接口契约

### 工具标准协议（所有工具必须实现）

| 方法 | 作用 |
|------|------|
| `isEnabled()` | 工具是否在当前上下文中启用（默认 true） |
| `isReadOnly()` | 是否仅读取，不修改状态（默认 false） |
| `isConcurrencySafe()` | 是否可安全并行执行（默认 false） |
| `isDestructive()` | 是否执行不可逆操作（默认 false） |
| `checkPermissions()` | 权限检查，返回 allow/deny/ask |
| `toAutoClassifierInput()` | 生成自动分类特征描述 |
| `userFacingName()` | 返回用户友好名称 |

### 钩子系统生命周期扩展点

- **PreToolUse**：工具执行前触发，可修改输入或阻止执行
- **PostToolUse**：工具执行后触发，可处理结果或记录日志
- **Session Hooks**：会话级事件（开始、结束）触发

## 架构设计模式速览

| 设计模式 | 应用位置 |
|---------|---------|
| Agent Loop | 对话主循环 |
| Factory Method | 工具类型系统（buildTool） |
| Registry | 工具注册表、技能注册表 |
| Plugin | MCP 集成、技能系统 |
| Observer | 钩子系统 |
| Strategy | 上下文压缩（多策略可互换） |
| Layered Architecture | 整体系统四层 |
| Async Generator | 查询引擎、流式输出 |
| Hierarchical Agent | 子智能体系统（分形结构） |
| Feature Flag | 全系统（编译时死代码消除） |

## 模块耦合度分析

| 耦合关系 | 程度 | 说明 |
|---------|------|------|
| 对话主循环 ↔ 查询引擎 | 紧耦合 | 共享消息数组数据模型 |
| 对话主循环 ↔ 工具注册表 | 紧耦合 | 工具路由嵌入主循环 |
| 工具注册表 ↔ MCP 集成 | 松耦合 | MCP 工具通过动态注册机制接入 |
| 子智能体 ↔ 对话主循环 | 递归耦合 | 子智能体递归调用主循环，形成分形结构 |
| 钩子系统 ↔ 工具执行 | 事件耦合 | 通过生命周期事件触发，不影响核心路径 |
| 记忆系统 ↔ 查询引擎 | 松耦合 | 记忆内容作为系统提示一部分注入 |

## 相关概念

- [[MCP]] — Model Context Protocol 集成详解
- [[Claude-Code-系统化用法]] — 使用层面的实践指南
- [[Tool-vs-Skill-vs-SubAgent选型]] — 能力层三类扩展的选型决策
- [[Context-压缩]] — 上下文压缩策略深度分析
- [[Harness-Engineering]] — Agent Harness 工程化方法论
- [[Claude-Code-功能标志与术语表]] — 完整功能标志速查和术语定义
