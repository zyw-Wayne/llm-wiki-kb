---
title: Plan 模式与结构化工作流
tags: [plan-mode, workflow, 结构化执行, 调度系统, fork-agent, 权限控制, 规划执行分离]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Plan 模式与结构化工作流

## 定义

Plan 模式是 Claude Code 的工作流控制机制，将 Agent 行为分为**只读探索阶段（规划）**和**可写执行阶段（实施）**两个严格分离的阶段。核心理念：在动手之前先对齐意图，避免方向性错误导致的返工。Plan 模式解决的核心问题是**"过早行动"（Premature Action）**。

## 核心观点

- **"先规划后执行"不是流程，是权限约束**：Plan 模式通过切换权限上下文实现，规划阶段工具权限被限制为只读
- **计划文件是 Agent 的持久化意图**：三层恢复策略（直接读取、文件快照、消息历史）确保计划在各种故障场景下不丢失
- **子 Agent 禁止进入 Plan 模式**：防止父 Agent 被阻塞在用户无法理解的审批请求上
- **Plan-Execute-Verify 是 Agent 工作流的基础范式**：三段分离使每个阶段可独立优化和调试

## 详细内容

### Plan 模式架构

**进入 Plan 模式（EnterPlanModeTool）**：
1. 保存当前权限模式到 `prePlanMode`
2. 运行 classifier activation（`prepareContextForPlanMode`）
3. 切换权限上下文为 plan 模式配置
4. 返回六步行为指令给 Agent

六步指令暗含认知模型（发散 → 收敛）：
- Step 1–2（发散）：探索代码库、识别相似特性
- Step 3–4（过渡）：考虑多种方案及权衡、使用 AskUserQuestion 澄清
- Step 5–6（收敛）：设计具体实施策略、使用 ExitPlanMode 呈现计划

**退出 Plan 模式（ExitPlanModeV2Tool）**：
- 读取 `prePlanMode` 保存值并恢复
- **断路器防御**：如果 `prePlanMode` 是 `auto` 但 auto mode gate 当前关闭（期间安全策略可能变更），则回退到 `default` 模式，防止绕过新激活的安全限制
- **Teammate 路径**：通过 mailbox 机制将计划发送给 team lead 审批，而非弹出本地 UI

### 计划文件管理

**命名策略**：slug-based，主会话用 `{slug}.md`，子 Agent 用 `{slug}-agent-{agentId}.md`，避免文件冲突。

**三层恢复策略**（优先级从高到低）：

| 层级 | 方式 | 覆盖场景 |
|------|------|---------|
| 1 | 直接读取本地文件 | 本地文件正常，会话重启 |
| 2 | 文件快照恢复（`findFileSnapshotEntry`） | 远程会话，本地文件未持久化 |
| 3a | ExitPlanMode tool_use input | API 协议保证，最可靠 |
| 3b | User message planContent 字段 | clear context 后保留 |
| 3c | Attachment message plan_file_reference | auto-compact 后保留 |

**Fork 恢复**：`copyPlanForFork` 为 fork 会话生成新 slug 并复制计划文件，防止原始会话和 fork 会话互相覆盖。

### 计划验证机制

`registerPlanVerificationHook` 必须在 context clear **之后**注册（context clear 会清除所有 hooks）。

验证智能体作为独立后台 Agent 运行，持有原始计划文件快照，以"旁观者"视角对照实施结果。避免执行者用自己的理解来评判自己的工作（自我审查偏差）。

验证流程：
1. ExitPlanMode 保存计划文件 → 清除上下文 → 注册验证钩子
2. 执行阶段按计划实施
3. 执行完成 → 验证钩子触发 → 启动验证智能体
4. 验证智能体读取计划文件，输出合规项/偏差项/遗漏项报告

### Fork Agent 的状态隔离

`createSubagentContext` 创建完全隔离的上下文：

| 隔离项 | 策略 | 原因 |
|--------|------|------|
| readFileState | 从父上下文克隆 | 子 Agent 可读外部世界 |
| abortController | 创建子控制器，链接父控制器 | 父中止传播到子，子中止不影响父 |
| setAppState | 默认 no-op | 子 Agent 不能修改父状态 |
| UI callbacks | 全部 undefined | 防止 prompt injection 利用子 Agent 钓鱼 |

唯一通信通道：结构化的 `tool_result` 消息返回给父 Agent。

**最小权限原则**：只传递子 Agent 完成任务所需的最小上下文，减少安全风险和 token 消耗。

### 调度系统

**ScheduleCronTool** 支持两类任务：
- **one-shot**（`recurring: false`）：触发一次后自动删除
- **recurring**（`recurring: true`）：按计划重复触发

存储在 `<project>/.claude/scheduled_tasks.json`。`durable: false` 的任务仅存在于进程内存，会话结束即消失。

**CronScheduler 关键特性：**

1. **文件锁**：确保同一项目目录下多个 Claude 会话不重复触发同一任务。非 owner 会话每 5 秒探测一次锁，owner 崩溃则接管。

2. **Jitter 机制**：避免惊群效应。Recurring 任务添加确定性正向延迟（与触发间隔成正比，默认不超过 15 分钟）。One-shot 任务使用反向 jitter（提前触发）。

3. **错过任务检测**：启动时检查是否有下次触发时间已过去的任务。

4. **自动过期**：Recurring 任务默认 7 天后过期，防止僵尸任务持续消耗资源。`permanent` 标志的内置任务豁免此限制。

**RemoteTriggerTool** 通过 Anthropic API 的 triggers 端点实现远程触发，支持 list/get/create/update/run 五种操作。需同时满足：feature flag `tengu_surreal_dali` + policy limit `allow_remote_sessions`。

**`lastCacheSafeParams` 全局 slot**：在 post-turn hooks 中保存当前轮次的 cache-safe params，让 post-turn fork（promptSuggestion、postTurnSummary 等）可以直接复用主循环的 prompt cache，无需每个调用者手动传递参数。写入和读取发生在同一同步执行段，无竞态条件。

### 工作流设计模式

**推荐模式一：Plan-Execute-Verify 循环**

Plan（只读探索）→ Execute（按计划实施）→ Verify（独立验证）→ Done 或回到 Plan

**推荐模式二：Event-Triggered Workflow**

外部事件 → 触发 Agent → Plan 模式 → 用户审批 → 执行 → 报告结果

适用于运维自动化、CI/CD 集成。

**推荐模式三：Polling Loop with Escalation**

定时任务触发 → 轮询检查 → 正常则继续，异常则升级进入 Plan 模式

低成本定时轮询 + 高成本 Agent 推理仅在异常时触发。

**反模式：**
- 对复杂任务不经 Plan 模式直接执行（风险：大面积返工）
- 对简单任务过度使用 Plan 模式（浪费 token）
- 创建 recurring 任务不设过期时间（僵尸任务）
- 主对话中同步等待轮询结果（正确做法：异步推送通知）

## 相关概念

- [[Harness-Engineering]] — Plan 模式是 Harness 四大支柱"结构化执行"的具体实现
- [[Spec-Coding]] — 规范驱动开发，Plan 模式的外部工具类比
- [[流式架构与性能优化]] — Fork Agent 的提示缓存共享机制
- [[Agent-Harness-构建指南]] — 自定义 Harness 中的六步构建，权限管线与 Plan 模式互补
- [[Agent构建四阶段]] — Plan 模式建立在 Agent Loop 之上
