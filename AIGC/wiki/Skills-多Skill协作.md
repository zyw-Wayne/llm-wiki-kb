---
title: Skills 多 Skill 协作设计
tags: [Skills, Agent, 架构, 协作模式, 单一职责]
sources: [raw/一个人管不了所有事——如何让多个 Skill 协同工作/article.md]
updated: 2026-04-13
---

# Skills 多 Skill 协作设计

## 定义

当任务复杂到单个 Skill 无法独立完成时，通过单一职责拆分 + 协作模式编排，让多个小 Skill 像流水线一样协同完成大任务。核心原则：**一个 Skill，只做一件事。**

## 为什么不能把所有功能塞进一个 Skill？

| 问题 | 说明 |
|------|------|
| 上下文成本 | Skill 内容注入系统提示，越大占用上下文越多，挤压有效信息空间 |
| 触发混乱 | 职责越多 description 越难精准，导致触发过宽或无法触发 |
| 维护困难 | 出问题时难以定位，拆分后哪里出问题改哪里 |

## 单一职责自测法

> 用一句话描述这个 Skill 是干什么的，这句话里有没有「和」这个字？如果有——拆。

```
搜索新闻 和 生成摘要  →  拆成两个 Skill
整理文件 和 发送通知  →  拆成两个 Skill
```

## 三种协作模式

### 模式一：串行管道（Pipeline）

A 的输出作为 B 的输入，依次传递。

```
用户请求
    ↓
[search-news]  搜索原始信息
    ↓
[summarize]    提炼摘要
    ↓
[format-brief] 格式化输出
    ↓
用户收到简报
```

**实现关键**：不需要显式写「调用 X Skill」，AI 会根据能力描述自动整合。重要的是把每一步的**输入输出说清楚**。

### 模式二：并行聚合（Fan-out / Fan-in）

多个 Skill 同时处理不同维度，最后汇总。

```
用户请求竞品分析
    ↓
[search-A] [search-B] [search-market]  ← 并行执行
    ↓
[compare-report]  汇总对比报告
```

**适用场景**：需要从多个来源收集信息再综合判断（竞品分析、多维度评估）。

### 模式三：条件路由（Conditional Routing）

根据用户输入意图，路由到不同 Skill 处理。

```
用户输入
    ↓
[router-skill]  判断意图
    ↙         ↘
[skill-A]    [skill-B]
写作类请求    搜索类请求
```

**实现方式**：写一个专门的「路由 Skill」，唯一职责是判断意图并引导处理方向，触发条件设为「其他 Skill 均未匹配时激活」。

## 避免 Skill 之间的「内耗」

两个 Skill 的 description 描述相似场景，AI 不知道用哪个，结果两个都不用或用错。

**检查方法**：把所有 Skill 的 description 列成表，横向对比触发场景是否重叠。若两个 Skill 触发场景完全重叠——**合并它们**。

## 用 MEMORY.md 传递跨 Skill 状态

Skill 之间默认无状态，可用 `MEMORY.md` 作为「共享黑板」：

```markdown
## 当前上下文
活跃项目：OpenClaw Skill 开发
当前阶段：调试触发问题
最近更新：2026-03-27
```

每个相关 Skill 在执行步骤中加：
1. 读取 MEMORY.md 中的「当前上下文」
2. 基于当前背景处理请求
3. 最后：如有必要，更新 MEMORY.md

## 完整示例：每日 AI 内参系统

```
daily-brief        主 Skill，负责编排整个流程
  └─ search-ai     搜索 AI 行业最新动态
  └─ search-tools  搜索新工具发布信息
  └─ summarize     对每条信息提炼摘要
  └─ format-brief  套用简报模板格式化输出
```

## 核心原则总结

| 原则 | 具体做法 |
|------|---------|
| 单一职责 | 一个 Skill 只做一件事，描述里没有「和」 |
| 串行管道 | 明确每步的输入输出，让结果自然流转 |
| 并行聚合 | 多路收集信息，最后汇总到一个出口 |
| 条件路由 | 写路由 Skill 判断意图，不要让 AI 猜 |
| 避免内耗 | description 触发场景互相排他，不重叠 |
| 共享状态 | 用 MEMORY.md 作为跨 Skill 的黑板 |

## 相关概念

- [[Skill定义规范]] — Skill 定义、SKILL.md 标准写法与安全规范
- [[Skill实战踩坑指南]] — Anthropic 数百个 Skill 实战踩坑经验，9 类分类与九大改进原则
- [[Skills-Github化]] — Skill 的构建方法：将 GitHub 开源项目封装为 Skill
- [[代码执行范式]] — Agent 通过代码调用工具的技术基础
- [[Spec-Coding]] — 规范驱动开发，复杂系统的另一种控制手段
- [[Skills-逆向建模]] — 需求迭代场景下的实体/流程/规则建模，复杂需求可拆分为多 Skill 协同
- [[Skills-设计哲学]] — 渐进式披露在多 Skill 场景的实现，单一职责的设计哲学基础
- [[AI辅助CodeReview]] — AI-CR 流程中多 Skill 协作的实践案例

## 来源

- [一个人管不了所有事——如何让多个 Skill 协同工作](../raw/一个人管不了所有事——如何让多个 Skill 协同工作/article.md) — 单一职责原则、三种协作模式、MEMORY.md 共享状态
