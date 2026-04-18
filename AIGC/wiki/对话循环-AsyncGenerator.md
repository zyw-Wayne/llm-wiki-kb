---
title: 对话循环 AsyncGenerator 设计
tags: [agent, async-generator, 对话循环, 状态机, 压缩管线, 依赖注入, Claude Code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# 对话循环 AsyncGenerator 设计

## 定义

Claude Code 的对话主循环是一个以 `async function*` 定义的异步生成器。它不是一次性执行完毕的普通函数，而是一个可暂停、可恢复、可取消的"活"的流程——Agent 的"心跳"。

## 核心观点

- AsyncGenerator 是 Agent 循环的最佳载体：`yield` 提供自然流式输出，`yield*` 提供子生成器委托，`.return()` 提供确定性取消（来源：raw/御舆：解码 Agent Harness/article.md）
- 状态不可变，转换可追踪：每次 continue 都构造新 `State`，配合 `transition` 字段使每次跳转有据可查
- 预处理管线是上下文管理的核心：四层压缩策略从轻量裁剪到全量摘要，确保 Agent 在无限对话中保持 token 预算内

## 详细内容

### 函数签名

对话循环是一个导出的异步生成器函数，可向外产出五种类型的事件：
- `stream_request_start`：每次 API 请求开始前发出，供 UI 显示"正在思考..."
- `StreamEvent`：来自 Anthropic API 的原始流式事件，直接透传给 UI
- `Message`：结构化消息对象（AssistantMessage、UserMessage、SystemMessage 等）
- `TombstoneMessage`：流式回退发生时，标记已产出消息为废弃
- `ToolUseSummaryMessage`：批量工具执行后的异步摘要，用于 UI 折叠展示

最终返回 `Terminal` 对象（对话终结原因）。

### 消息类型体系

| 消息类型 | API 角色 | 职责 |
|---------|---------|------|
| UserMessage | user | 用户输入 + 工具执行结果（tool_result） |
| AssistantMessage | assistant | 模型回复，可同时包含文本和 tool_use 块 |
| SystemMessage | 不参与 API | 系统级通知（权限变更、模型回退等） |
| AttachmentMessage | user | 文件变更通知、内存文件内容、任务通知 |
| ProgressMessage | - | 工具执行进度（Bash 输出流、文件读取进度） |
| TombstoneMessage | - | 标记消息"失效"（流式回退触发） |

> 注意：工具结果以 user 角色发送，这是 API 协议约束（只有三种角色：system/user/assistant）驱动的设计决策。

### 一个完整 Turn 的五阶段生命周期

```
阶段一：状态初始化
  └─ 从状态对象解构当前迭代所需变量
  └─ 构建新的可变状态容器

阶段二：上下文预处理（七步管线）
  ① 工具结果预算 → ② Snip压缩 → ③ Microcompact
  ④ Context Collapse → ⑤ 系统提示组装
  ⑥ Autocompact → ⑦ Token阻断检查

阶段三：API 调用（流式接收）
  └─ 发送系统提示 + 消息历史 + 工具定义
  └─ 流式接收 + 收集 assistant 消息和工具调用块

阶段四：工具调用执行（如有）
  └─ 权限检查 · 并发分区调度 · 流式/批量执行

阶段五：工具结果回填
  └─ 附件注入 · 打包新状态对象 · continue 到阶段一
```

### 上下文预处理：四层压缩管线

**核心原则：从轻量到重量排列，每一步先尝试最小代价方案。**

| 压缩手段 | 策略 | 信息损失 |
|---------|------|---------|
| **Snip** | 直接截断过长消息内容 | 较高，但只截断超长部分 |
| **Microcompact** | 缓存友好的轻量压缩，复用已缓存 token | 低 |
| **Context Collapse** | 将连续消息折叠为紧凑视图，保留信息 | 低，细粒度保留 |
| **Autocompact** | 全量摘要，替换历史对话 | 高，最后手段 |

阶段七 Token 阻断检查是"快速失败"机制——与其发送一个注定失败的 API 请求，不如在本地阻止它。

### 终止条件：十种 Terminal 原因

| 终止原因 | 类型 | 触发条件 |
|----------|------|---------|
| `completed` | 正常 | 模型正常回复且无工具调用 |
| `aborted_streaming` | 用户主动 | 用户中断（Ctrl+C） |
| `aborted_tools` | 用户主动 | 工具执行期间中断 |
| `max_turns` | 异常 | 达到最大循环次数 |
| `blocking_limit` | 异常 | Token 数超过硬性限制 |
| `prompt_too_long` | 异常 | 上下文过长且所有压缩手段已用尽 |
| `model_error` | 异常 | API 调用异常 |
| `stop_hook_prevented` | 异常 | Stop hook 阻止继续 |
| `hook_stopped` | 异常 | 工具 hook 阻止继续 |
| `image_error` | 异常 | 图片尺寸/格式错误 |

> 设计洞察：十种终止原因的精细划分是可观测性（Observability）的基础，调试 Agent 时准确的终止原因是定位问题的第一线索。

### 七条 Continue 路径（状态转换）

1. **next_turn**：正常的工具调用后继续，最常见路径
2. **max_output_tokens_recovery**：模型输出被截断时注入恢复消息，最多重试 3 次
3. **max_output_tokens_escalate**：首次截断时提升输出 token 限制（比 recovery 更优雅）
4. **reactive_compact_retry**：上下文过长时通过响应式压缩恢复
5. **collapse_drain_retry**：上下文折叠的溢出恢复路径，优先于响应式压缩
6. **stop_hook_blocking**：Stop hook 返回阻塞错误时，注入错误到消息列表让模型修正
7. **token_budget_continuation**：Token 预算不足时注入提醒消息（类似"流量余额不足"提醒）

`transition` 字段记录每次 continue 的原因，是调试 Agent 行为的"面包屑"。

### 依赖注入与可测试性

系统定义 `QueryDeps` 接口，包含四个核心依赖：
- `callModel`：模型调用函数
- `microcompact`：轻量压缩函数
- `autocompact`：自动压缩函数
- `uuid`：UUID 生成器

测试时传入自定义依赖对象，无需 spyOn-per-module 的 mock 样板代码。

### 为什么选函数式而非 Class

1. **天然状态隔离**：每次调用创建全新闭包，无跨调用状态泄漏风险
2. **生成器背压语义**：`yield` 暂停执行直到消费者请求，防止内存堆积
3. **取消传播**：`.return()` 触发 finally 块清理资源，确定性取消
4. **可组合性**：`yield*` 委托让主循环与工具执行无缝串联

## 反模式警告

- 避免将对话状态存储在全局变量或 class 的实例属性中
- 避免 `while (true) { callAPI(); parseResponse(); executeTool(); }` 的简单循环——缺少流式输出、上下文管理和错误恢复

## 相关概念

- [[Agent-Harness-设计哲学五大原则]] — 原则一（异步流式优先）和原则五（不可变状态流转）的源头
- [[工具系统五要素协议]] — 对话循环阶段四所调用的工具系统架构
- [[Agent-权限管线四阶段]] — 对话循环阶段四中嵌入的权限检查管线
- [[上下文管理策略]] — 预处理管线四层压缩与通用上下文管理的关联
