---
title: Agent 权限管线四阶段
tags: [agent, 权限, 安全, 纵深防御, 权限模式, BashTool, Claude Code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Agent 权限管线四阶段

## 定义

Claude Code 的权限检查是一条由四个阶段组成的管线（Pipeline），而非单一的布尔判断。每个阶段有独立职责和短路逻辑——只要前一阶段做出终局决定，后续阶段便不再执行。这是"纵深防御（Defense in Depth）"思想在软件系统中的体现。

## 核心观点

- 权限系统不是附加的安全层，而是内嵌到架构核心管线中（来源：raw/御舆：解码 Agent Harness/article.md）
- **优先级铁律**：deny 始终优先于 allow，无论来源如何——显式拒绝的力量大于显式允许
- PermissionContext 的不可变性确保并发安全：多个工具同时检查权限时各自读取确定性快照

## 详细内容

### 四阶段管线

```
工具调用请求
  ↓
阶段一：validateInput（Zod Schema 验证）
  失败 → 降级为 ask（优雅降级，不直接崩溃）
  通过 ↓
阶段二：规则匹配（deny > ask > allow 优先级铁律）
  deny → 拒绝（不可覆盖）
  allow → 放行
  ask 或无匹配 ↓
阶段三：checkPermissions（工具特定的上下文评估）
  deny/allow → 终局决定
  passthrough/ask ↓
阶段四：交互式提示
  Hook 脚本 → allow/deny（最高优先级）
  AI 分类器 → allow/deny（竞赛机制，2 秒超时）
  用户确认 → allow/deny/allow-once
```

### 阶段一：validateInput — Zod Schema 验证

在权限检查之前先验证输入数据的合法性。解析失败时系统优雅降级为请求用户确认，而非直接崩溃。

工程原则：**在安全系统中，错误处理应该是"安全的"而非"正确的"。**

### 阶段二：hasPermissionsToUseTool — 规则匹配

按严格优先级检查三类规则：deny > ask > allow。

**七种规则来源（就近原则，从低到高优先级）：**

```
userSettings（最低）
  → projectSettings
  → localSettings（不提交 Git）
  → flagSettings
  → policySettings
  → cliArg
  → command
  → session（最高优先级）
```

匹配逻辑：
- 精确工具名匹配
- MCP 服务器级通配符（如 `mcp__server1__*` 匹配该服务器所有工具）

### 阶段三：checkPermissions — 上下文评估

每个工具可实现自己的 `checkPermissions` 方法，进行更精细的上下文评估（如 BashTool 会解析命令、检查路径安全性、匹配前缀规则）。

返回四种行为：
- `allow`：直接放行（可携带 `updatedInput`）
- `deny`：拒绝执行
- `ask`：需要用户确认
- `passthrough`：交由后续阶段决定（最终变为 `ask`）

`passthrough` vs `ask` 的区别：passthrough 表示"我没有意见，交给后续决定"，如果后续 allow 规则匹配可被覆盖；ask 表示"我认为需要用户确认"，不会被 allow 规则覆盖。

### 阶段四：交互式提示 — 三方竞赛机制

触发 `ask` 状态后，三个决策者竞赛：

| 决策者 | 信任等级 | 适用场景 |
|-------|---------|---------|
| Hook 脚本 | 最高 | CI/CD 自定义逻辑，代表系统管理员意图 |
| 用户 | 中等 | 手动选择，代表当前操作者意图 |
| AI 分类器 | 最低 | auto 模式下自动判断，可能出错 |

**ResolveOnce 原子竞争**：使用 `claim()` 原子操作解决竞争条件（JavaScript 单线程模型下的轻量级互斥锁），"先到先得"，防止多个异步回调同时 resolve。

分类器有 2 秒超时——"尽力而为"策略，不成为用户体验瓶颈。

### PermissionContext 设计

`ToolPermissionContext` 是权限系统的核心数据结构：
- 所有字段均为 `readonly`（完全不可变）
- 包含权限模式、各级规则（allow/deny/ask）、bypass 标志等
- 规则按来源索引，可精确追踪"此规则来自哪里"

每次权限更新产生新的 Context 对象，而非修改现有对象——确保并发场景下每个权限检查使用确定性快照。

### 五种权限模式谱系

```
严格 ←────────────────────────────→ 宽松

default    plan    auto    bypassPermissions
 逐次确认  只读为主  AI分类器   完全跳过（仍受deny规则约束）

（内部模式）bubble：子智能体权限冒泡回主智能体
```

**default 模式**：每次工具调用需用户确认。适用：日常交互式开发，需要完全掌控 Agent 行为。

**plan 模式**：写入类工具（Edit、Write）被 deny，读取类工具正常放行。适用：代码审查、架构分析（先看后改的两阶段工作流）。

**auto 模式**：使用 AI 分类器（"YOLO classifier"）代替人工审批。
- acceptEdits 快速路径：先检查 acceptEdits 模式是否通过，通过则直接放行（短路优化）
- 安全工具白名单：Read、Grep、Glob、TodoWrite 等跳过分类器检查
- 拒绝追踪：分类器连续拒绝多次后自动回退到交互式提示（熔断器）
- 某些安全检查类型（如 `.git/`、`.claude/` 目录操作）是"分类器免疫"的

**bypassPermissions 模式**：完全跳过权限检查，但仍受以下防线约束：
- 步骤 1a 的 deny 规则
- `requiresUserInteraction` 检查
- 内容级 ask 规则
- `safetyCheck`

适用：CI/CD 管道、容器隔离的自动化环境。

**bubble 模式（内部）**：子智能体权限"冒泡"回主智能体上下文，确保子智能体不获得超出主智能体的权限。

### BashTool 精细权限控制

Shell 命令的组合性和表达力远超其他工具，需要语义级别的权限控制。

**三种规则匹配格式（表达能力递增）：**

| 格式 | 示例 | 说明 |
|-----|------|------|
| 精确匹配 | `Bash(npm test)` | 仅匹配这一条特定命令 |
| 前缀匹配 | `Bash(npm:*)` | 匹配所有 npm 开头的命令 |
| 通配符匹配 | `Bash(git commit *)` | 匹配 git commit 后跟任意参数 |

优先使用前缀匹配（`:*` 语法）——语义更清晰、不容易出错。

分类器投机性审批：有 2 秒超时的 Promise 竞赛，高置信度匹配则自动批准。

### 权限更新双层机制

`applyPermissionUpdates`（同步内存应用）+ `persistPermissionUpdates`（异步文件持久化）分离：

- 内存应用：即时生效，影响当前行为
- 文件持久化：重启后生效，支持 local/user/project 三个目标
- session/cliArg 级别：仅运行时生效，不支持持久化

支持六种操作 × 五种配置源 = 30 种可能的更新操作。

### 企业级配置最佳实践

```json
// .claude/settings.json（提交到 Git，团队共享）
{
  "permissions": {
    "allow": ["Bash(npm test)", "Bash(git:*)", "Read", "Glob", "Grep"],
    "deny": ["Bash(npm publish)", "Bash(rm -rf *)"]
  }
}
```

策略：先用宽泛的 allow 规则授权基本操作，再用精确的 deny 规则排除危险操作（利用 deny 优先于 allow 的铁律）。

个人偏好用 `.claude/settings.local.json`（gitignore，不提交），团队共享用 `.claude/settings.json`（提交到版本控制）。

## 相关概念

- [[Agent-Harness-设计哲学五大原则]] — 原则二（安全边界内嵌）的源头
- [[工具系统五要素协议]] — 工具五要素中权限模型三层检查的调用方
- [[对话循环-AsyncGenerator]] — 权限管线被调用的对话主循环上下文
- [[Harness-Engineering]] — 权限管线是 Harness 四大支柱中"结构化执行"的安全保障
