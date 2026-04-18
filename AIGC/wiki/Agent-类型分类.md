---
title: Agent 类型分类
tags: [agent, workflow, anthropic, agentic systems, 分类, 架构]
sources: [raw/有效的 Context 工程（精读、万字梳理）｜见知录 004/article.md]
updated: 2026-04-15
---

# Agent 类型分类

## 定义

Anthropic 在《Building effective agents》中提出的 agentic systems（智能系统）分类体系，将 AI 智能系统划分为两大类：Workflow（工作流，预编排路径）和 Agent（自主智能体，循环使用工具）。

## 核心观点

- 统称为 **agentic systems**，不硬性区分优劣，以解决问题为目标灵活组合
- **最小化设计原则**：如无必要，无增实体。从简单提示+优秀模型开始，智能不足时再添加步骤
- **可解释性优先**：不可解释的 Agent 无法维护，无法维护则无法针对生产环境调优（来源：[[有效的-Context-工程-见知录004]]）

## Workflow 类型

### 增强型 LLM（The Augmented LLM）
- 给 LLM 配上检索、工具、记忆等增强功能
- 与 Agent 的区别：不会规划任务流程，无法自行决定下一步

### 提示链（Prompt Chaining）
- 将任务分解为多个子环节，前一个 LLM 的输出作为下一个的输入
- 示例：营销文案生成 → 翻译；大纲生成 → 检查 → 正文编写

### 路由（Routing）
- LLM 分类 input，在更合适的子任务中解决
- 示例：AI 客服、Chatbot 切换回答模型（简单问题→小模型）

### 并行（Parallelization）
- **Sectioning**：并行处理独立子任务（如意图理解 + 合规审查）
- **Voting**：多次运行同一任务获取多样化结果或投票过滤
- 适用：LLM 同时处理多因素困难时，分解为单因素单模型处理

### 协调-执行（Orchestrator-Workers）
- 中央 LLM 分解任务（子任务不预先定义），工作者 LLM 分别执行并返回结果
- 示例：多文件复杂更改、多来源信息收集

### 评估-优化（Evaluator-Optimizer）
- 一个 LLM 生成，另一个 LLM 评估反馈，循环优化
- 适用：Search、多轮文学创作与编辑

## Agent 类型

- 定义：LLMs autonomously using tools in a loop
- 自主智能体，基于环境反馈循环使用工具，能理解复杂输入、推理规划、从错误恢复
- 常见：Computer Use、Coding Agent
- 随底层模型能力提升而提升独立解决问题的能力

## 设计建议

1. 无硬性规定与优劣，以解决问题为目标出发
2. 最小化设计：先用简单提示+优秀模型实验，只有智能不足时才添加步骤和 Context 指引
3. 保持 Agent 规划步骤的透明度——不可解释则不可维护

## LLM 角色的根本转变（来源：raw/御舆：解码 Agent Harness/article.md，Ch1）

2024 年的核心认知转变：**LLM 的真正价值不在于生成文本，而在于作为推理引擎来编排工具调用。**

- 过去：LLM = 被动的文本生成器（对话伙伴）
- 现在：LLM = 主动的任务编排器（决策中枢），决定调用什么工具、传递什么参数、如何处理返回结果

类比：如果过去的 LLM 是只会口述指令的军事参谋，那么工具调用让 LLM 变成了可以直接调度各兵种协同作战的指挥官。

**工具调用带来的六大工程挑战**（催生 Agent Harness 概念的根本原因）：
1. 工具注册与发现
2. 参数校验（LLM 输出不可控）
3. 权限管控（如何阻止危险操作）
4. 错误恢复（工具执行可能失败）
5. 状态一致性（多工具调用操作同一资源）
6. 并发与调度（哪些可并行，哪些必须串行）

这六大挑战与 Anthropic 的 agentic systems 分类体系相辅相成——Agent 类型决定了需要哪些挑战的解决方案，工程实现决定了系统的健壮程度。

## 相关概念

- [[Context-Engineering]] — Context 工程支撑 agentic systems 的信息管理
- [[Harness-Engineering]] — 约束工程，Agent 设计的更高阶框架
- [[代码执行范式]] — Anthropic 提出的 Agent 代码执行新范式
- [[Agent构建四阶段]] — 从零构建 Agent 的四阶段递进教程（单次对话→ReAct 循环）
- [[有效的-Context-工程-见知录004]] — 原文梳理了完整分类体系
- [[AI编程范式三次浪潮]] — Chatbot 到 Agent 范式转移的历史背景
- [[工具系统五要素协议]] — 工具调用在 Claude Code 中的具体工程实现

## 来源

- [有效的 Context 工程（精读、万字梳理）｜见知录 004](../raw/有效的%20Context%20工程（精读、万字梳理）｜见知录%20004/article.md) — Anthropic workflow 与 agent 分类体系详解
- [御舆：解码 Agent Harness](../raw/御舆：解码%20Agent%20Harness/article.md) — 第1章：LLM 从对话伙伴到推理引擎的认知转变，工具调用六大工程挑战
