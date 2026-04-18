---
title: Skill vs Prompt
tags: [Skill, Prompt, 对比, 入门, Prompt Engineering]
sources: [raw/AI入门——一文读懂什么是Skill以及如何开发Skill/article.md, raw/Agent Skills 终极指南：入门、精通、预测/article.md]
updated: 2026-04-13
---

# Skill vs Prompt

## 定义

Skill 与 Prompt 的关系定位：**Skill 不是 Prompt 的对立面，而是结构化、可复用、可发现、可按需加载的 Prompt 工程产物**。

核心公式：

> Skill = Prompt + 触发机制 + 资源组织 + 执行约定 + 权限/作用域管理

- Prompt 更偏"内容"
- Skill 更偏"机制 + 容器 + 生命周期"

## 核心观点

- Skill 仍然建立在 Prompt 之上，最终通过上下文注入（system prompt / developer prompt / tool description 等）让模型理解（来源：[AI入门——一文读懂什么是Skill以及如何开发Skill](../raw/AI入门——一文读懂什么是Skill以及如何开发Skill/article.md)）
- 两者都属于广义 Prompt Engineering，Skill 是更工程化和结构化的形态
- 判断标准：**重复、稳定、可流程化的 Prompt，值得沉淀为 Skill**

## 六项相同点

1. 都用于影响模型行为
2. 都依赖上下文注入（不修改模型参数）
3. 都可以包含：角色、背景、任务目标、约束条件、输出格式、示例、注意事项
4. 都会影响输出稳定性
5. 都需要持续迭代优化
6. 都属于 Prompt Engineering 范畴

## 十二项区别

| 维度 | Prompt | Skill |
|------|--------|-------|
| 粒度 | 面向一次请求/单次任务 | 面向一类重复出现的任务 |
| 生命周期 | 临时、一次性 | 长期存在、可复用、可版本管理 |
| 复用方式 | 复制、模板、人工复用 | 客户端自动发现、匹配、加载、注入 |
| 触发方式 | 用户直接输入 | 自动触发或手动显式调用 |
| 结构化程度 | 自由文本 | 固定结构（SKILL.md + scripts/ + references/ + assets/） |
| 工程集成 | 偏"和模型对话" | 偏"Agent Runtime 中的能力模块" |
| 可发现性 | 不会被系统主动发现和管理 | 被客户端扫描、索引，暴露给模型选择 |
| 权限管理 | 一般无独立权限体系 | 可单独控制访问权限（allow/deny/ask） |
| 资源承载 | 主要承载文本 | 可附带脚本、模板、参考文档、静态资源 |
| 执行方式 | 模型读完直接回答 | 模型读完后调用工具/脚本/参考资料完成任务 |
| 适用场景 | 探索性、临时性、开放性任务 | 重复性、流程化、专业化任务 |
| 维护方式 | 个人使用、分散维护 | 团队共享、版本控制、协作演进 |

## 选型指南

### 适合直接写 Prompt

- 一次性任务
- 探索性需求
- 需求尚不稳定
- 快速试验输出效果
- 没有长期复用价值

### 适合沉淀为 Skill

- 某类任务经常重复出现
- 工作步骤比较固定
- 对输出一致性要求高
- 需要配套脚本、模板或参考资料
- 希望团队共享和版本化维护

## 通俗比喻（来源：新员工类比）

> 想象你开了一家公司，AI 是你招来的新员工。

- **Prompt** = 口头指令："帮我写个方案""语气正式一点"——说完就没了，关对话框就失忆
- **Skills** = 岗位手册：含工作流程、输出模板、质量标准、参考案例、工具脚本，可传承给下一个员工
- **MCP** = 门禁卡：进不了数据库、打不开文件柜、连不上 API，就是白搭

**一句话总结**：Prompt 是嘴巴，Skills 是大脑里的知识，MCP 是手和脚。

Skills 与 Prompt 模板的根本区别在于"活"：
- 上下文感知——AI 根据当前任务自动判断该不该调用
- 工具链——可以调脚本、读文件、连 API
- 协作——写作 Skill 写完初稿，排版 Skill 自动接手

> "这就像一个存在手机备忘录里的菜谱，和美团上一键下单的区别。"

## Skills 的连续体：从 Prompt 到 Workflow

Skills 不是 Prompt 和 Workflow 的二选一，而是覆盖了从 Prompt 到 Workflow 的整个光谱：

> **Skills 慢起来可以是 prompt，快起来也可以是 workflow。**

- **Prompt 端**：Skill 包含大量 tokens 的指令文档，引导模型推理思考
- **Workflow 端**：Skill 指向可直接运行的脚本代码，Agent 只承担类似 hook 的角色，实质上和正常程序运行并无差别

这是因为 Skills 能直接调用代码逻辑（代码不进 Context 窗口），不需要 Agent 一直推理。结合 token 价格下降和 agent 速度提升的趋势，以 Skills 为基础的垂直 Agent 在性能和开销上的问题并非持续性障碍。

**对 AI 产品设计的影响**：Skills-based 的 Agent 产品能用同一个多模态输入框，处理用户各种不同的输入，灵活应对未被规划的边缘问题，为用户提供个性化生成需求。

## 相关概念

- [[Skill定义规范]] — Skill 的定义、SKILL.md 标准写法与安全规范
- [[Skill框架源码分析]] — 框架架构、五种执行模式
- [[Skills-渐进式披露]] — Skill 三级加载机制（元数据/指令/资源），渐进式披露详解
- [[Prompt语言选择]] — Prompt 语言选择对输出质量的影响
- [[Chain-of-Thought]] — Prompt 层面的推理增强技术
- [[Skills-设计哲学]] — "公共的 Prompt 就是 Skills"的推导逻辑与渐进式加载理念
- [[Harness-Engineering]] — Skills 的连续体设计是 Harness 精细化控制的具体体现

## 来源

- [AI入门——一文读懂什么是Skill以及如何开发Skill](../raw/AI入门——一文读懂什么是Skill以及如何开发Skill/article.md) — 程序猿某人，程序猿不加班，2026-03-30
- [一文带你看懂，火爆全网的Skills到底是个啥](../raw/一文带你看懂，火爆全网的Skills到底是个啥/article.md) — Lumilous，爱AI的大刘，2026-02-11
- [Agent Skills 终极指南：入门、精通、预测](../raw/Agent%20Skills%20终极指南：入门、精通、预测/article.md) — 一泽Eze，Skills 的连续体设计、渐进式披露、对 AI 产品设计的影响
