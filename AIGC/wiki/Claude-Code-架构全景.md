---
title: Claude Code 架构全景
tags: [Claude Code, 架构, 技术栈, TypeScript, Bun, React-Ink, Zod, 模块]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Claude Code 架构全景

## 定义

Claude Code 是 Anthropic 于 2025 年发布的终端原生 AI 编程 Agent，超过 50 万行 TypeScript 代码（1884 个 TS 文件，512,664 行），是 Agent Harness 架构的完整工程实现参考。不依赖编辑器，直接运行在命令行中。

## 核心观点

- Claude Code 的规模不亚于一个完整的编辑器框架（VS Code 核心约 50 万行 TypeScript）（来源：raw/御舆：解码 Agent Harness/article.md）
- 代码组织表现出高度模块化：每个工具是独立模块，每个子系统有清晰边界，遵循单一职责原则
- 架构的代表性：涵盖 Agent Harness 所有核心子系统，理解 Claude Code 即建立可迁移到任何 Agent 框架的心智模型

## 技术栈选型

| 技术组件 | 选择 | 设计考量 | 为何不选其他 |
|---------|------|---------|------------|
| **运行时** | Bun | 原生 TS 支持、更快启动、原生 fetch | Node.js 需编译步骤，Deno 生态成熟度不足 |
| **终端 UI** | React + Ink | 组件化 UI 模型、声明式渲染、React 生态复用 | blessed/ncurses 过于底层，原始 console.log 无法处理复杂布局 |
| **CLI 框架** | Commander.js | 成熟的命令行参数解析、子命令支持 | yargs 更重，oclif 面向大型 CLI 项目 |
| **Schema 验证** | Zod v4 | 运行时类型安全、工具输入校验、JSON Schema 生成 | Joi 不支持类型推导，io-ts 学习曲线陡峭 |
| **LLM SDK** | @anthropic-ai/sdk | Anthropic 官方 SDK、流式响应支持 | 直接 fetch 缺少类型安全和重试逻辑 |

React + Ink 的选择体现了设计理念：**终端 UI 不应该比 Web UI 低一等**。进度条、权限确认对话框、多列结果展示背后都是 React 组件。

Zod v4 承担双重职责：运行时参数校验 + 生成 JSON Schema 发给 API（"类型定义即文档"）。

## 五个核心模块

### 1. 入口点模块

三个核心职责：
- **启动优化**：性能探针最先执行，MDM 配置和 macOS Keychain 并行预取（将串行 I/O 重叠执行）
- **CLI 解析**：Commander.js 处理模型选择、允许工具列表、权限模式等参数
- **React/Ink 初始化**：创建渲染上下文，挂载根组件，启动交互式 REPL

工程原则：**启动路径是用户体验的第一印象**——通过并行预取、延迟加载和性能探针，将启动时间压缩到用户感知不到的水平。

### 2. LLM 查询引擎（QueryEngine）

`QueryEngine` class 管理对话生命周期和会话状态：对话消息历史、文件缓存、用量统计、权限拒绝记录等。

同时服务于两种运行模式：
- **交互式 REPL**：终端中的人机交互
- **无头 SDK**：通过 API 调用

"一个核心、多种入口"设计模式：避免两种模式走不同的核心路径，确保 bug 修复和功能添加只需在一个地方完成。

### 3. 异步生成器对话主循环

Claude Code 最精巧的模块，详见 [[对话循环-AsyncGenerator]]。

每轮迭代的七步流程：
1. 构建 API 请求（系统 Prompt + 消息历史 + 工具定义）
2. 调用 LLM API 并流式接收响应
3. 解析响应中的工具调用指令
4. 通过权限管线校验每个工具调用
5. 执行被允许的工具调用
6. 将工具结果注入消息历史
7. 决定继续循环（有新工具调用）或终止（纯文本回复）

### 4. 工具类型系统基石

定义所有工具必须遵循的类型契约，详见 [[工具系统五要素协议]]。

关键设计信息：
- 执行方法接收权限检查回调（权限检查内嵌到工具执行流程）
- 进度回调支持增量进度报告（流式 UI 基础）
- 并发安全声明影响调度策略
- 中断行为定义用户 Ctrl+C 时的行为
- 破坏性标记标识不可逆操作

### 5. 工具注册中心（getAllBaseTools）

工具系统的"单一事实来源"，三个设计细节：

1. **条件注册**：Feature Flag 控制，REPL 工具只在内部版本可用，定时任务工具只在特定模式启用
2. **延迟加载**：动态导入避免循环依赖 + 减少启动时间
3. **工具过滤**：发送给 LLM 之前根据权限上下文过滤，模型无法"看到"不应该使用的工具

## 架构层次

```
用户入口（CLI / SDK）
    ↓
QueryEngine（生命周期管理）
    ↓
对话主循环（AsyncGenerator 心脏）
    ↓           ↓
工具系统      权限管线
（双手）      （护栏）
    ↓           ↓
工具注册中心  PermissionContext
    ↓
扩展层（MCP / Plugin / Skill）
```

对话主循环是全系统的"枢纽"——连接工具系统、权限管线、上下文管理和子智能体调度。

## 六层配置源（优先级从低到高）

```
pluginSettings
  → userSettings（用户全局）
  → projectSettings（项目共享，入 Git）
  → localSettings（个人偏好，不入 Git）
  → flagSettings（CLI 标志，一次性覆盖）
  → policySettings（企业策略，最高优先级）
```

"地层沉积"模型：后加载者覆盖前者，使用深度合并策略配合自定义规则。

## Claude Code 意外公开事件

2026 年 3 月 31 日，安全研究员 Chaofan Shou 发现 npm 包 `@anthropic-ai/claude-code` 包含 source map 文件，推文获超过 1700 万次浏览。Anthropic 随后修补配置问题。这场讨论让 Agent Harness 从冷门工程概念变成整个开发者社区都关注的话题。

## 相关概念

- [[AI编程范式三次浪潮]] — Claude Code 所处的第三次浪潮背景
- [[对话循环-AsyncGenerator]] — 核心模块 3 的深度解析
- [[工具系统五要素协议]] — 核心模块 4/5 的深度解析
- [[Agent-权限管线四阶段]] — 对话循环中嵌入的安全护栏
- [[Agent-Harness-设计哲学五大原则]] — 贯穿整个架构的五大设计原则
- [[MCP]] — 工具系统渐进式扩展层的标准协议
