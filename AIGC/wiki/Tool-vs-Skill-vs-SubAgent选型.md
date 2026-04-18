---
title: Tool vs Skill vs Sub-Agent 选型
tags: [agent, tool, skill, sub-agent, 选型, 决策树, 架构]
sources: [raw/零废话！一文讲透从0构建AI Agent/article.md]
updated: 2026-04-15
---

# Tool vs Skill vs Sub-Agent 选型

## 定义

AI Agent 系统中三种不同粒度的能力单元：Tool（原子操作）、Skill（固定流程模板）、Sub-Agent（独立子任务），各自适用于不同场景。

## 决策树

```
需求来了
├─ 单个原子操作？（读文件、发请求、执行命令）
│   └─ → 用 Tool
├─ 固定流程？（每次步骤一样，只是输入不同）
│   └─ → 用 Skill
├─ 需要独立上下文？（中间过程会污染主对话）
│   └─ → 用 Sub-Agent
└─ 需要不同模型？（子任务适合用专门的模型）
    └─ → 用 Sub-Agent
```

## 三者对比

| 维度 | Tool | Skill | Sub-Agent |
|------|------|-------|-----------|
| 粒度 | 单个动作 | 一套完整流程 | 独立的子任务 |
| 触发 | LLM 自动选择 | 用户命令 / LLM 调用 | LLM 调用 |
| 执行方式 | 直接执行函数 | prompt 注入主对话上下文 | 派生独立 Chat 实例 |
| 上下文 | 共享主 Agent 上下文 | 共享主 Agent 上下文 | 独立上下文 |
| 典型场景 | 读文件、发请求 | 生成 commit、代码审查流程 | 技术调研、竞品分析 |

## 具体场景对照

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| 读取一个文件 | Tool | 单个原子操作 |
| 生成 Git commit message | Skill | 固定流程（查 diff → 写 message → 提交） |
| 调研某个技术方案 | Sub-Agent | 需要大量搜索，中间结果会污染主上下文 |
| 执行代码审查 | Sub-Agent | 需要独立上下文，可能需要不同模型 |
| 格式化代码 | Tool | 单个操作，调用 formatter 即可 |
| 生成单元测试 | Skill | 固定流程（读源码 → 分析 → 生成测试） |
| 同时调研 3 个竞品 | Sub-Agent × 3 | 并行执行，各自独立上下文 |

## 组合使用

实际项目中三者经常组合。典型模式：用户触发 Skill → Skill prompt 指导主 Agent 拆分任务 → 派生多个 Sub-Agent 并行执行 → 各 Sub-Agent 使用 Tool 完成原子操作 → 主 Agent 汇总结果。

## Sub-Agent 的两种并行模式（源码级补充）

Claude Code 的 Sub-Agent 系统提供两种并行策略，选型需根据任务特征决定（来源：raw/御舆：解码 Agent Harness/article.md）：

| 维度 | Fork 模式 | Coordinator 模式 |
|------|-----------|------------------|
| 架构 | 去中心化（对等并行） | 中心化（协调者-工作者） |
| 上下文共享 | 所有子智能体继承完整父上下文 | 工作者只看到分配的任务 |
| 缓存效率 | 字节级共享，可节省 60%+ token | 不共享缓存前缀 |
| 适用场景 | 独立的并行调查/搜索任务 | 需要协调的复杂多步骤任务 |
| 工作者间通信 | 无直接通信 | Scratchpad 共享空间 |
| 激活方式 | feature gate + 非 Coordinator 模式 | 环境变量 `CLAUDE_CODE_COORDINATOR_MODE` |

**两者互斥**：Coordinator 模式激活时 Fork 模式被自动禁用。

### 四种内置 Sub-Agent 专业分工

| Agent | 职责 | 关键约束 |
|-------|------|---------|
| Explore | 只读代码探索 | 双重锁：提示层软约束 + 工具层硬约束；omitClaudeMd |
| Plan | 结构化规划 | 禁止修改类工具；关注"做什么"而非"怎么做" |
| General Purpose | 通用执行 | 全部工具权限（受全局过滤函数约束） |
| Verification | 对抗性验证 | 目标是破坏代码；后台运行；禁止修改文件 |

## Claude Code 四级扩展模型（来源：raw/御舆：解码 Agent Harness/article.md，Ch1）

Claude Code 将扩展能力设计为四级渐进模型（"渐进式能力扩展"原则），与选型层次对应：

| 扩展级别 | 机制 | 适用场景 | 扩展者 |
|---------|------|---------|-------|
| **Tool** | 实现 `Tool` 类型接口 | 添加原子操作能力 | 核心开发者 |
| **Skill** | Markdown + 脚本的声明式工具 | 封装可复用任务模板 | 高级用户 |
| **Plugin** | 带生命周期的工具包 | 组织相关工具和配置 | 生态开发者 |
| **MCP 服务器** | 标准化协议外部工具集成 | 第三方工具生态 | 第三方开发者 |

设计哲学：**渐进增强**——每一级建立在前一级基础上，简单需求声明一个 Skill，复杂集成通过 MCP 连接外部服务。

工具类型中 `isConcurrencySafe` 标记决定是否可并行调度：只读工具（GlobTool、GrepTool）可并行；写入工具（FileEditTool、BashTool）串行。正确的原子性划分同时是性能问题——影响并发分区调度效率。

## 相关概念

- [[Agent构建四阶段]] — Agent 从零构建的四阶段递进
- [[Skill定义规范]] — Skill 的定义格式与触发机制
- [[Agent-类型分类]] — Anthropic workflow 与 agent 分类
- [[Skills-设计哲学]] — Skill 的"中台思维"推导
- [[SubAgent-与-Fork模式]] — Fork 模式详解（字节级缓存共享、递归防护）
- [[Coordinator模式]] — 多智能体企业级编排（四阶段工作流、Scratchpad）
- [[工具系统五要素协议]] — Tool 的完整类型契约和并发分区策略
- [[Agent-Harness-设计哲学五大原则]] — 原则四（渐进式能力扩展）的来源

## 来源

- [零废话！一文讲透从0构建AI Agent](../raw/零废话！一文讲透从0构建AI%20Agent/article.md) — Tool/Skill/Sub-Agent 选型决策树与对比表
- [御舆：解码 Agent Harness](../raw/御舆：解码 Agent Harness/article.md) — 第1章：四级扩展模型与渐进式能力扩展；第9-10章：Fork 模式、Coordinator 模式、内置智能体分工
