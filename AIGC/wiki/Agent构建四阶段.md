---
title: Agent 构建四阶段
tags: [agent, 构建, 入门, ReAct, Tool-Calling, 循环, 教程]
sources: [raw/零废话！一文讲透从0构建AI Agent/article.md]
updated: 2026-04-15
---

# Agent 构建四阶段

## 定义

从最简单的 LLM API 调用到完整的 Agent 循环，分为四个递进阶段，每个阶段解锁一项新能力：单次对话 → 多轮对话 → 工具调用 → Agent Loop。

## 核心观点

- **Agent = LLM（大脑） + Tools（手脚） + Loop（驱动循环）**
- LLM 本质是无状态的"文字接龙"机器，"记忆"不在 AI 脑子里，而在客户端发送的 messages 数组里（来源：[零废话！一文讲透从0构建AI Agent](../raw/零废话！一文讲透从0构建AI%20Agent/article.md)）
- Tool Calling 是从"聊天机器人"进化到"Agent"的关键一步——LLM 只负责决策，代码负责执行
- Agent Loop 即 ReAct（Reason + Act）循环：推理 → 行动 → 观察，反复执行直到任务完成

## 四阶段详解

### 阶段一：单次对话（API 调用）

最简单的起点——调用 LLM API，一问一答。关掉程序就全忘。

核心要素：system prompt（角色指令）+ messages（用户消息）。

### 阶段二：多轮对话（上下文维护）

LLM 本身无状态，每次 API 调用都是独立请求。解决方案：客户端维护 history 数组，每次请求带上完整历史。

> "记忆"不在 AI 脑子里，而在你发送的数组里。

每个模型有**上下文窗口**限制（如 128K tokens），超出则截断最早内容。

### 阶段三：工具调用（Function Calling）

给 AI 一双"手"。向 API 注册工具定义，AI 返回结构化的函数调用请求，代码负责真正执行。

```
AI 返回: { "functionCall": { "name": "get_weather", "args": { "city": "深圳" } } }
实际执行: 你的代码调用天气 API，把结果返回给 AI
```

### 阶段四：Agent Loop（ReAct 循环）

将工具调用放入自动循环，让 AI 反复"思考→行动→观察"直到任务完成。

```
用户："帮我看看当前目录有什么文件，然后统计代码行数"
第 1 圈：AI → 执行 ls -la → 代码返回结果
第 2 圈：AI → 执行 wc -l src/*.ts → 代码返回结果
第 3 圈：AI → 返回文本总结 → 循环结束 ✅
```

MAX_ROUNDS 是安全阀，防止 AI 死循环。

## 工程化原则

1. **工具设计**：从"万能胶"到"手术刀"——原子化、结构化的工具比万能的 run_shell_command 成功率更高
2. **上下文注入**：每轮启动前自动注入 CWD、文件列表、时间戳等环境信息
3. **显示优化**：工具执行结果分 data（给 LLM）和 displayText（给用户），不甩原始 JSON
4. **指令优化**：system prompt 强制要求禁止废话、角色定位明确、鼓励调用工具前做简短推理

## 相关概念

- [[Context-Engineering]] — 上下文管理策略（滑动窗口/摘要压缩/RAG）
- [[MCP]] — Agent 构建阶段四之上的标准化工具协议
- [[Agent-类型分类]] — Anthropic 的 agentic systems 分类体系
- [[Harness-Engineering]] — Agent 约束工程，更高阶的框架设计
- [[Skill定义规范]] — Agent Skill 的定义与触发方式

## 来源

- [零废话！一文讲透从0构建AI Agent](../raw/零废话！一文讲透从0构建AI%20Agent/article.md) — 从零构建 Agent 的四阶段系统教程
