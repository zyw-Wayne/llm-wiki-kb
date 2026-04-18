---
title: Chain-of-Thought 横纵分析
tags: [CoT, ToT, ReAct, 推理, 横纵分析, Prompt]
category: pattern
date: 2026-04-13
sources:
  - 百度百科：思维链
  - 知乎：CoT/ToT/ReAct 等 Reasoning 方法对比&实践
  - 百家号：AI推理方法演进：CoT/ToT/GoT 技术对比分析
  - CSDN：一文了解Agent：ReAct、CoT、ToT 三大推理框架全解析
  - 知乎：大模型的CoT思维链，只是在"表演"思考
  - 知识库已有页面：Chain-of-Thought
---

# Chain-of-Thought 横纵分析

> 横纵分析法研究 CoT 的学术脉络、关键演进节点与横向竞品对比。

## 一、纵向分析：起源与演进

### 诞生背景（2021–2022）

2022 年 1 月，**Jason Wei**（谷歌大脑高级研究员）在 arXiv 发布开山论文 *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*（arXiv: 2201.11903），同年被 **NeurIPS 2022** 接收。

时代背景：NLP Prompt Learning 兴起约两年，OpenAI 刚推出 GPT-3 第二个大版本更新。核心矛盾：**仅靠扩大模型规模并不能在算术、常识、符号推理等复杂任务上取得好性能**。

核心发现：仅用 **8 个思维链样例**提示 PaLM-540B，在 GSM8K 数学应用题上达到 SOTA 精度，甚至超过了用验证器微调的 GPT-3。

### 演进时间线

| 时间 | 里程碑 | 核心突破 |
|------|--------|----------|
| **2022.01** | CoT 诞生（Wei et al.） | Few-shot CoT，GSM8K 准确率 17.9%→58.1% |
| **2022.05** | Zero-shot CoT（Kojima et al.） | "Let's think step by step" 即可触发推理，准确率 69.9%→79.0% |
| **2022.05** | Self-Consistency（Wang et al.） | 多路径采样+投票，解决单一推理路径偏差 |
| **2022.10** | Auto-CoT（Zhang et al.） | 自动化生成推理链，无需人工编写示例 |
| **2022.10** | ReAct（Yao et al.） | 推理+行动闭环，引入外部工具调用 |
| **2023.05** | Tree-of-Thought（Yao et al.） | 线性链条→树状分支，支持回溯剪枝 |
| **2023** | Graph-of-Thought | 推理建模为图结构，支持分支合并、循环引用 |
| **2024–2025** | 推理模型时代 | OpenAI o1/o3、Claude 3.5 Sonnet、DeepSeek-R1 将 CoT 内化为模型固有能力 |
| **2025** | 端侧部署 | 高通展示手机/眼镜/汽车多设备协同推理 |

### 三条进化主线

- **深度方向**：CoT → Self-Consistency → Reflexion/Self-Refine
- **结构方向**：线性链条 → 树状分支（ToT）→ 图网络（GoT）
- **交互方向**：纯思维推理（CoT）→ 推理+行动（ReAct）→ 多 Agent 协作

## 二、横向分析：竞品对比

### 核心竞品对比

| 维度 | **CoT** | **ToT** | **ReAct** | **GoT** |
|------|---------|---------|-----------|---------|
| 提出时间 | 2022.01 | 2023.05 | 2022.10 | 2023 |
| 核心结构 | 线性链条 | 树状分支 | 推理-行动闭环 | 任意图结构 |
| 搜索策略 | 贪心（单路径） | BFS/DFS（多路径） | 环境反馈驱动 | 图搜索+合并 |
| 外部交互 | 无 | 无 | 有（工具调用） | 无 |
| 错误恢复 | 一旦出错沿错继续 | 回溯+剪枝 | 环境反馈纠正 | 节点合并重构 |
| Token 消耗 | 1x | 5–10x | 3–8x | 10–20x |
| 适用场景 | 数学、逻辑、常识推理 | 探索规划类复杂问题 | 需要外部信息的任务 | 多源信息综合 |
| 工程落地难度 | 极低 | 中等 | 中等 | 高（仍在研究阶段） |

### 选型指南

| 场景 | 推荐方案 |
|------|----------|
| 日常 prompt 快速提效 | Zero-shot CoT（"Let's think step by step"） |
| 技术选型、架构决策 | Few-shot CoT（见 [[Chain-of-Thought]] 四种实战用法） |
| 需要高可靠性 | Self-Consistency（多路径投票） |
| 需要调用外部工具/API | ReAct（Agent 框架核心） |
| 探索大量可能性 | ToT |
| 2025 年推理模型（o1、Claude 4） | 模型已内置 CoT，无需显式提示 |

## 三、横纵交汇：今天的 CoT 处于什么位置

1. **CoT 作为 prompt 技巧已逐渐"退休"**。2025 年推理模型（o1/o3、DeepSeek-R1、Claude Opus 4）已将 CoT 内化为固有能力，不再需要显式写 "Let's think step by step"。

2. **CoT 作为思想仍无处不在**。Agent 框架、多步推理、自我反思……所有现代 AI 系统的推理层都能追溯到 CoT。

3. **2025 年争议**：亚利桑那州立大学论文 *Is Chain-of-Thought Reasoning of LLMs a Mirage?* 提出质疑——CoT 可能不是真正的推理，而是模型对训练数据中结构化模式的复制。当测试问题偏离训练分布时，CoT 会崩溃。

4. **从知识库视角**：已在 Claude Code 中实践了 CoT 的四种高阶用法（架构决策、Bug 诊断、边界推演、自我质疑）。下一步可关注 **Self-Consistency**（多路径采样投票）提升推理可靠性。

## 推荐深挖方向

- **理解理论**：读原论文 arXiv 2201.11903，以及 "Mirage" 质疑论文
- **工程落地**：在 Claude Code 中组合 CoT + Self-Consistency
- **看前沿**：关注 o1/o3 类推理模型的内部机制揭秘

## 关联页面

- [[Chain-of-Thought]] — CoT 在 Claude Code 中的四种实战用法、常见坑与适用判断
- [[横纵分析法]] — 本次研究使用的方法论框架
- [[Prompt语言选择]] — 语言选择对推理质量的影响
