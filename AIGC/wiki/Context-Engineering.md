---
title: Context Engineering
tags: [context, engineering, anthropic, 上下文管理, agent, 分层上下文]
sources: [raw/踏马的 Agent/article.md, raw/Harness Engineering 深度解析/article.md, raw/有效的 Context 工程（精读、万字梳理）｜见知录 004/article.md]
updated: 2026-04-15
---

# Context Engineering

## 定义

Context Engineering 是 AI 应用开发的第二阶段范式，关注如何管理模型推理时能看到的全部信息——系统指令、工具定义、外部数据、对话历史、MCP 接入的各种服务——在有限的上下文窗口中塞进信号最强的部分，同时把噪音挡在外面。

2025 年 9 月，Anthropic 发布工程博客「Effective context engineering for AI agents」，标志着这一概念正式确立。

## 从 Prompt 到 Context 的换挡

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|-------------------|
| 关注点 | 怎么写指令 | 怎么管理全部信息 |
| 隐含假设 | 模型够聪明，你不会问而已 | 模型能处理更多信息，但你不知道该给什么 |
| 信息量 | 一条提示词 | 系统指令 + 工具 + 数据 + 历史 + MCP |
| 瓶颈 | 措辞质量 | 信息选择与组织 |

## 核心问题

### 信息过载

上下文窗口从 4K → 128K → 百万 token。RAG、工具调用、MCP 相继出现。模型能接收的信息量大了好几个数量级，你能塞给它的东西也多了好几个数量级。

> 你会说话了，但给多了它消化不动，给少了它缺信息，给错了更糟糕。

### 错误上下文是最致命的

模型会非常认真地基于错误的上下文，产出一个看起来很对、实际上离谱的结果。它不会告诉你「你给我的信息有问题」，它只会老老实实地用错误的前提推出一个自洽的结论。

### Context 是有限资源

每一个 token 都有成本。Context Engineering 就是在这个有限窗口里，塞进信号最强的那部分，同时把噪音挡在外面。

### 上下文窗口甜蜜区间（~40%）

Dex Horthy 的量化经验：**上下文填到约 40% 就开始走下坡路**。以 168K token 的上下文窗口为例：

- **Smart Zone（前约 40%）**：聚焦、准确的推理，Agent 拥有相关、精炼的信息
- **Dumb Zone（超过约 40%）**：幻觉、循环、格式错误的工具调用、低质量代码

给 Agent 塞一堆 MCP 工具、冗长文档和累积的对话历史，不会让它更聪明——反而会让它变笨。

## 三层上下文体系（实践框架）

多个独立团队（OpenAI、Anthropic、Horthy、Vasilopoulos）收敛到分层上下文与渐进式披露的方案：

| 层级 | 加载时机 | 内容 | 上下文占用 |
|------|---------|------|-----------|
| Tier 1：会话常驻 | 每次会话自动加载 | AGENTS.md / CLAUDE.md，项目结构概览 | 最小 |
| Tier 2：按需加载 | 特定子 Agent 或技能调用时 | 专业化 Agent 上下文、领域知识 | 中等 |
| Tier 3：持久化知识库 | Agent 主动查询时 | 研究文档、规格说明、历史会话 | 按需 |

Vasilopoulos（2026 论文，283 个开发会话验证）将其形式化为：热记忆（Hot Memory）、领域专家（Domain Experts）、冷记忆知识库（Cold-Memory Knowledge）。

## System Prompt 编写原则

有效 Context 工程的起点。来自 Anthropic 实践和一泽 Eze 的总结：

- **启发式引导**：系统提示应足够灵活，既能输出具体结果，又能泛化应对边界情况
- **结构化提示**：用 XML 标签（`<tag>`）或 Markdown 语法（`##title`）分割提示词，简化人类理解维护
- **先用聪明模型写最小化 Prompt**：优先定义"有什么、做什么"，而非"怎么做"
- **精选最小可行工具集**：工具应自包含、能被 LLM 理解、功能不重叠
- **谨慎使用 few-shot**：避免塞满边缘例子，设计多样化、规范的核心例子；推理模型中尽量避免 few-shot
- **system prompt 本身就是最小化的初始 context**——一个清晰高效的 prompt 用最有必要的 tokens 提供方向指引

## 即时上下文（Just-in-Time Context）

越来越多 AI 应用采用 Agent 自主探索的即时上下文方案：

- 不要求 Agent 一次性回忆全部上下文，而是自主导航检索信息（类似人类"回忆笔记在哪→翻阅记录"的多步思考）
- Cursor 等 AI Coding 工具的典型模式：先看 readme.md → 到对应目录找文件
- 文件名、大小、创建时间等元数据也帮助 Agent 判断信息相关性与价值
- 需权衡即时探索 vs 向量检索/直接拼入 context 的耗时与效果

## 无限上下文：超长程任务方案

| 策略 | 适用场景 | 特点 |
|------|----------|------|
| 压缩（Compaction） | 多轮对话交互 | 有损压缩，丢弃冗余历史 |
| 结构化笔记 | 持久化工作记忆 | 极小开销，外部文件持久化 |
| 多智能体架构 | 复杂任务分解 | 分而治之，降低单 Agent 压力 |

可灵活组合使用，共同为超长程任务提供可能性。

## 瓶颈

**人不知道该给什么信息**——这正是 Context Engineering 阶段的核心瓶颈。

## Claude Code 上下文工程实现（来源：[御舆：解码 Agent Harness](../raw/御舆：解码%20Agent%20Harness/article.md)）

《御舆》Ch5-8 从源码层验证了 Context Engineering 的核心理念，并给出了工业级实现细节：

- **有效窗口约束**：有效窗口 = 模型窗口 - min(最大输出令牌, 20,000)，200K 模型实际可用约 183,616 令牌
- **甜蜜区间的工程实现**：系统在 85%、90%、95% 设置三级阈值，自动触发不同级别的压缩以维持在有效区间内
- **即时上下文的记忆补充**：四类型记忆系统（user/feedback/project/reference）为跨会话的即时上下文提供基础——记忆是线索，Agent 在使用时仍需独立验证当前状态
- **渐进式披露的技术实现**：MicroCompact 保留最近 N 个工具结果（keepRecent），Snip 清除已分析完的文件内容，实现"恰好需要时才保留"

> 上下文管理工程实现详见 [[上下文四级压缩策略]]；跨会话记忆详见 [[Agent-记忆系统]]

## 相关概念

- [[Context-Rot]] — 上下文腐烂是 Context Engineering 需要解决的核心问题
- [[Context-压缩]] — 压缩是应对超长上下文的工程手段之一
- [[上下文管理策略]] — 滑动窗口/摘要压缩/RAG 三大实操策略与组合方案
- [[上下文四级压缩策略]] — Claude Code 的四级渐进压缩工程实现
- [[Agent-记忆系统]] — 跨会话的长期记忆，Context Engineering 的持久化延伸
- [[Harness-Engineering]] — Context Engineering 的下一阶段，从选信息到设约束
- [[Agent-类型分类]] — Anthropic agentic systems 分类体系
- [[MCP]] — 上下文工程的具体实现手段，提供"按需加载"外部数据的标准化协议
- [[LLM-Wiki]] — Karpathy 提出的知识编译方案，将知识预处理后提供给模型
- [[RAG-vs-LLM-Wiki]] — 两种上下文补充策略的对比

## 来源

- [踏马的 Agent](../raw/踏马的%20Agent/article.md) — Prompt/Context/Harness 三阶段演进，Context Engineering 作为中间阶段的定位与局限
- [Harness Engineering 深度解析](../raw/Harness%20Engineering%20深度解析/article.md) — 三层上下文体系、40% 甜蜜区间量化数据、Vasilopoulos 学术验证
- [有效的 Context 工程（精读、万字梳理）｜见知录 004](../raw/有效的%20Context%20工程（精读、万字梳理）｜见知录%20004/article.md) — System Prompt 编写原则、即时上下文、无限上下文三大策略
- [御舆：解码 Agent Harness](../raw/御舆：解码%20Agent%20Harness/article.md) — Ch5-8：有效窗口约束、三级阈值机制、四级压缩策略的源码级工程实现
