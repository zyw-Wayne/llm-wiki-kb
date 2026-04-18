---
title: Harness Engineering
tags: [agent, harness, engineering, anthropic, openai, stripe, 约束设计, 四大支柱]
sources: [raw/踏马的 Agent/article.md, raw/Harness Engineering 深度解析/article.md, raw/三论Harness——Hermes Agent 完全指南：当 Harness Engineering 遇见自我进化的智能体/article.md, raw/一文带你看懂，火爆全网的Harness Engineering到底是个啥。/article.md]
updated: 2026-04-15
---

# Harness Engineering

## 定义

Harness Engineering 是指围绕 AI Agent（特别是 Coding Agent）设计和构建**约束机制、反馈回路、工作流控制和持续改进循环**的系统工程实践。它解决的核心问题是：当 AI Agent 拥有了强大的代码生成能力后，如何确保其输出的可靠性、一致性和长期可维护性。

"Harness"本意是马具——缰绳、鞍具那一套东西，把马的力气引到正确方向上。LLM 就像一匹蛮力十足但方向感不太行的马，跑得快但容易跑偏。Phil Schmid 的类比更直接：**模型是 CPU，Harness 是操作系统**——CPU 再强，OS 拉胯也白搭。

## 三层工程概念的嵌套关系

Harness Engineering 是 Prompt Engineering 和 [[Context-Engineering]] 的自然延伸，三者构成嵌套关系：

| 阶段 | 时间 | 核心问题 | 人的角色 |
|------|------|---------|---------|
| Prompt Engineering | 2023 | 人不会跟模型说话 | 写指令 |
| Context Engineering | 2025.09 | 人不知道该给什么信息 | 选信息 |
| Harness Engineering | 2025.11+ | 人不知道怎么指挥 Agent | 设约束 |

mtrajan 的区分：Context Engineering 管"给 Agent 看什么"，Harness Engineering 管"系统怎么防崩、怎么量化、怎么修"。

> 模型越强，人需要做的事情反而越多。做的事不一样了而已。

### 游戏类比（来源：数字生命卡兹克）

三个阶段的直观类比，帮助非技术人群理解：

- **只狼**（动作游戏）→ Prompt Engineering：每一招都得你手搓，按一下键出一招
- **金铲铲之战**（自走棋）→ Context Engineering：活全在前期配置（选英雄、凑羁绊、配装备），配完棋子自己打
- **全面战争**（即时战略）→ Harness Engineering：几千个单位自己在跑，你靠编队、阵型、AI指令去驾驭整盘战局

### 文明类比

三个阶段也映射了人类驯服自然力量的历史：

- **用火**（Prompt）：小心翼翼地喂柴，每一次输入直接决定输出
- **建炉子**（Context）：把火关在结构里，通过调节进气口和烟囱控制火势
- **蒸汽机**（Harness）：火在精密系统里自动运行，有锅炉、气缸、调节阀、安全阀，你管的是系统怎么设计

## 术语起源

2025 年底已有人零星提到，但真正结晶成术语是 2026 年 2 月：

- **Mitchell Hashimoto**（HashiCorp 联合创始人）首次明确命名，核心理念："每当发现 Agent 犯错，就花时间设计一个解决方案，确保 Agent 永远不会再犯同样的错误"
- **OpenAI** 数天后发布百万行代码实验报告
- **Martin Fowler** 深度分析，将 Harness 分为三个领域：Context Engineering、Architecture Constraints、Garbage Collection

## 双类控制机制（Birgitta Böckeler 框架）

Birgitta Böckeler（Thoughtworks）将 Harness 分为两类控制机制（来源：数字生命卡兹克）：

| 类型 | 英文 | 作用 | 典型手段 |
|------|------|------|---------|
| **引导** | Guides (feedforward controls) | AI 行动**之前**设好规则 | CLAUDE.md、代码规范、架构决策记录 |
| **检测** | Sensors (feedback controls) | AI 做完事**之后**验证对不对 | 自动化测试、Lint、CI 流水线 |

好的 Harness 是 Guides 和 Sensors 的组合——前者防患于未然，后者亡羊补牢，形成闭环。

## Harness 与古老学科的映射

AI 使用方式的演变，本质上映射到人类历史上的经典学科（来源：数字生命卡兹克）：

- **Harness** → 控制论（反馈机制驱动复杂系统稳定运行），详见 [[Harness-Engineering-控制论视角]]
- **Skill** → 分类学
- **Prompt** → 语言学
- **Context** → 信息科学
- **Reasoning** → 认知心理学
- **多 Agent 协同** → 管理学

## 为什么需要 Harness Engineering

### 瓶颈在基础设施，不在模型智能

这是业界最核心的共识，得到量化验证：

- **Can.ac 实验**：仅改变 Harness 的工具格式（编辑接口），Grok Code Fast 1 从 6.7% 跃升至 68.3%——没有修改任何模型权重
- **LangChain 实验**：仅通过 Harness 改进，在 Terminal Bench 2.0 上从第 30 名跃升至第 5 名，同一模型提升 13.7 分

结论：在纠结模型选择之前，先审视 Harness 设计能获得更高的投资回报率。

### Agent 的典型失败模式（Anthropic 总结）

1. **试图一步到位（One-shotting）**：上下文窗口耗尽，下一会话面对半成品
2. **过早宣布胜利**：看到部分进展就宣布完成，大量功能未实现
3. **过早标记功能完成**：写完代码就标"完成"，没做端到端测试
4. **环境启动困难**：每次新会话花大量 token 弄清怎么运行应用

## 四大支柱

综合 OpenAI、Anthropic（Carlini C 编译器）、Huntley、Horthy 等五个独立团队实践，四种模式反复出现形成收敛。

### 支柱一：上下文架构（Context Architecture）

**核心原则**：Agent 应恰好获得当前任务所需的上下文——不多不少。

每个团队都独立发现，将所有指令塞进一个文件无法扩展。解决方案是**分层上下文与渐进式披露**：

| 层级 | 加载时机 | 内容 | 上下文占用 |
|------|---------|------|-----------|
| Tier 1：会话常驻 | 每次会话自动加载 | AGENTS.md / CLAUDE.md，项目结构概览 | 最小 |
| Tier 2：按需加载 | 特定子 Agent 或技能调用时 | 专业化 Agent 上下文、领域知识 | 中等 |
| Tier 3：持久化知识库 | Agent 主动查询时 | 研究文档、规格说明、历史会话 | 按需 |

**上下文窗口甜蜜区间**：Dex Horthy 的经验观察——上下文填到约 **40%** 就开始走下坡路。前 40% 是 Smart Zone（聚焦准确），超过后进入 Dumb Zone（幻觉、循环、低质量代码）。给 Agent 塞一堆 MCP 工具和冗长文档，不会让它更聪明——反而变笨。

### 支柱二：Agent 专业化（Agent Specialization）

**核心原则**：专注于特定领域、拥有受限工具的 Agent 优于拥有全部权限的通用 Agent。

专业化不仅是组织性的——它本身就是上下文管理策略。每个专家携带更少无关信息，运行在"Smart Zone"内。

实践中的角色分工：

| Agent 角色 | 职责 | 工具权限 |
|-----------|------|---------|
| 研究 Agent | 探索代码库、分析实现 | 只读（Read, Grep, Glob） |
| 规划 Agent | 将需求分解为结构化任务 | 只读，无写入 |
| 执行 Agent | 实现单个具体任务 | 限定范围读写 |
| 审查 Agent | 审计完成工作，标记问题 | 只读 + 标记 |
| 调试 Agent | 修复审查发现的问题 | 限定范围修复 |
| 清理 Agent | 对抗熵积累 | 读写 |

### 支柱三：持久化记忆（Persistent Memory）

**核心原则**：进度持久化在文件系统上，而非上下文窗口中。每次新 Agent 会话从零开始，通过文件系统制品重建上下文。

Anthropic 的经典方案：

- **初始化 Agent**：建立 init.sh 脚本、claude-progress.txt 进度文件、初始 git 提交
- **编码 Agent**：每次会话做增量进展，留下结构化更新

关键发现：**JSON 格式追踪 feature 状态比 Markdown 更有效**——Agent 不太可能不恰当地修改结构化数据。

### 支柱四：结构化执行（Structured Execution）

**核心原则**：将思考与执行分离。**理解 → 规划 → 执行 → 验证**。

- **审查计划远比审查代码快**——规格正确时实现自然可靠，规格有误时可在 500 行代码生成前纠正
- Boris Tane（Cloudflare）："永远不要让 Agent 在你审查和批准书面计划之前写代码"

## 先进团队实战案例

### OpenAI：百万行代码零手写

- 3 名工程师，5 个月，约 100 万行代码，0 行手写
- 约 1,500 个合并 PR，日均 3.5 PR/人，效率提升约 10 倍

**五大原则**：

1. **设计环境，而非编写代码** — Agent 卡住时诊断"缺少什么能力"
2. **机械化执行架构约束** — Types → Config → Repo → Service → Runtime → UI，自定义 Linter 自动检测违规
3. **代码仓库作为唯一事实源** — Slack/Google Docs 中的知识对 Agent 等于不存在
4. **可观测性连接到 Agent** — Chrome DevTools 捕获 DOM 快照，日志/指标可查询
5. **对抗熵** — 自动化"垃圾回收" Agent，清理吞吐量与生成吞吐量成比例

**自定义 Linter 的巧妙设计**：错误消息不仅标记违规——**还告诉 Agent 如何修复**。工具在 Agent 工作时同时"教会"它。

### Anthropic：16 个 Agent 构建 C 编译器（Carlini）

- 约 2 周，16 个 Claude Opus 4.6 并行，约 2,000 次会话
- 100,000 行 Rust 代码，GCC torture test 通过率 99%
- 可编译 150+ 真实项目（PostgreSQL, Redis, FFmpeg, CPython, Linux 6.9 Kernel）
- 总 API 成本约 $20,000

**关键 Harness 设计**：
- **上下文窗口污染缓解**：最小化控制台输出，grep 友好的单行错误格式
- **确定性测试子采样**：每个 Agent 跑 1-10% 测试，跨 VM 随机，集体覆盖全套
- **CI 作为 Harness**：用 Harness 层面的解决方案应对模型层面的问题

### Stripe：千级 PR 的 Minions 系统

开发者 Slack 发任务 → Agent 全程包办到 PR → 人只在最后审查

- **Toolshed MCP 服务器**：近 500 个工具，覆盖内部系统和 SaaS
- **隔离预热 Devbox**：与人类工程师相同的开发环境，但与生产/互联网隔离
- Agent 需要和人类工程师一样的上下文和工具——一等公民，非事后补上

### Huntley 的 Ralph Wiggum Loop

`while :; do cat PROMPT.md | claude-code; done` 的核心不是循环——而是**反压（Backpressure）**：

- **上游反压**：确定性设置、一致上下文、现有代码模式引导首选实现
- **下游反压**：测试、类型检查、Lint、构建、安全扫描器拒绝无效工作
- NixOS 裸金属运行，直接推 master，无分支，无人工 CR，30 秒部署

## AGENTS.md——Agent 的活文档

不是一次性编写的静态文档，而是**每当 Agent 犯错时都要更新的反馈循环**。Hashimoto 的 Ghostty 项目中 AGENTS.md 每一行都对应一个历史 Agent 失败案例。

OpenAI 的进阶实践：小型 AGENTS.md 指向更深层事实源（设计文档、架构图、执行计划），后台 Agent 定期扫描过期文档并提交清理 PR——**Agent 为 Agent 维护文档**。

## Harness 成熟度模型

| 阶段 | 特征 | 工程师角色 |
|------|------|-----------|
| Level 0：无 Harness | 直接给 prompt，无结构化约束 | 手动写代码 + 偶尔用 AI |
| Level 1：基础约束 | AGENTS.md + 基础 Linter + 手动测试 | 主要写代码，AI 辅助 |
| Level 2：反馈回路 | CI/CD 集成 + 自动化测试 + 进度追踪 | 规划+审查为主 |
| Level 3：专业化 Agent | 多 Agent 角色分工 + 分层上下文 + 持久化记忆 | 环境设计+管理为主 |
| Level 4：自治循环 | 无人值守并行化 + 自动化熵管理 + 自修复 | 架构师+质量把关者 |

## 工程落地范例：[[Hermes-Agent]]

Nous Research 的 Hermes Agent（2026.02 开源，GitHub ~22k 星标）是 Harness Engineering 从理论到工程的完整实现。Agent = Model + Harness 在 Hermes 中体现为五大子系统：

- **Agent Loop**（控制层）：Provider 动态选择、Prompt 多源组装、工具调用/重试/回退
- **Prompt System**：提示词即声明式配置——SOUL.md、MEMORY.md、Skill Documents 都是版本控制的代码文件，支持 Anthropic 缓存断点和上下文智能压缩
- **Provider Resolution**：模型无关设计，支持 18+ Provider，驾驭逻辑与模型后端解耦
- **Tool System**：47 个工具跨 20 个工具集，终端工具支持 Local/Docker/SSH/云端多后端
- **Session Persistence**：SQLite FTS5 + 向量存储，状态持久化为可搜索的知识图谱

**三层记忆系统**（短期推理 → 程序性技能文档 → 情境持久层）将 Harness 的"状态"管理从变量存储升级为结构化知识演化。**驾驭闭环**（自主技能创建 + 用户建模 + RL 训练）使 Harness 从静态约束变为自我进化的控制策略。

> 详细分析见 [[Hermes-Agent]]

## 业界六大共识

1. **瓶颈在基础设施，不在模型智能** — 6+ 来源支持，无反对
2. **文档必须是活的反馈循环** — 不是静态制品
3. **思考与执行必须分离** — 所有团队独立发现
4. **上下文不是越多越好** — 量化数据支持（~40% 甜蜜区间）
5. **约束必须机械化执行** — "if it cannot be enforced mechanically, agents will deviate"
6. **工程师角色正在转变** — 从"写代码"到"设计环境 + 管理工作"

## 四大分歧

1. **Harness 简化 vs 精细化** — Manus 每次重写都在简化；OpenAI 构建了大量定制工具。取决于通用产品 vs 定制项目
2. **单 Agent vs 多 Agent** — 任务复杂度和代码库规模是决定因素，Anthropic 承认这是 open question
3. **人类介入程度** — 从 Hashimoto 深度参与到 Stripe/Huntley 无人值守，取决于 Harness 成熟度
4. **术语边界** — 嵌套关系（Harness ⊇ Context ⊇ Prompt）vs 互补关系

## 三大空白区

1. **棕地项目的 Harness 改造** — 所有成功案例全是绿地项目，零成功案例用于遗留代码库
2. **功能和行为验证的系统化方案** — "约束不做错事"已有方法，"验证做对了事"远未解决
3. **AI 生成代码的长期可维护性** — Agent 积累技术债的方式不同于人类

## 趋势

- **Harness 将成为新的服务模板**（Martin Fowler）：团队从预制 Harness 中选择，像今天的服务模板
- **模型越强，Harness 越重要**：Opus 4.5 → 4.6，每个能力级别都得重新设计 Harness
- **Harness 应趋向简化**：随模型能力提升越做越薄，越做越复杂大概率是过度工程化

## Claude Code 基础篇（来自御舆第一部分，Ch1-4）

以下内容来自《御舆：解码 Agent Harness》第 1-4 章（基础篇），补充了 Claude Code 的基础设计层说明。

### Agent Harness 的精确定义

> "Agent Harness 是围绕 LLM 构建的运行时框架，它将 LLM 从一个文本生成器提升为一个能够安全、可靠、高效地与外部世界交互的自主智能体。它不是 SDK，不是 API 封装，更不是简单的 prompt engineering，而是一套让 LLM 真正'上路行驶'的工程基础设施。"

### 马车隐喻——御舆命名由来

"御舆"取自《周礼·考工记》，以先秦马车各构件隐喻 Harness 各子系统：辐（工具系统）、軎辖（权限管线）、辕（对话循环）、轼（钩子系统）。御者技艺 = 架构认知。

### 五大设计原则（从 Claude Code 架构提炼）

从 Claude Code 源码提炼，是四大支柱在技术实现层的表达。详见 [[Agent-Harness-设计哲学五大原则]]。

1. **异步流式优先**（Async Generator First）— AsyncGenerator 唯一同时满足流式输出、可取消、类型安全
2. **安全边界内嵌**（Security at the Perimeter）— 四阶段权限管线纵深防御，见 [[Agent-权限管线四阶段]]
3. **缓存感知设计**（Cache-Aware Architecture）— 工具顺序排序、系统 Prompt 稳定性保障缓存命中
4. **渐进式能力扩展**（Progressive Capability）— Tool → Skill → Plugin → MCP 四级扩展
5. **不可变状态流转**（Immutable State Flow）— 每次 continue 构造新 State 对象，transition 字段追踪跳转原因

### 简单封装的反模式

`while (true) { callAPI(); parseResponse(); executeTool(); }` 是"简单封装"反模式，假设了理想化的世界。Harness 系统性解决六大工程挑战：流式输出、权限管控、上下文管理、错误恢复、状态持久化、可扩展性。见 [[AI编程范式三次浪潮]]。

## Claude Code 工程实现（来自御舆系列，第四部分）

以下内容来自《御舆：解码 Agent Harness》第13-15章，呈现了 Claude Code 作为工业级 Harness 的具体工程实现细节。

### 流式架构：Harness 的基石

Claude Code 的流式架构体现了 Harness 四大支柱中"结构化执行"的技术实现：

- **QueryEngine** 作为会话状态的唯一所有者，通过 `AsyncGenerator` 实现流式消息传递，状态在 turn 之间持久保持
- **StreamingToolExecutor** 实现"流到即执行"，只读工具（Read/Grep/Glob）并行，写入工具（Bash/Edit/Write）串行独占，结果按顺序缓冲输出
- **启动性能**：通过并行预取将 MDM 读取、Keychain 预取与 import 链并行化，启动时间从 160ms 降至 65ms（节省 59%）

> 详见 [[流式架构与性能优化]]

### Plan 模式：结构化执行的落地

Plan 模式是"思考与执行分离"原则的系统实现：

- **权限切换机制**：通过 `EnterPlanMode`/`ExitPlanMode` 工具切换权限上下文，规划阶段仅允许只读工具
- **计划文件持久化**：三层恢复策略（本地文件、文件快照、消息历史）确保计划在各种故障场景不丢失
- **断路器防御**：退出 Plan 模式时检查 auto mode gate 状态，防止绕过期间激活的安全限制
- **调度系统**：CronScheduler 支持 one-shot/recurring 任务，文件锁防止多会话重复触发，jitter 防止惊群效应

> 详见 [[Plan模式与结构化工作流]]

### Harness 构建六步法

从 Claude Code 源码提炼的 Harness 构建路线图：

1. **对话循环**：`while(true)` + State 对象 + `continue` 管理状态流转（循环优于递归）
2. **工具系统**：`buildTool` 工厂函数 + fail-closed 默认值（默认不安全、有副作用）
3. **权限管线**：四阶段短路检查（validateInput → checkPermissions → 钩子 → 用户确认）
4. **上下文管理**：渐进式压缩（Snip → 微压缩 → 摘要，分级触发）
5. **记忆系统**：提取高价值记忆（偏好、约定、决策），按相关性选择性注入
6. **钩子系统**：Shell 命令扩展点，20+ 生命周期节点，fail-open 设计

> 详见 [[Agent-Harness-构建指南]]

### 成本控制的工程实现

- Token 成本追踪：`updateUsage`（`> 0` 守卫，单调递增保证）+ `accumulateUsage`（跨消息累加）
- 提示缓存：缓存键五维度（system prompt、tools、model、messages prefix、thinking config），fork 通过字节级一致性共享父请求缓存，命中率 80% 时节省 72% 成本
- 工具输出的 token 成本往往高于模型输出：一次大文件读取可产生 60K+ 缓存创建 token

## Claude Code 核心系统篇（来自御舆，第二部分 Ch5-8）

第二部分从源码级揭示了 Harness 四大支柱在 Claude Code 中的具体工程实现：

### 支柱一（上下文架构）：配置系统（Ch5）

六层优先级配置（pluginSettings → policySettings），信任半径递减安全边界——projectSettings 在所有安全敏感检查中被排除防供应链攻击（RCE 风险）。编译时 `feature()` 零开销功能开关 + GrowthBook 运行时实验。34 行不可变 Store 状态管理（DeepImmutable + useSyncExternalStore）。
> 详见 [[Agent-配置系统]]

### 支柱三（持久化记忆）：记忆系统（Ch6）

闭合四类型（user/feedback/project/reference），只保存不可从代码推导的信息。Fork 模式后台提取共享提示缓存（节省 96.7% 成本），互斥机制防重复，工具权限白名单实现最小权限原则。
> 详见 [[Agent-记忆系统]]

### 支柱一（上下文架构）：上下文管理（Ch7）

四级渐进压缩（Snip → MicroCompact → Collapse → AutoCompact），断路器防 API 调用雪崩（真实数据：曾有会话出现 3,272 次连续失败），双阶段压缩提示（analysis 草稿本丢弃 + summary 进入上下文）。
> 详见 [[上下文四级压缩策略]]

### 约束机械化：钩子系统（Ch8）

五种钩子类型（Command/Prompt/Agent/HTTP/Function）、26 个生命周期事件，三层安全门禁（全局禁用 → 仅托管钩子 → 工作区信任），结构化 JSON 响应协议（decision/updatedInput/additionalContext）。
> 详见 [[Agent-钩子系统]]

**核心验证**：断路器模式、projectSettings 排除原则、缓存感知架构（接口一致行为可变）——均是"约束基础设施比模型能力更关键"共识的量化工程证据。

## 相关概念

- [[Context-Engineering]] — Harness 的子集，管理模型推理时的全部信息
- [[代码执行范式]] — Anthropic 提出的 Agent 新范式，Token 消耗降低 98.7%
- [[AI编程行为准则]] — Karpathy 四原则，行为约束比更大模型更有效
- [[约束先行]] — Harness 引导控制层的通俗表达，非技术用户的最小可行实践
- [[Spec-Coding]] — 规范驱动开发，用三文档结构控制复杂 AI 项目
- [[Skill实战踩坑指南]] — Anthropic 数百个 Skill 实战踩坑经验
- [[Claude-Code-系统化用法]] — Harness 的终端用户视角：12 种系统化用法是约束设计的个人实践
- [[Hermes-Agent]] — Harness Engineering 的第一个完整工程落地范例
- [[流式架构与性能优化]] — Harness 技术基础：流式处理、并发控制、启动优化、提示缓存
- [[Plan模式与结构化工作流]] — 结构化执行支柱的落地：Plan 模式、调度系统、Fork Agent
- [[Agent-Harness-构建指南]] — 从源码提炼的六步构建路线图和架构教训
- [[Agent-配置系统]] — Ch5：六层配置、安全边界、功能开关、状态管理
- [[Agent-记忆系统]] — Ch6：四类型记忆、Fork 模式提取、缓存感知架构
- [[上下文四级压缩策略]] — Ch7：四级渐进压缩、断路器、双阶段提示工程
- [[Agent-钩子系统]] — Ch8：五种钩子、26 个生命周期事件、三层安全门禁

## 来源

- [踏马的 Agent](../raw/踏马的%20Agent/article.md) — Prompt/Context/Harness 三阶段演进分析，核心论点是"瓶颈一直在人身上"
- [Harness Engineering 深度解析](../raw/Harness%20Engineering%20深度解析/article.md) — 8 个独立信息源交叉对比，四大支柱框架、实战案例（OpenAI/Anthropic/Stripe）、成熟度模型、六大共识与四大分歧
- [三论Harness——Hermes Agent 完全指南](../raw/三论Harness——Hermes%20Agent%20完全指南：当%20Harness%20Engineering%20遇见自我进化的智能体/article.md) — Hermes Agent 架构详解、Harness 五大子系统工程实现、多智能体演进路线图
- [御舆：解码 Agent Harness](../raw/御舆：解码%20Agent%20Harness/article.md) — Claude Code 源码深度解析：第二部分（Ch5-8）覆盖配置系统/记忆系统/上下文管理/钩子系统四大核心系统；第三部分（Ch13-15）覆盖流式架构、Plan 模式、Harness 构建工程实践
- [一文带你看懂，火爆全网的Harness Engineering到底是个啥。](../raw/一文带你看懂，火爆全网的Harness%20Engineering到底是个啥。/article.md) — 数字生命卡兹克，2026-04-15，非技术视角科普：游戏/文明类比、Birgitta 双类控制机制、学科映射
- [一文讲透：Harness Engineering即控制论！](../raw/一文讲透：Harness%20Engineering即控制论！/article.md) — 邬俊杰，腾讯云开发者，2026-04-17，控制论视角深度解读
