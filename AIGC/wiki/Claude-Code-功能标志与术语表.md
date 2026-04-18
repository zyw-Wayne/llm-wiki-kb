---
title: Claude Code 功能标志与术语表
tags: [claude-code, feature-flag, tools, glossary, 速查]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Claude Code 功能标志与术语表

## 定义

本页综合附录 B（工具完整清单）、附录 C（功能标志速查）和附录 D（术语表），作为 Claude Code 系统的结构化参考索引。功能标志是编译时通过 Bun define 注入的布尔开关，控制特定功能代码是否包含在最终产物中（false 分支通过死代码消除被整段移除）。

---

## 一、工具完整清单

### 工具权限模型说明

| 字段 | 含义 |
|------|------|
| readOnly | 仅执行读取操作，不修改文件系统或外部状态 |
| destructive | 执行不可逆操作（删除、覆盖、发送） |
| concurrencySafe | 可安全并行执行，不依赖共享可变状态 |

工具均通过 `buildTool` 工厂函数创建，未显式覆盖的方法默认值：`isEnabled=true`、`isReadOnly=false`、`isConcurrencySafe=false`、`isDestructive=false`。

### 1. 文件操作

| 工具 | readOnly | concurrencySafe | 说明 |
|------|----------|-----------------|------|
| FileReadTool | true | true | 读取文件，支持行号范围、PDF 分页、图片、notebook |
| FileWriteTool | false | false | 写入文件，覆盖已有或创建新文件 |
| FileEditTool | false | false | 精确字符串替换编辑，支持单处/全局替换 |
| NotebookEditTool | false | false | 编辑 Jupyter Notebook 单元格（替换/插入/删除） |

**典型工作流**：FileReadTool → 分析 → FileEditTool/FileWriteTool

### 2. 搜索

| 工具 | readOnly | concurrencySafe | 说明 |
|------|----------|-----------------|------|
| GlobTool | true | true | 文件名 glob 模式匹配，按修改时间排序 |
| GrepTool | true | true | 基于 ripgrep 的正则内容搜索，支持 files_with_matches/content/count 三种输出 |
| ToolSearchTool | false | false | 工具发现搜索，按关键词匹配延迟加载的工具（启用条件：`ToolSearch` 标志） |

### 3. 执行

| 工具 | readOnly | concurrencySafe | 说明 |
|------|----------|-----------------|------|
| BashTool | 动态 | 动态 | Shell 命令执行，只读命令可并行，写命令串行 |
| PowerShellTool | 动态 | 动态 | Windows PowerShell 执行（仅 Windows 环境启用） |

**BashTool 特殊机制**：内置只读命令清单（ls/cat/git status 等）；长时间命令有超时机制；输出超限时持久化到磁盘。

### 4. 网络

| 工具 | readOnly | concurrencySafe | 说明 |
|------|----------|-----------------|------|
| WebFetchTool | true | true | 抓取 URL 内容并转换为 markdown |
| WebSearchTool | true | true | 执行网络搜索，返回结构化搜索结果 |

**组合模式**：WebSearchTool 广度搜索 → WebFetchTool 深度获取具体页面。

### 5. 智能体

| 工具 | 说明 |
|------|------|
| AgentTool | 启动子智能体（内置/自定义），支持 fork、resume、后台执行 |
| SendMessageTool | 向其他智能体或频道发送消息（纯文本消息为 readOnly） |
| TeamCreateTool | 创建多智能体团队（启用条件：`Agent Swarms` 模式） |
| TeamDeleteTool | 删除已创建的团队 |

### 6. 任务管理（启用条件：`Todo V2`）

| 工具 | readOnly | 说明 |
|------|----------|------|
| TaskCreateTool | false | 创建后台任务 |
| TaskGetTool | true | 获取单个任务详情 |
| TaskUpdateTool | false | 更新任务状态/内容 |
| TaskListTool | true | 列出所有任务 |
| TaskOutputTool | 动态 | 获取任务输出流 |
| TaskStopTool | false | 停止正在运行的任务 |

### 7. 计划模式

| 工具 | 说明 |
|------|------|
| EnterPlanModeTool | 进入计划模式，限制为只读工具集 |
| ExitPlanModeV2Tool | 退出计划模式，恢复正常工具权限 |

### 8. 工作树（启用条件：`Worktree Mode`）

| 工具 | 说明 |
|------|------|
| EnterWorktreeTool | 创建 git worktree 并切换工作目录 |
| ExitWorktreeTool | 退出 worktree，保留或移除工作目录 |

### 9. 调度（启用条件：`AGENT_TRIGGERS`）

| 工具 | 说明 |
|------|------|
| CronCreateTool | 创建 cron 定时任务 |
| CronDeleteTool | 删除 cron 定时任务 |
| CronListTool | 列出所有 cron 定时任务 |
| RemoteTriggerTool | 远程触发器管理（启用条件：`AGENT_TRIGGERS_REMOTE`） |

### 10. 交互

| 工具 | 说明 |
|------|------|
| AskUserQuestionTool | 向用户提问并等待回复（readOnly） |
| SkillTool | 调用已注册的 slash command 技能 |
| ConfigTool | 运行时配置查看/修改（仅 ant 构建） |

### 11. MCP 资源

| 工具 | 说明 |
|------|------|
| ListMcpResourcesTool | 列出 MCP 服务器提供的资源（readOnly） |
| ReadMcpResourceTool | 读取 MCP 服务器上的特定资源（readOnly） |

### 12. 其他（条件启用）

| 工具 | 启用条件 | 说明 |
|------|---------|------|
| TodoWriteTool | 始终 | 待办事项面板写入，结果不渲染到 transcript |
| LSPTool | `ENABLE_LSP_TOOL` | LSP 语言服务协议操作 |
| SleepTool | `PROACTIVE`/`KAIROS` | 延时等待 |
| MonitorTool | `MONITOR_TOOL` | 监控后台任务输出 |
| WorkflowTool | `WORKFLOW_SCRIPTS` | 工作流脚本执行 |
| WebBrowserTool | `WEB_BROWSER_TOOL` | 浏览器工具 |
| SnipTool | `HISTORY_SNIP` | 历史消息裁剪 |
| CtxInspectTool | `CONTEXT_COLLAPSE` | 上下文检查器 |
| TerminalCaptureTool | `TERMINAL_PANEL` | 终端截图捕获 |
| VerifyPlanExecutionTool | `CLAUDE_CODE_VERIFY_PLAN` | 计划执行验证 |
| ListPeersTool | `UDS_INBOX` | 列出对等智能体 |
| SuggestBackgroundPRTool | ant 构建 | 建议后台 PR 创建 |
| SendUserFileTool | `KAIROS` | 向用户发送文件 |
| PushNotificationTool | `KAIROS` | 推送通知 |
| SubscribePRTool | `KAIROS_GITHUB_WEBHOOKS` | PR Webhook 订阅 |

---

## 二、功能标志速查

> 功能标志类型：**编译时** = 打包阶段求值，false 分支完全移除；**编译时+运行时门控** = 编译时为 true，但运行时还需额外条件激活。

### C.1 核心交互模式（12 个）

| 标志 | 类型 | 作用 |
|------|------|------|
| `PROACTIVE` | 编译时 | 主动模式，智能体可在用户空闲时主动建议 |
| `KAIROS` | 编译时+运行时 | 助手模式（IDE 协作完整功能集），含频道、会话恢复等 |
| `KAIROS_BRIEF` | 编译时 | KAIROS 精简子集，仅会话恢复等基础能力 |
| `KAIROS_CHANNELS` | 编译时+运行时 | 独立频道消息通信功能 |
| `KAIROS_DREAM` | 编译时 | 加载额外 Dream 技能集 |
| `KAIROS_GITHUB_WEBHOOKS` | 编译时 | GitHub Webhook 集成，接收 GitHub 事件推送 |
| `KAIROS_PUSH_NOTIFICATION` | 编译时 | KAIROS 推送通知功能 |
| `BRIDGE_MODE` | 编译时+运行时 | IDE 桥接模式，与 VS Code/JetBrains 双向通信 |
| `COORDINATOR_MODE` | 编译时+运行时 | 协调器模式，多智能体任务分发与汇总 |
| `VOICE_MODE` | 编译时 | 语音输入/输出、Push-to-Talk 支持 |
| `DAEMON` | 编译时 | 守护进程模式，支持长期运行 |
| `BUDDY` | 编译时 | 伴侣精灵动画角色，情感化反馈 |

### C.2 智能体与子任务（9 个）

| 标志 | 作用 |
|------|------|
| `FORK_SUBAGENT` | Fork 子智能体，创建独立子智能体处理子任务 |
| `AGENT_TRIGGERS` | 智能体触发器，定时任务调度（cron） |
| `AGENT_TRIGGERS_REMOTE` | 远程智能体触发器（依赖 `AGENT_TRIGGERS`） |
| `ULTRAPLAN` | 超级规划模式，交互式计划审查界面 |
| `VERIFICATION_AGENT` | 任务完成后自动启动验证流程 |
| `BUILTIN_EXPLORE_PLAN_AGENTS` | 内置 ExploreAgent/PlanAgent（依赖 `FORK_SUBAGENT`） |
| `AGENT_MEMORY_SNAPSHOT` | 智能体状态快照与跨会话恢复 |
| `WORKFLOW_SCRIPTS` | 工作流脚本，本地多步骤任务编排 |
| `TEMPLATES` | 作业分类器，识别并路由不同类型请求 |

### C.3 上下文管理与压缩（8 个）

| 标志 | 压缩力度 | 作用 |
|------|---------|------|
| `CACHED_MICROCOMPACT` | 轻度 | 缓存式微压缩，维护 Prompt Cache 边界 |
| `REACTIVE_COMPACT` | 中度 | 响应式压缩，接近阈值自动触发 |
| `HISTORY_SNIP` | 中高度 | 历史裁剪，智能裁剪已处理对话历史 |
| `CONTEXT_COLLAPSE` | 极高 | 上下文折叠，激进缩减策略 |
| `TOKEN_BUDGET` | — | Token 预算管理，追踪与预警 |
| `COMPACTION_REMINDERS` | — | 压缩前后向用户展示提醒 |
| `PROMPT_CACHE_BREAK_DETECTION` | — | 检测压缩操作中的缓存断裂 |
| `COMMIT_ATTRIBUTION` | — | 为压缩后提交添加上下文归属标记 |

**推荐配置**：`REACTIVE_COMPACT` + `CACHED_MICROCOMPACT` + `TOKEN_BUDGET`

### C.4 权限与安全（5 个）

| 标志 | 作用 |
|------|------|
| `TRANSCRIPT_CLASSIFIER` | 转录分类器，基于对话内容自动判断权限模式 |
| `BASH_CLASSIFIER` | Bash 分类器，对命令进行安全性分类，安全命令自动放行 |
| `HARD_FAIL` | 硬失败模式，关键错误时直接终止 |
| `NATIVE_CLIENT_ATTESTATION` | 原生客户端认证，平台原生身份验证 |
| `ANTI_DISTILLATION_CC` | 反蒸馏保护，防止模型输出被蒸馏攻击 |

**安全级别推荐**：
- 企业环境：全部 5 个标志
- 标准安全：`TRANSCRIPT_CLASSIFIER` + `BASH_CLASSIFIER`
- 开发调试：`HARD_FAIL`

### C.5 工具与技能（14 个）

| 标志 | 作用 |
|------|------|
| `MONITOR_TOOL` | MonitorTool，后台任务监控 |
| `WEB_BROWSER_TOOL` | WebBrowserTool，内置浏览器面板 |
| `MCP_SKILLS` | 从 MCP 服务器动态发现和加载技能 |
| `EXPERIMENTAL_SKILL_SEARCH` | 语义技能索引和搜索 |
| `SKILL_IMPROVEMENT` | 已安装技能自动优化 |
| `RUN_SKILL_GENERATOR` | 动态生成新技能 |
| `BUILDING_CLAUDE_APPS` | Claude 应用构建专用技能集 |
| `REVIEW_ARTIFACT` | 代码审阅专用技能集 |
| `HOOK_PROMPTS` | 钩子注入自定义提示词到对话流 |
| `TREE_SITTER_BASH` | Tree-sitter 对 Bash 命令进行 AST 解析 |
| `TREE_SITTER_BASH_SHADOW` | Tree-sitter 影子模式，并行对比验证 |
| `UDS_INBOX` | Unix Domain Socket 收件箱 |
| `MCP_RICH_OUTPUT` | MCP 工具返回结构化富媒体内容 |
| `CONNECTOR_TEXT` | 支持连接器文本块类型渲染 |

### C.6–C.12 其他标志分类

| 分类 | 数量 | 代表标志 |
|------|------|---------|
| 会话与持久化 | 4 | `BG_SESSIONS`, `AWAY_SUMMARY`, `FILE_PERSISTENCE`, `NEW_INIT` |
| 记忆与知识管理 | 4 | `TEAMMEM`, `EXTRACT_MEMORIES`, `LODESTONE`, `MEMORY_SHAPE_TELEMETRY` |
| 远程与连接 | 8 | `SSH_REMOTE`, `DIRECT_CONNECT`, `CCR_AUTO_CONNECT`, `SELF_HOSTED_RUNNER`, `BYOC_ENVIRONMENT_RUNNER` |
| UI 与界面 | 7 | `MESSAGE_ACTIONS`, `TERMINAL_PANEL`, `QUICK_SEARCH`, `HISTORY_PICKER`, `AUTO_THEME`, `NATIVE_CLIPBOARD_IMAGE` |
| 设置同步 | 2 | `UPLOAD_USER_SETTINGS`, `DOWNLOAD_USER_SETTINGS` |
| 遥测与诊断 | 6 | `PERFETTO_TRACING`, `SHOT_STATS`, `SLOW_OPERATION_LOGGING`, `DUMP_SYSTEM_PROMPT` |
| 基础设施与构建 | 10 | `UNATTENDED_RETRY`, `ULTRATHINK`, `TORCH`, `DUMP_SYSTEM_PROMPT`, `OVERFLOW_TEST_TOOL` |

**总计：89 个功能标志**

### 常见场景配置推荐

| 场景 | 核心标志组合 |
|------|------------|
| 日常开发 | `BASH_CLASSIFIER` + `TRANSCRIPT_CLASSIFIER` + `REACTIVE_COMPACT` + `TOKEN_BUDGET` |
| IDE 集成 | `BRIDGE_MODE` + `KAIROS` + `KAIROS_CHANNELS` + `BASH_CLASSIFIER` |
| 多智能体协作 | `COORDINATOR_MODE` + `FORK_SUBAGENT` + `BUILTIN_EXPLORE_PLAN_AGENTS` + `VERIFICATION_AGENT` |
| CI/CD 自动化 | `DAEMON` + `AGENT_TRIGGERS` + `UNATTENDED_RETRY` + `HARD_FAIL` |
| 企业安全 | `HARD_FAIL` + `TRANSCRIPT_CLASSIFIER` + `BASH_CLASSIFIER` + `NATIVE_CLIENT_ATTESTATION` |
| 性能调试 | `PERFETTO_TRACING` + `SHOT_STATS` + `SLOW_OPERATION_LOGGING` + `DUMP_SYSTEM_PROMPT` |

---

## 三、核心术语表（精选）

> 完整术语约 100 条，按字母排序。以下收录高频/核心术语。

| 术语 | 中文 | 简明定义 |
|------|------|---------|
| Agent | 智能体 | 具备自主规划、工具调用和迭代执行能力的 AI 程序实体 |
| Agent Harness | 智能体线束 | 包裹 LLM 的运行时基础设施层，编排工具调度、权限管控、上下文管理等 |
| API Turn | API 轮次 | 一次完整的"发送请求-接收响应"周期，一个用户请求可触发多个 API Turn |
| Async Generator | 异步生成器 | `async function*` 函数，通过 yield 逐步产出流式结果，是流式输出核心原语 |
| Auto Compact | 自动压缩 | 上下文接近 Token 阈值时自动触发的压缩机制 |
| Bash Classifier | Bash 分类器 | 对 Bash 命令进行安全性 ML 分类，高置信度安全命令自动放行 |
| Bridge Mode | 桥接模式 | Claude Code 与 IDE 之间的双向通信协议，支持权限回调和状态同步 |
| Build Tool (buildTool) | 工具工厂函数 | 创建工具实例的统一工厂函数，为未显式定义的方法填充安全默认值 |
| CLAUDE.md | CLAUDE 记忆文件 | 记忆系统核心载体，支持全局/项目/目录级嵌套，每轮对话注入系统提示 |
| Compaction | 上下文压缩 | 将对话历史压缩为精简摘要以释放 Token 空间，有四种策略 |
| Concurrency Safe | 并发安全 | 工具属性，标记后可被流式工具执行器同时并行调度 |
| Context Collapse | 上下文折叠 | 最激进的压缩策略，有专门折叠 UI，用户可选择恢复 |
| Context Window | 上下文窗口 | 模型单次请求可处理的最大 Token 数量，所有上下文管理的根本约束 |
| Coordinator Mode | 协调器模式 | 多智能体协作中的中枢，负责任务分发、进度汇总和结果整合 |
| Dead Code Elimination | 死代码消除 | 编译器移除永远不会执行的代码路径，Feature Flag false 分支通过此机制移除 |
| Dynamic Tool Registration | 动态工具注册 | MCP 工具在运行时被发现、适配并注册到工具注册表的过程 |
| Extended Thinking | 扩展思考 | 模型在生成回复前的内部推理过程，由 `ULTRATHINK` 标志控制 |
| Feature Flag | 功能标志 | 编译时布尔开关，false 时守卫的代码分支被整段移除 |
| Fork Mode | Fork 模式 | 通过 fork 机制创建独立子进程/子智能体处理子任务 |
| Hook | 钩子 | 在工具执行前后、会话生命周期等关键节点触发的用户自定义回调 |
| Ink | Ink 框架 | 基于 React 的终端 UI 渲染框架，将 React 组件树渲染为终端文本输出 |
| KAIROS | 助手模式 | 面向 IDE 集成的完整协作功能集（频道通信、会话恢复、GitHub Webhook 等） |
| LLM | 大语言模型 | Claude 等大规模预训练语言模型，是智能体的推理核心 |
| Main Loop | 主循环 | Claude Code 核心执行循环，接收输入→调用 API→处理工具→渲染输出的迭代流程 |
| MCP | 模型上下文协议 | Anthropic 定义的标准化协议，允许外部工具服务器向模型提供工具和上下文 |
| Memory | 记忆 | 持久化存储的用户偏好、项目知识，核心载体是 CLAUDE.md |
| Memory Injection | 记忆注入 | 每轮对话开始时将 CLAUDE.md 内容注入系统提示的过程 |
| Micro Compact | 微压缩 | 轻量级上下文缩减策略，配合 Cached Micro Compact 可维护缓存边界 |
| Multi-tier Settings | 三级配置 | 全局（~/.claude）、项目（.claude/）、本地（.local.json）三级层叠配置 |
| Permission Mode | 权限模式 | 控制智能体自主执行级别：ask / auto-edit / full-auto / plan |
| Permission Pipeline | 权限管线 | 工具执行前的多层权限检查管道，包含分类器、用户确认、自动放行规则 |
| Plan Mode | Plan 模式 | 进入后仅允许只读工具，用于安全规划和风险评估 |
| Prompt Cache | 提示缓存 | Anthropic API 对重复出现的系统提示前缀进行缓存，降低延迟和成本 |
| Query Engine | 查询引擎 | 管理与 Anthropic Messages API 所有通信的底层引擎，含重试和流式解析 |
| REPL | 交互式循环 | Claude Code 主界面，持续接收输入、执行、渲染，基于 Ink 框架 |
| Runtime Gate | 运行时门控 | 编译时为 true 但运行时还需额外条件才真正激活的标志类型 |
| Skill | 技能 | 可安装的扩展能力包，通过提示词模板和工具定义扩展特定领域能力，以 slash command 触发 |
| Slash Command | 斜杠命令 | 以 `/` 前缀触发的技能调用方式，如 `/commit`、`/review-pr` |
| Stop Reason | 停止原因 | 模型响应中标识结束原因：`end_turn`（正常结束）或 `tool_use`（需执行工具） |
| Streaming Tool Executor | 流式工具执行器 | 以流式方式执行工具并实时渲染，支持并发安全工具的并行调度 |
| Subagent | 子智能体 | 由主智能体 fork 的独立执行单元，拥有独立上下文，递归调用 Main Loop |
| System Prompt | 系统提示词 | 每次 API 请求开头的指令文本，定义行为规范和可用工具；记忆内容在此注入 |
| Team Memory | 团队记忆 | 面向团队的共享记忆文件系统，由 `TEAMMEM` 标志控制 |
| Token | 词元 | 模型处理文本的基本单位，也是上下文窗口容量和 API 计费的核心度量 |
| Token Budget | Token 预算 | 对单次会话可用 Token 进行规划和监控的管理机制 |
| Tool | 工具 | 智能体可调用的外部能力单元，通过标准化接口注册执行 |
| Tool Interface | 工具接口 | 所有工具必须遵循的标准协议（isEnabled/isReadOnly/checkPermissions 等） |
| Tool Registry | 工具注册表 | 全局工具注册与组装中心，内置工具与 MCP 工具合并，内置工具同名优先 |
| Transcript Classifier | 转录分类器 | 基于对话内容自动判断权限模式的分类模型，由 `TRANSCRIPT_CLASSIFIER` 控制 |
| Tree-sitter | Tree-sitter 解析器 | 增量式语法分析框架，对 Bash 命令进行 AST 级别的安全分析 |
| Turn | 轮次 | 对话主循环的一次完整迭代，从发送消息到接收完整响应 |
| Ultraplan | 超级规划 | 提供交互式计划审查界面，执行前允许用户审批和修改方案 |
| Ultrathink | 深度思考 | 启用扩展思考能力的标志，允许模型更深入推理 |
| Worktree | 工作树 | Git worktree 封装，为并行任务创建隔离工作目录 |
| Zod Schema | Zod 验证模式 | 用于工具输入参数运行时类型校验和自动补全提示的验证模式 |

---

## 相关概念

- [[Claude-Code-源码架构]] — 架构模块、数据流路径和设计模式的完整分析
- [[Claude-Code-系统化用法]] — 使用层面的实践指南
- [[MCP]] — Model Context Protocol 集成详解
- [[Tool-vs-Skill-vs-SubAgent选型]] — 工具/技能/子智能体的选型决策
- [[Context-压缩]] — 上下文压缩策略深度分析
- [[Harness-Engineering]] — Agent Harness 工程化方法论
