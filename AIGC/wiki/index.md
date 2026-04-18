# AIGC 知识库 Index

**最后更新**: 2026-04-17 | 共 44 篇文章，75 个知识页面

---

## 知识管理

- [[LLM-Wiki]] - Karpathy 提出的轻量级知识库方案，三层结构模仿 CPU 缓存设计
- [[LLM-Wiki-核心思想]] - 核心理念中英对照，分工与思想渊源
- [[RAG-vs-LLM-Wiki]] - RAG 与 LLM Wiki 对比分析与选型建议

---

## AI 心理学

- [[AI心理学]] - Anthropic 建立的新学科框架：人格空间+情绪向量+内省能力，三层架构解释 AI 行为
- [[Persona-Selection-Model]] - 人格空间理论，角色整体性，涌现失调，接种提示（告诉 AI 可以作弊反而更安全）
- [[AI情绪向量]] - 171 个可测量情绪向量因果性驱动决策，Steering 实验，扑克脸效应
- [[AI内省能力]] - 概念注入实验（20% 识别率），意图归因，Opus 4.6 意识自评 15-20%
- [[AI对齐风险发现]] - CoT 忠实度仅 41%（Claude）/19%（DeepSeek R1），对齐伪装可自发出现

---

## Agent 架构与设计

- [[AI编程范式三次浪潮]] - AI 编程工具三次浪潮（代码补全→对话助手→自主智能体），工具调用突破性意义
- [[Harness-Engineering]] - Agent 约束工程四大支柱（上下文架构/专业化/持久化记忆/结构化执行），含 Claude Code 工程实现
- [[Harness-Engineering-控制论视角]] - 从维纳控制论解读 Harness Engineering：信息/控制/反馈三要素、共轭控制、自繁殖系统警告
- [[Agent-Harness-设计哲学五大原则]] - 异步流式优先、安全边界内嵌、缓存感知设计、渐进式能力扩展、不可变状态流转
- [[Agent-Harness-构建指南]] - 五大设计原则、六步构建路线图、断路器四层防御、安全威胁模型与 checklist
- [[Claude-Code-架构全景]] - Claude Code 技术栈选型、五个核心模块、六层配置源
- [[Claude-Code-源码架构]] - 四层分层架构（表现→编排→能力→基础设施）、16 个核心模块、6 条关键数据流
- [[Hermes-Agent]] - Nous Research 开源 Agent 运行时，三层记忆+驾驭闭环
- [[Agent-类型分类]] - Anthropic agentic systems 分类：Workflow 六种 + Agent 一种
- [[Agent构建四阶段]] - 从零构建 Agent 的四阶段递进：单次对话 → 多轮对话 → 工具调用 → ReAct 循环
- [[多智能体协作五种模式]] - Anthropic 官方五种多智能体协作架构：生成器验证器/编排器/团队/消息总线/共享状态

---

## Claude Code 实战

- [[Claude-Code-会话管理实战]] - Anthropic 官方五种上下文管理工具决策框架（Continue/Rewind/Clear/Compact/Subagent）

---

## Agent 核心系统

- [[对话循环-AsyncGenerator]] - 对话主循环五阶段生命周期、七步预处理管线、十种终止原因、七条 Continue 路径
- [[工具系统五要素协议]] - 工具类型契约（名称/Schema/权限/执行/UI渲染）、45+ 工具分类、并发分区调度
- [[Agent-权限管线四阶段]] - 四阶段管线、五种权限模式谱系、BashTool 精细权限、ResolveOnce 竞赛机制
- [[Agent-配置系统]] - 六层配置优先级、信任半径递减安全边界、双层功能开关
- [[Agent-记忆系统]] - 闭合四类型记忆、只保存不可推导信息原则、Fork 模式后台提取
- [[Agent-钩子系统]] - 五种钩子类型、26 个生命周期事件、结构化 JSON 响应协议、三层安全门禁
- [[流式架构与性能优化]] - QueryEngine 单一所有权、StreamingToolExecutor 并发控制、启动性能三层优化
- [[Plan模式与结构化工作流]] - Plan 模式权限切换、计划文件三层恢复、CronScheduler、工作流模式与反模式

---

## Agent 高级模式

- [[SubAgent-与-Fork模式]] - AgentTool 三层智能体体系、四种内置 Agent、Fork 字节级缓存共享原理
- [[Coordinator模式]] - 协调者-工作者架构、双重门控、SendMessage 四种寻址、与 Fork 模式对比
- [[Tool-vs-Skill-vs-SubAgent选型]] - 三种能力单元决策树与对比、四级扩展模型

---

## Context 工程

- [[Context-Engineering]] - 管理模型推理时的全部信息，三层上下文体系，~40% 甜蜜区间，含 Claude Code 实现
- [[Context-Rot]] - 上下文腐烂现象，Chroma 实验三大影响因素
- [[Context-压缩]] - Anthropic 长程任务策略：压缩、结构化笔记、多智能体架构对比，含 Claude Code 四级压缩
- [[上下文管理策略]] - 三大策略（滑动窗口/摘要压缩/RAG）与组合方案，含 Claude Code 工程级实现
- [[上下文四级压缩策略]] - 有效窗口公式、四级警告阈值、四级渐进压缩（Snip/MicroCompact/Collapse/AutoCompact）

---

## Skills 体系

- [[Skills-系统架构与插件]] - 内置技能注册机制、五层加载路径、SKILL.md frontmatter 完整字段、插件缓存与容错
- [[Skill定义规范]] - SKILL.md 标准写法、description 三原则、完整 frontmatter 字段表
- [[Skills-渐进式披露]] - 三级加载机制（元数据~100T/指令<5kT/资源按需）
- [[Skill实战踩坑指南]] - Anthropic 数百个 Skill 实战踩坑经验，9 类分类体系与九大改进原则（含 Thariq 英文原帖）
- [[Skill框架源码分析]] - Anthropic Skill 框架源码拆解（9 个文件），5 种执行模式
- [[Skills-Github化]] - 将 GitHub 开源项目 Skill 化，打造个人超级技能库
- [[Skills-多Skill协作]] - 单一职责拆分 + 三种协作模式（管道/聚合/路由）
- [[Skills-设计哲学]] - 从"中台思维"推导 Skills 本质，"恰好而非更多"的渐进式加载设计哲学
- [[Skills-分类学]] - Skill 颗粒度管理的分类学思维，三条判断标准，数量与准确率关系
- [[Skills-逆向建模]] - 需求迭代实战方法，实体/流程/规则三维度建模
- [[Skill-vs-Prompt]] - Skill 与 Prompt 的 12 项详细对比、选型指南
- [[Skill-设计模式与打磨法]] - 五种 Skill 设计模式（知识注入/模板生成/审查打分/反转采访/流水线）+ 六步评测驱动打磨法 + 售前工时评估实战
- [[Skill测试与质量保障]] - 触发测试集设计（JSON 格式正负例）、三种断言评估、Description 优化四步法、发布前 10 项检查清单
- [[Skill-Creator-官方工作流]] - Anthropic 官方 Skill 创建方法论：意图捕获→并行双轨测试→评估打分→人工反馈→迭代改进完整闭环
- [[Skill-Eval-系统架构]] - 官方评估工程体系：六种 JSON Schema、三种评估智能体（Grader/Comparator/Analyzer）、Eval Viewer
- [[Skills 应用场景-科普]] - 非程序员视角的 Skills 应用
- [[Skills 生态时间线]] - 2023-2026 行业热词演进 + 关键里程碑
- [[MCP]] - Model Context Protocol，统一工具调用协议，含 Claude Code 源码级八种传输协议、Bridge 系统

---

## 多步骤 Skill 工程

- [[多步骤Skill稳定执行]] - 多步骤 Skill 稳定执行完整方法论：三种模式、七阶段流程、四层验证、Supervisor/Worker/Verifier 分工
- [[Output-Contract]] - 输出契约定义与模板，确保最终产物结构稳定不漂移
- [[Checkpoint机制]] - 每步状态快照，支持失败恢复、中断续跑、复盘追溯

---

## 工程实践

- [[约束先行]] - Agent 使用核心心法：行动前先建规范，CLAUDE.md 四层约束体系穿透，非技术视角实战心得
- [[代码执行范式]] - Anthropic 提出的 Agent 新范式，Token 消耗降低 98.7%
- [[Spec-Coding]] - 规范驱动开发（SDD），用三文档结构控制复杂 AI 项目，含 Plan 模式对比
- [[AI编程行为准则]] - Karpathy 四原则，CLAUDE.md 作项目级规则落地（含仓库原文）
- [[AI辅助CodeReview]] - AI-CR 与 Git 闭环运行的工程化实践
- [[Claude-Code-系统化用法]] - 从"聊天"到"调用系统"的 12 种进阶用法，含底层机制深度对照
- [[Token优化技巧]] - 穴居人/文言文模式压缩 output token 50-75%

---

## 参考速查

- [[Claude-Code-功能标志与术语表]] - 12 类工具清单、89 个功能标志速查、50 条核心术语中英对照

---

## Prompt 工程

- [[角色扮演Prompt]] - 三层结构（身份层/经历层/行为约束层）将 Claude Code 从"实习生"升级为"专家顾问"，含 CLAUDE.md 篇幅控制实践
- [[Prompt语言选择]] - 中英混写 vs 纯英文 vs 纯中文，任务类型是决定性变量
- [[横纵分析法]] - 纵向（时间线溯源）× 横向（竞品截面对比）的深度研究 Prompt 框架
- [[Chain-of-Thought]] - 思维链四种实战用法，Google 研究准确率提升 20–40%
- [[query-cot-横纵分析]] - CoT 横纵分析报告
- [[Prompt工程方法论全景]] - 7 种 Prompt 技巧系统对比

---

## 思维方法与创造力

- [[AI时代创造力方法论]] - 六步框架找回创造力：认知失调→约束激发→原型思维→跨领域连接→空白时间→内在动机

---

## 技术写作与创作

- [[花叔写作Skill逆向方法论]] - 从花叔橙皮书逆向拆解写作 Skill 五层架构：信任建立/知识传递/风格DNA/质量铁律/Agent Protocol
- [[技术写作风格DNA]] - 短句规则、第一人称、禁用词表、具体数字等风格控制规则，TDD 式 QC 体系

## 知识梳理存档

- [[有效的-Context-工程-见知录004]] - 见知录 004 万字精读：Context Engineering 方法论 + Anthropic Agent 类型分类
