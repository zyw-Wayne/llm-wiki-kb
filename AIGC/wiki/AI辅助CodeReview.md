---
title: AI 辅助 Code Review
tags: [Skill, Code Review, Git, 团队协作, 工程化]
sources: [raw/一文搞懂爆火的SKills原理及实践案例/article.md]
updated: 2026-04-13
---

# AI 辅助 Code Review

## 定义

将 AI-CR（AI 辅助代码审查）与 Git 工作流闭环运行的工程化实践，包含自动化 CR 和人工 CR 两种模式。

## 核心实践

### 自动化 CR

AI-CR 与 Git 闭环运行：代码提交后自动触发 AI 审查，审查结果回流到 Git 工作流中。

### 人工 CR 增强

- AI 辅助人工 CR 是最高频场景
- 结合 [[Skills-逆向建模|逆向建模]] 思路：**把设计图和代码 Diff 一起提交到 MR**
- 效果：大幅降低 Code Review 的理解成本，提升团队效能

## 关键技巧

- 设计图 + 代码 Diff 一同入 MR，reviewer 不需要在脑中重建设计意图
- AI 可以自动检查 Diff 是否与设计图一致

## 与其他概念的关系

- [[Skill实战踩坑指南]] — Code Review 属于"代码质量与审查"类 Skill
- [[Skills-逆向建模]] — CR 中结合建模图纸，降低理解成本
- [[Skill定义规范]] — 可将 CR 流程沉淀为标准化 Skill

## 来源

- [一文搞懂爆火的SKills原理及实践案例](../raw/一文搞懂爆火的SKills原理及实践案例/article.md) — 吕昊俣，腾讯云开发者，2026-03-13
