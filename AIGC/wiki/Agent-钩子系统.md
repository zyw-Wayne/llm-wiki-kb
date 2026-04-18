---
title: Agent 钩子系统
tags: [claude-code, hooks, 生命周期, 扩展点, 安全, 观察者模式]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Agent 钩子系统

## 定义

钩子系统是 Claude Code 的"神经系统"——为 Agent 整个生命周期提供精细的扩展点。权限管线决定 Agent **能否**执行某个操作，而钩子系统决定在执行操作**前后**会发生什么。它遵循经典的**观察者模式（Observer Pattern）** + **责任链模式（Chain of Responsibility）**。

## 五种钩子类型

| 类型 | 执行引擎 | 可持久化 | 延迟 | 典型场景 |
|------|---------|---------|------|---------|
| **Command** | Shell 命令 | 是 | 毫秒级 | 脚本检查、文件操作 |
| **Prompt** | LLM 推理（单轮） | 是 | 秒级 | 内容审核、语义判断 |
| **Agent** | LLM 多步推理 | 是 | 秒~分钟 | 测试验证、复杂审批 |
| **HTTP** | HTTP POST 请求 | 是 | 网络依赖 | CI 集成、审计日志 |
| **Function** | TypeScript 回调 | **否** | 毫秒级 | 运行时深度交互（SDK 模式） |

Function 钩子不可持久化的原因：TypeScript 回调无法序列化为 JSON 配置，这是"声明式配置"与"命令式代码"的边界。

### 同步 vs 异步执行模式

- **同步**（默认）：阻塞等待，适合"先审批再执行"
- **`async: true`**：后台运行，结果不可见，适合"发后即忘"的日志
- **`asyncRewake: true`**：后台运行，以退出码 2 结束时唤醒模型继续对话，适合长时间监控任务

## 26 个生命周期事件

事件按功能分为六层：

### 工具调用层（最常用）

| 事件 | 可阻止 | 核心用途 |
|------|-------|---------|
| **PreToolUse** | 是 | 拦截/修改工具输入，权限管线之后的第二道防线 |
| **PostToolUse** | 否 | 审计/后处理工具输出（MCP 输出可覆写） |
| **PostToolUseFailure** | 否 | 失败诊断（含 error_type/is_interrupt/is_timeout） |

PreToolUse 退出码语义：
- 退出码 0：静默通过
- 退出码 2：阻止调用并将 stderr 展示给模型
- 其他非 0：警告用户但不阻止

### 用户交互层

| 事件 | 可阻止 | 核心用途 |
|------|-------|---------|
| **UserPromptSubmit** | 是 | 修改/阻止用户消息，注入额外上下文 |
| **Notification** | 否 | 与外部通知系统集成 |

### 会话管理层

| 事件 | 可阻止 | 核心用途 |
|------|-------|---------|
| **SessionStart** | 否* | 环境初始化、上下文注入（来源：startup/resume/clear/compact） |
| **SessionEnd** | 否 | 清理工作（超时限制 1,500ms） |
| **Stop** | 是 | 强制继续/质量检查（退出码 2 注入 stderr 强制对话继续） |
| **StopFailure** | 否 | API 错误上报（即发即忘） |

*SessionStart 阻止错误被忽略（优雅降级原则）：核心功能不应被扩展逻辑劫持。

### 子代理层

| 事件 | 可阻止 | 核心用途 |
|------|-------|---------|
| **SubagentStart** | 否 | 子代理监控，stdout 展示给子代理 |
| **SubagentStop** | 是 | 结果验证，退出码 2 让子代理继续运行 |

### 压缩层

| 事件 | 可阻止 | 核心用途 |
|------|-------|---------|
| **PreCompact** | 是 | stdout 作为自定义压缩指令附加到压缩提示 |
| **PostCompact** | 否 | 压缩质量检查 |

### 其他事件

- **PermissionRequest**：权限对话框前自动审批（第二阶段决策）
- **PermissionDenied**：软拒绝策略，引导模型使用更安全方式
- **ConfigChange**：配置变更审计，可阻止恶意修改
- **Setup**：仓库初始化时执行环境检查
- **Elicitation / ElicitationResult**：MCP 服务器交互式认证监控
- **CwdChanged / FileChanged**：环境变更通知
- **InstructionsLoaded**：指令加载审计（仅可观测，不可阻止）

## 钩子响应协议（JSON）

钩子输出是结构化 JSON，通过 stdout 传递：

```json
{
  "decision": "approve | block",
  "reason": "阻止原因（block 时）",
  "additionalContext": "注入额外上下文",
  "continue": true,
  "stopReason": "停止原因",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "updatedInput": {},
    "permissionDecision": "allow | deny | ask"
  }
}
```

**双通道设计**：
- stdout 非 JSON 部分 → 展示给用户（退出码非 0 时）
- stdout JSON 部分 → 结构化控制
- stderr → 展示给模型（退出码 2 时）

### 退出码与 JSON 协作

| 退出码 | JSON decision | 最终效果 |
|--------|--------------|---------|
| 0 | approve 或无 | 正常通过 |
| 0 | block | 阻止（JSON 优先） |
| 2 | 任意 | 阻止，stderr 展示给模型 |
| 其他非 0 | approve | 警告但继续 |
| 其他非 0 | block | 阻止 |

**设计原则**：默认放行，显式阻止。JSON 格式错误或无 decision 字段时，系统默认继续执行，避免意外阻止正常操作。

### updatedInput：运行时修改工具输入

允许钩子在不改变用户意图的前提下修改实际参数。例如自动为 Bash 命令添加 `--dry-run`。

**透明原则**：修改应可预测、有文档记录、对用户可见，不应静默改变操作语义。

## 优先级排序

```
userSettings > projectSettings > localSettings > pluginHook > builtinHook > sessionHook
```

来自 `sortMatchersByPriority` 函数。用户主权原则：个人配置应能覆盖项目配置和插件默认值。

**多钩子执行规则**：所有匹配钩子按优先级依次执行，任意一个 block 决策生效后操作被阻止，但后续低优先级钩子仍会执行。

## 三层安全门禁

```
第一层：disableAllHooks: true → 全部禁用（终极开关）
第二层：allowManagedHooksOnly: true → 只运行企业策略钩子
第三层：工作区信任检查 → 未信任的工作区拒绝执行
```

与配置系统的 `projectSettings` 排除机制协同，防止克隆恶意仓库后自动执行恶意 hooks。

## 配置示例

### 企业级安全审计配置

```json
{
  "hooks": {
    "SessionStart": [{ "hooks": [{ "type": "command", "command": "python3 scripts/session_init.py" }] }],
    "PreToolUse": [
      { "matcher": "Bash(rm *)", "hooks": [{ "type": "command", "command": "validate_delete.py", "timeout": 3000 }] },
      { "matcher": "Write", "hooks": [{ "type": "prompt", "prompt": "Analyze write operation. $ARGUMENTS" }] }
    ],
    "PostToolUse": [{ "matcher": "Bash", "hooks": [{ "type": "http", "url": "https://audit.internal/log", "async": true }] }],
    "Stop": [{ "hooks": [{ "type": "command", "command": "check_completion.py", "asyncRewake": true }] }]
  }
}
```

## 会话钩子的性能优化

使用 `Map` 而非 `Record` 存储会话钩子：`Map.set()` 是 O(1) 且返回相同引用不触发监听器，避免 N 个并行 Agent 注册钩子时的 O(N²) 开销。

## 最佳实践与反模式

**最佳实践**：
- 使用 matcher 精确过滤，避免每次工具调用都执行钩子
- PostToolUse 尽量使用 async 模式
- SessionEnd 钩子保持轻量（1,500ms 超时限制）
- 始终显式设置 timeout

**反模式**：
- PreToolUse 中执行耗时操作（如 `npm run full-code-review`）
- 钩子之间的循环依赖
- 过度使用 updatedInput 静默修改操作语义
- 忘记设置超时导致无限阻塞

## 相关概念

- [[Agent-配置系统]] — 钩子配置遵循六层配置系统的合并规则，安全门禁与配置系统协同
- [[Context-压缩]] — PreCompact/PostCompact 钩子允许定制压缩策略
- [[Harness-Engineering]] — 钩子系统是 Harness 实施"机械化约束"的核心手段
- [[Claude-Code-系统化用法]] — 用法 07（post-tool hook 自动验证）和用法 09（AgentShield 安全规则）的底层机制
