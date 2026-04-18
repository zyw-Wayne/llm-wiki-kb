---
title: 有效的 Context 工程（见知录 004）
tags: [context, engineering, anthropic, agent, 上下文管理, 见知录, 知识梳理]
sources: [raw/有效的 Context 工程（精读、万字梳理）｜见知录 004/article.md]
updated: 2026-04-15
---

# 有效的 Context 工程（见知录 004）

## 定义

知乎专栏「见知录 AI with Me」第 004 期，作者一泽 Eze，发布于 2025-10-21。文章万字精读梳理了 Anthropic 的 Context Engineering 方法论和 Agent 类型分类体系，是一篇高质量的中文技术综述。

## 文章结构

文章分三大板块：

1. **方法：有效的 Context 工程** — Context Engineering 概念、Context Rot、三大策略
2. **梳理：Anthropic 界定的 Agent 类型** — Workflow 六种 + Agent 一种的分类体系
3. **反思：止损线，亦是起跑线** — 关于长期决策与筹码分配的个人思考

## 核心观点

- Prompt Engineering 是 Context Engineering 的子集，而非替代关系
- Context Rot（上下文腐烂）三大因素来自 Chroma 团队实验研究
- 有效 Context 工程三大策略：System Prompt 编写、即时上下文探索、无限上下文实现
- 超长程任务三种方案：压缩、结构化笔记、多智能体架构

## 提炼的独立知识页面

本次 ingest 基于本文创建了以下知识页面：

- [[Context-Rot]] — 上下文腐烂现象、Chroma 实验细节
- [[Context-压缩]] — Anthropic 压缩策略与其他超长程方案对比
- [[Agent-类型分类]] — Anthropic agentic systems 分类体系

## 更新的已有页面

- [[Context-Engineering]] — 补充了 Context Rot 关联、有效 Context 工程方法、System Prompt 编写原则

## 文章中有趣的洞察

- "system prompt 就是最小化的初始 context"——将 Prompt Engineering 自然嵌入 Context Engineering 体系
- "针在语义上与周围格格不入反而更容易被识别"——类似找茬游戏原理
- "先用聪明模型写一版最小化提示"——区分模型能力问题与 Prompt 设计问题
- Anthropic 观点：few-shot 要避免塞满边缘例子，应设计多样化、规范的核心例子
- Anthropic 定义 Agent 为 "LLMs autonomously using tools in a loop"

## 作者专栏信息

- 专栏：见知录 AI with Me
- 作者：一泽 Eze
- 往期：Vol.001 最小化提示词原则、Vol.002 新年特刊、Vol.003 AI 无法替代的工作
- 风格：AI + 产品 + 生活 + 哲学，精炼、体系化

## 来源

- [有效的 Context 工程（精读、万字梳理）｜见知录 004](../raw/有效的%20Context%20工程（精读、万字梳理）｜见知录%20004/article.md) — 完整原文
