---
title: Hermes Agent
tags: [agent, hermes, harness, memory, skill-document, nous-research, multi-agent]
sources: [raw/三论Harness——Hermes Agent 完全指南：当 Harness Engineering 遇见自我进化的智能体/article.md]
updated: 2026-04-14
---

# Hermes Agent

## 定义

Hermes Agent 是 Nous Research 于 2026 年 2 月发布的开源 AI Agent 运行时，具备持久记忆和自主技能创建能力。它是 [[Harness-Engineering]] 理念的第一个完整落地范例，核心定位是 **"与你共同成长的智能体"（The agent that grows with you）**。截至 2026 年 4 月初，GitHub 约 22,000 星标。

## 核心架构

Hermes 的架构完美诠释了 Harness Engineering 的三大核心要素：**控制（Control）、契约（Contracts）、状态（State）**。

### 三层记忆系统

| 层级 | 名称 | 技术实现 | 作用 |
|------|------|---------|------|
| 第一层 | 短期推理记忆 | 标准 Transformer 上下文 + 上下文压缩 | 即时提示-响应循环，超过阈值时智能总结 |
| 第二层 | 程序性技能文档 | agentskills.io 标准的 Markdown 文件 | 任务完成后自动合成为可复用的 Skill Document |
| 第三层 | 情境持久层 | SQLite FTS5 全文搜索 + 向量存储 | 索引所有技能文档和会话历史，支持数周前的记忆召回 |

> 这三层的本质是将 Harness 中的"状态"管理从简单变量存储升级为结构化的知识演化系统。

### 驾驭闭环（Closed Learning Loop）

1. **自主技能创建**：任务完成后分析执行轨迹，提取可复用模式，生成 Skill Document（程序性知识的显式编码）
2. **用户建模**：通过 Honcho 辩证系统构建持久用户画像，实现个性化约束
3. **强化学习训练**：与 Atropos（Nous Research 的 RL 框架）集成，生成批量轨迹数据微调更小模型，将经验转化为模型能力的永久提升

## 五大子系统

### Agent Loop（同步编排引擎）

`AIAgent` 类（`run_agent.py`）处理：Provider 选择、Prompt 构建、工具执行（调用/重试/回退）、回调与持久化。对应 Harness 的 **控制层**。

### Prompt System（提示词即架构）

包含三个模块：
- **prompt_builder.py**：从 SOUL.md、MEMORY.md、USER.md、技能文档、上下文文件等多维度组装系统提示词
- **prompt_caching.py**：应用 Anthropic 缓存断点进行前缀缓存
- **context_compressor.py**：超过阈值时智能总结中间轮次

> 核心洞见：提示词不是静态文本，而是动态组装的控制界面。

### Provider Resolution（模型无关的驾驭层）

映射 `(provider, model)` 到 `(api_mode, api_key, base_url)`，支持 18+ Provider、OAuth 流程、凭证池和别名解析。驾驭逻辑与模型后端解耦——可随时切换模型而保持技能库、记忆和配置不变。

### Tool System（工具即契约）

中心工具注册表（`tools/registry.py`）包含 47 个注册工具，横跨 20 个工具集。终端工具支持多后端：Local / Docker / SSH / Daytona / Modal / Singularity。每个工具带有明确的输入输出规范、错误处理策略和环境适配能力。

### Session Persistence（状态即真相）

基于 SQLite 的会话存储，支持 FTS5 全文搜索、谱系追踪（父子关系跨越压缩操作）、平台隔离、带竞争处理的原子写入。将状态持久化为可搜索的知识图谱，而非简单的键值对。

## 多智能体演进

### 当前能力（v0.6.0）

通过 `delegate_task` 生成临时子 Agent，支持 Convoy 模式（并行任务 + 合成），属于 **委托（Delegation）** 而非真正的编排。

### 路线图（#344 议题）

四个维度向真正的多智能体架构演进：

| 维度 | 内容 | Harness 对应 |
|------|------|-------------|
| 专业化角色 | 研究员/编码员/审查员/浏览器 Agent | 基于角色的控制策略 |
| 结构化工作流 | 依赖感知的 DAG 任务分解 | 编排逻辑 |
| 智能体间协作 | 共享上下文、基于彼此成果迭代 | 协调协议 |
| 弹性执行 | 崩溃恢复、卡住检测、健康监控 | 故障处理 |

### 三级 Agent 间通信

| 级别 | 机制 | 用例 |
|------|------|------|
| L0: 隔离 | 无共享，父 Agent 中继 | 简单委托 |
| L1: 结果传递 | 上游结果自动注入下游上下文 | 工作流 DAG |
| L2: 共享草稿本 | 读写共享键值存储 | 复杂工作流 |
| L3: 实时对话 | 回合制 Agent 间对话 | 辩论/审查模式 |

## 与其他框架的对比

| 维度 | Hermes Agent | Claude Code | AutoGen |
|------|-------------|-------------|---------|
| 核心定位 | 自我进化的 Agent OS | 工程师的 Agent IDE | 多智能体对话框架 |
| 记忆策略 | 三层记忆 + 技能文档 | 会话级上下文 | 短期对话历史 |
| 控制粒度 | 细粒度 Harness | 粗粒度（系统提示词） | 中粒度（对话流程） |
| 模型依赖 | 模型无关（18+ Provider） | 仅 Claude | 模型相对无关 |
| 学习机制 | 自主技能创建 + RL 训练 | 无内置学习 | 无内置学习 |

## 趋势意义

Hermes 揭示的四个 Harness Engineering 趋势：
1. **提示词即代码（Prompts as Code）**：SOUL.md、Skill Documents 都是版本控制的代码文件
2. **记忆即服务（Memory as a Service）**：三层记忆架构可能演变为独立的 Agent 记忆云
3. **契约即接口（Contracts as APIs）**：能力从隐式的模型权重向显式的契约定义迁移
4. **驾驭即学习（Harness as Learning）**：Harness 不是静态约束，而是自我进化的控制策略

## 相关概念

- [[Harness-Engineering]] — Hermes 是 Harness 理念的第一个完整落地
- [[Context-Engineering]] — 提示词系统中 prompt_builder 的多维度组装是 Context Engineering 的工程化
- [[Skill定义规范]] — Skill Document 遵循 agentskills.io 标准，与 Claude Code Skill 体系理念相通
- [[Skill实战踩坑指南]] — Hermes 的自主技能创建可看作踩坑经验的系统化自动编码
- [[Claude-Code-系统化用法]] — 对比视角：Hermes 的 Agent OS 定位 vs Claude Code 的 IDE 定位

## 来源

- [三论Harness——Hermes Agent 完全指南](../raw/三论Harness——Hermes%20Agent%20完全指南：当%20Harness%20Engineering%20遇见自我进化的智能体/article.md) — Hermes Agent 架构全景、Harness 五大子系统详解、多智能体演进路线、实战指南与框架对比
