---
title: Claude Code 会话管理实战
tags: [claude-code, 会话管理, context, compact, rewind, clear, subagent, 1M上下文, 最佳实践]
sources: [raw/Using Claude Code session management and 1M context/article.md]
updated: 2026-04-17
---

# Claude Code 会话管理实战

## 定义

Claude Code 官方团队总结的会话管理实战指南，覆盖上下文窗口、压缩、回退、子代理等五种上下文管理工具的使用时机和决策框架。作者 Thariq Shihipar（Anthropic 工程师，Claude Code 团队成员）。

## 核心观点

- **每个 turn 都是分支点**：每次 Claude 完成任务后，有 Continue / Rewind / Clear / Compact / Subagent 五种选择，而不仅是 Continue
- **新任务 = 新会话**：基本规则——开始新任务时应启动新会话，尽管 1M 上下文可以容纳更长对话
- **Rewind 优于纠正**：Claude 走错方向时，回退到出错前重新 prompt 比追加"那不对，改用 X"更有效
- **主动 Compact 优于被动 AutoCompact**：在 70% 时主动 `/compact` 附带指令，比等 95% 自动触发效果更好——关键原因：**模型在 context rot 最严重时执行压缩，正好是最"不聪明"的时刻**
- **Subagent 的判断标准**：`will I need this tool output again, or just the conclusion?`（我还需要这些工具输出吗，还是只需要结论？）

## 上下文腐烂与压缩的悖论

文章揭示了一个关键矛盾：

> Bad compacts happen when the model can't predict the direction your work is going.
> 特别困难之处在于：由于 context rot，模型在执行压缩时正处于最不智能的状态。

这意味着：
- 自动压缩在最需要智能判断的时刻（压缩时），恰恰是模型能力最弱的时刻
- 1M 上下文窗口给了更多余裕来**主动**压缩——不用等到 95% 才动手

## 五种上下文管理工具

| 工具 | 触发方式 | 本质 | 适用场景 |
|------|---------|------|---------|
| Continue | 发送新消息 | 继续当前会话 | 同一任务，上下文仍然有效 |
| `/rewind` (Esc Esc) | 双击 Esc | 回退到之前消息，丢弃后续 | Claude 走错方向，想用新指令重试 |
| `/compact <hint>` | 手动触发 | LLM 摘要对话历史，继续工作 | 会话臃肿但仍在同一任务中 |
| `/clear` | 手动触发 | 用户自己写下要点，全新会话 | 开始全新任务 |
| Subagent | 自动/显式指令 | 子代理独立上下文，只返回结论 | 下一步会产生大量中间输出 |

## 场景决策表（官方速查）

| 场景 | 选择 | 理由 |
|------|------|------|
| 同一任务，上下文仍有效 | Continue | 窗口内所有信息都在支撑当前工作，不要花钱重建 |
| Claude 走错方向 | Rewind (Esc-Esc) | 保留有用的文件读取，丢弃失败尝试，用学到的教训重新 prompt |
| 任务中但会话因调试/探索臃肿 | `/compact <hint>` | 低成本，Claude 决定哪些信息重要，必要时用指令引导 |
| 开始全新任务 | `/clear` | 零腐烂，你完全控制哪些信息延续 |
| 下一步会产生大量中间输出 | Subagent | 中间工具噪音留在子代理上下文，只拿结论回来 |

## Rewind 的高级用法

- **"summarize from here"**：让 Claude 在回退点总结学到的教训，创建交接消息——像"从未来的自己给过去的自己写信"
- **回退后重 prompt**：不要说"那不对，试试 X"，而是说"Don't use approach A, the foo module doesn't expose that—go straight to B"——直接给出排除原因和正确路径

## Compact vs Clear 深度对比

| 维度 | `/compact` | `/clear` |
|------|-----------|---------|
| 谁做摘要 | Claude（模型） | 你（用户） |
| 信息损耗 | 有损，模型决定保留什么 | 无损（保留你写的部分） |
| 操作成本 | 低（一键触发） | 高（需要自己写要点） |
| 可引导性 | 可附带指令（`/compact focus on auth refactor`） | 完全控制 |
| 适用场景 | 任务进行中，想清理但不想重写 | 全新任务，想精确控制延续内容 |

## Subagent 实战指令示例

- "Spin up a subagent to verify the result of this work based on the following spec file"
- "Spin off a subagent to read through this other codebase and summarize how it implemented the auth flow, then implement it yourself in the same way"
- "Spin off a subagent to write the docs on this feature based on my git changes"

## 与已有知识的关系

本文从**用户视角**讲会话管理策略，与源码级的 [[上下文四级压缩策略]] 互补：
- 本文：用户在每个 turn 该做什么选择（Continue/Rewind/Clear/Compact/Subagent）
- 四级压缩：引擎盖下的自动压缩机制（Snip/MicroCompact/Collapse/AutoCompact）

两篇文章联合阅读可以获得完整的上下文管理心智模型：用户操作层 + 引擎实现层。

## 相关概念

- [[上下文管理策略]] — 三大传统策略（滑动窗口/摘要压缩/RAG）的理论框架
- [[上下文四级压缩策略]] — Claude Code 源码级的四级渐进压缩实现
- [[Context-Rot]] — 上下文腐烂现象的实验研究（本文反复引用的核心问题）
- [[Context-Engineering]] — 更宏观的 Context Engineering 方法论
- [[SubAgent-与-Fork模式]] — Subagent 的架构实现细节
- [[Claude-Code-系统化用法]] — Claude Code 进阶用法全集
- [[Plan模式与结构化工作流]] — Plan 模式的权限切换与工作流

## 来源

- [Using Claude Code: session management and 1M context](../raw/Using%20Claude%20Code%20session%20management%20and%201M%20context/article.md) — Anthropic 官方博客，Thariq Shihipar，2026-04-15
