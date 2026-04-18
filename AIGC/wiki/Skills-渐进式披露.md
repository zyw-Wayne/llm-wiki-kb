---
title: Skills 渐进式披露
tags: [skills, context-engineering, 渐进式披露, 上下文管理, token优化]
sources: [raw/Agent Skills 终极指南：入门、精通、预测/article.md, raw/Agent Skills 完全指南：从原理到实战彻底搞懂！/article.md]
updated: 2026-04-14
---

# Skills 渐进式披露

## 定义

Skills 渐进式披露（Progressive Disclosure）是 Agent Skills 的核心运行机制——Skill 内容按优先级分层加载，避免一次性全量注入导致上下文过长、模型能力下降。这是 [[Context-Engineering]] 在 Skill 领域的具体实现。

## 三级加载机制

| 层级 | 内容 | 加载时机 | Token 占用 |
|------|------|---------|-----------|
| **Level 1** | SKILL.md 元数据（name + description） | Agent 启动时始终加载，包含在系统提示中 | ~100 tokens |
| **Level 2** | SKILL.md 完整正文（工作流程、最佳实践） | 用户消息与元数据匹配时触发加载 | 建议 <5000 tokens |
| **Level 3** | 子技能文档、代码脚本、参考文档、资源文件 | 按需动态读取 | 无限制 |

### 加载流程

```
Level 1: SKILL.md 元数据（name + description）→ Agent 启动时加载
    ↓ 用户消息匹配
Level 2: SKILL.md 完整内容 → 触发时 bash 读取
    ↓ 任务执行需要
Level 3: Resources 中的具体文件 → 按需读取
```

### 关键设计

- **Level 1 默认只加载元数据** → 可给 Agent 同时安装很多 Skills 但不影响上下文性能
- **Level 2 触发式加载** → Agent 通过元数据判断是否需要调用，避免无谓占用
- **Level 3 按需加载** → 文件在被访问前不占 Context 长度，无大小限制
- **Scripts 代码脚本**：Agent 直接调用脚本执行，脚本代码本身不进 Context Window，只有执行后的输出进入

## 图书馆类比

渐进式披露可以类比为在图书馆查资料的三个步骤（来源：[Agent Skills 完全指南](../raw/Agent%20Skills%20完全指南：从原理到实战彻底搞懂！/article.md)）：

1. **先看目录**（Level 1 元数据）：系统启动时只加载每个 Skill 的名字和简短描述，几百 Token，让 Agent 知道"自己会什么"
2. **翻开手册**（Level 2 指令）：用户说"帮我处理 Excel"，Agent 匹配到对应 Skill，才去读取完整 SKILL.md
3. **动手干活**（Level 3 运行时资源）：执行具体步骤时按需读取 reference 文档和调用 scripts，脚本代码本身不进上下文

### 与 MCP 的 Token 开销对比

| 维度 | MCP（300 工具） | Skills（40 个） |
|------|----------------|----------------|
| 初始加载 | 所有工具完整 schema 一次性注入，数万 Token | 仅元数据（name + description），~几千 Token |
| 执行时 | 无额外加载（已全量注入） | 按需加载指令和资源 |
| 注意力 | 面对几百个工具容易"分心" | 漏斗式引导，每次只聚焦当前任务 |

> 即使是能力稍弱的模型，在渐进式披露的漏斗引导下也能保持极高的调用准确率。

## 与其他上下文管理策略的关系

渐进式披露是 Harness Engineering 中 **上下文架构（Context Architecture）** 支柱在 Skill 层的具体体现。对比：

| 策略 | 范围 | 对应 |
|------|------|------|
| Harness 上下文分层 | Agent 级别（会话常驻/按需加载/持久化知识库） | [[Harness-Engineering]] 支柱一 |
| Skill 渐进式披露 | Skill 级别（元数据/指令/资源） | 本页 |
| Context 压缩 | 对话级（超过阈值总结中间轮次） | [[Context-Engineering]] |

## 工程挑战

即使 Agent Skill 支持渐进式披露，在商业化 Agent 产品中，单个或多个 Skills 联用时，如何稳定控制运行过程中的 Context 长度，依然是绕不过的工程问题。

## 相关概念

- [[Context-Engineering]] — 渐进式披露是 Context 工程的 Skill 层实现
- [[Skill定义规范]] — SKILL.md 的元数据格式是渐进式披露 Level 1 的载体
- [[Skill实战踩坑指南]] — 实战中如何优化 Skill 的 Context 占用
- [[Token优化技巧]] — 渐进式披露与 Token 压缩的协同关系
- [[Harness-Engineering]] — 上下文架构支柱的全局视角
- [[Skills-设计哲学]] — "恰好而非更多"的设计理念与渐进式披露一脉相承

## 来源

- [Agent Skills 终极指南：入门、精通、预测](../raw/Agent%20Skills%20终极指南：入门、精通、预测/article.md) — 渐进式披露三级机制详解，Level 1/2/3 的加载时机与内容边界
- [万字干货！Agent Skills从入门到精通](../raw/万字干货！Agent Skills从入门到精通/article.md) — 冷逸，2026-04-13，汉堡店类比生动阐释三层加载机制
- [Agent Skills 完全指南：从原理到实战彻底搞懂！](../raw/Agent%20Skills%20完全指南：从原理到实战彻底搞懂！/article.md) — ConardLi，2026-01-27，图书馆类比、MCP Token 开销对比数据
