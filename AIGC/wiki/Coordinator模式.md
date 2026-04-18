---
title: Coordinator 模式（多智能体企业级编排）
tags: [Coordinator, 多智能体, 编排, Worker, Scratchpad, Claude Code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Coordinator 模式（多智能体企业级编排）

## 定义

Coordinator 模式是 Claude Code 的中心化多智能体编排方案，采用"协调者-工作者"架构：一个专职协调者（Coordinator）管理多个并行工作者（Worker）的生命周期和任务分配。协调者不直接执行代码，只负责分析、分配和综合；工作者拥有完整执行工具集，按协调者的指令实施具体操作。

## 核心观点

- 协调者的核心约束：**"理解"不能被委派**——不能简单转发工作者的发现，必须先消化再编写指令（来源：raw/御舆：解码 Agent Harness/article.md）
- Scratchpad 是多工作者之间传递**持久知识**的桥梁，弥补工作者间无法直接通信的限制
- 与 Fork 模式**互斥**：Coordinator 激活时 Fork 自动禁用
- 工作者默认使用 `acceptEdits` 权限模式，自动接受文件编辑，无需用户逐一确认

## 激活机制

双重门控：

1. **Feature gate**（编译时）：确定是否包含该功能代码
2. **环境变量**（运行时）：`CLAUDE_CODE_COORDINATOR_MODE` 显式启用

`matchSessionMode()` 函数处理会话恢复时的模式一致性——自动翻转环境变量以匹配原会话模式。

## 工具分工

```
Coordinator（协调者）— 管理权        Worker（工作者）— 执行权
─────────────────────────────        ───────────────────────
Agent Tool（创建/分配任务）           Read / Write / Edit / Bash
TaskStop Tool（停止工作者）           Grep / Glob / WebSearch
SendMessage Tool（向工作者发消息）    Skill / MCP 工具
Structured Output Tool               （不包含团队管理工具）
```

- **Simple 模式**（资源受限环境）：Worker 只有 Bash, Read, Edit 三个工具
- **Full 模式**（本地开发）：Worker 拥有完整白名单工具集

## 四阶段工作流

| 阶段 | 执行者 | 目的 | 典型输出 |
|------|--------|------|---------|
| Research | Workers（并行） | 调查代码库、发现文件 | Scratchpad 分析文档 |
| Synthesis | Coordinator | 阅读发现、编写实施规格 | 实施规格文档 |
| Implementation | Workers | 按规格进行精确修改 | 代码变更 |
| Verification | Workers | 测试修改是否正确 | 测试结果和问题列表 |

> "Never write 'based on your findings' or 'based on the research.' These phrases delegate understanding to the worker instead of doing it yourself. You never hand off understanding to another worker."

**并发策略**：
- 只读任务（Research）：自由并行
- 写密集任务（Implementation）：同一文件集一次一个，防止冲突
- Verification：有时可与 Implementation 并行（不同文件区域）

## Scratchpad 协作空间

- **物理位置**：`/tmp/claude-{uid}/{sanitized-cwd}/{sessionId}/scratchpad/`
- **设计原则**：
  1. 无权限提示，工作者自由读写
  2. 持久化跨工作者知识（消息传递是瞬时的，Scratchpad 是持久的）
  3. 会话隔离，避免跨会话污染
  4. 结构自由，系统不规定文件组织方式
- **典型模式**：Research 工作者写入分析 → Coordinator 读取综合 → 写入实施规格 → Implementation 工作者按规格执行 → Verification 工作者验证

### 置信度标注最佳实践

```
[HIGH] 确认文件路径和函数签名正确
[MEDIUM] 推测的依赖关系，需要实施时验证
[LOW] 未完全调查的区域，实施前需要额外研究
```

## 团队管理

### TeamCreateTool / TeamDeleteTool

创建流程：检查是否已在团队中 → 生成唯一团队名称（`team-{随机ID}`）→ 创建团队文件 → 更新全局状态 → 设置任务列表

删除安全保障：
1. 活跃成员检查（所有成员完成后才允许清理）
2. 清理顺序：团队目录 → worktree → 团队上下文
3. 错误容忍：单个清理失败不阻止其他资源清理

### SendMessage 寻址模式

| 模式 | 格式 | 范围 |
|------|------|------|
| 点对点 | `to: "agent-name"` | 单个工作者 |
| 广播 | `to: "*"` | 全体工作者 |
| 跨进程 | `to: "uds:<socket-path>"` | 不同 CLI 实例 |
| 跨会话 | `to: "bridge:<session-id>"` | 跨会话/远程 |

**停止-恢复模式**：向已停止的工作者发消息时自动重新激活，保留之前的分析状态，释放空闲期的 API 连接资源。

## 任务通知协议

工作者完成后以 XML 格式传递结果（XML 在 LLM 上下文中比 JSON 有更好的可识别性）：

```xml
<task-notification>
  <task-id>{agentId}</task-id>
  <status>completed|failed|killed</status>
  <summary>{human-readable status summary}</summary>
  <result>{agent's final text response}</result>
  <usage>
    <total_tokens>N</total_tokens>
    <tool_uses>N</tool_uses>
    <duration_ms>N</duration_ms>
  </usage>
</task-notification>
```

## 故障恢复

| 失败类型 | 应对策略 |
|---------|---------|
| 工具执行失败 | 分析原因，重试或调整策略 |
| maxTurns 耗尽 | 评估已完成部分，决定是否重新分配 |
| MCP 连接断开 | 降级到不依赖该工具的策略 |
| 上下文过长 | 压缩上下文或拆分为更小子任务 |

原则：**充分利用已完成的工作，而不是等待完美信息**。

## Coordinator 模式 vs Fork 模式

| 维度 | Coordinator 模式 | Fork 模式 |
|------|------------------|-----------|
| 架构 | 中心化（协调者-工作者） | 去中心化（对等并行） |
| 上下文共享 | 工作者只看到分配的任务 | 所有子智能体继承完整父上下文 |
| 通信 | 协调者中转、Scratchpad 共享 | 无直接通信，各自独立 |
| 缓存效率 | 不共享缓存前缀 | 字节级共享，高效缓存 |
| 适用场景 | 需要协调的复杂多步骤任务 | 独立的并行调查/搜索任务 |
| 故障恢复 | 协调者可以重新分配任务 | 主智能体收到失败通知后决定 |
| 心智模型 | 建筑工地（项目经理+工人） | 侦察队（多个独立侦察兵） |

## 反模式

- **"工作者检查工作者"**：导致信息链式传递，每次传递丢失细节
- **"转发理解"**：协调者直接将工作者发现转发给另一个工作者，导致格式不一致和信息缺失
- **提前删除团队**：Scratchpad 被清理、工作者文件修改丢失

## 相关概念

- [[多智能体协作五种模式]] — Anthropic 五种多智能体架构总览，Coordinator 对应"编排器与子智能体"模式
- [[SubAgent-与-Fork模式]] — Fork 模式详解，与 Coordinator 的互斥关系
- [[Tool-vs-Skill-vs-SubAgent选型]] — 三种能力单元的选型决策树
- [[Skill框架源码分析]] — Worker 可使用的 Skill 工具机制
