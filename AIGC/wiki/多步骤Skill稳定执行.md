---
title: 多步骤 Skill 稳定执行
tags: [Skill, 多步骤, 执行稳定性, Supervisor, Worker, Verifier, Output Contract, Checkpoint, 验证]
sources: [raw/多步骤 Skill 稳定执行手册/article.md]
updated: 2026-04-16
---

# 多步骤 Skill 稳定执行

## 定义

一套让多步骤 Skill 从"能跑"升级到"稳定跑"的系统方法论。核心理念：**把一系列可验证、可恢复、可审计的动作稳定地串起来**，而非简单地把很多动作串起来。

## 核心观点

- 五个最常见的失败根源：步骤定义太粗、缺少中间验证、上下文丢失、没有 Checkpoint、缺少监督角色（来源：[多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md)）
- 单步骤任务核心是"做出来"，多步骤任务核心是"每一步都做对，而且后面的人知道前面的人到底做了什么"
- 执行稳定不等于输出稳定——流程能跑完但字段缺失、排序变化、单位漂移仍是不稳定

## 三种执行模式

| 模式 | 适用场景 | 特征 |
|------|---------|------|
| 模式 A：单 Agent 顺序执行 | 步骤 < 3，确定性动作 | 确认→执行→校验→完成 |
| 模式 B：Supervisor+Worker+Verifier | 步骤 ≥ 3，有中间产物 | 需要重试、恢复、验收 |
| 模式 C：工程级工作流 | 软件开发、大型改造 | workflow-plan→issue→execute→verify |

决策树：步骤数 < 3 → 模式 A；是软件工程任务 → 模式 C；其余 → 模式 B

## 完整执行流程（七阶段）

1. **Confirm**：确认意图，防止一开始就做错
2. **Freeze Contract**：冻结 Execution Contract（怎么做）+ Output Contract（交什么）— v4.0 最强调的一步
3. **Create Plan**：拆原子步骤，配依赖、验证方式、超时、并行性
4. **Approval Gate**：有副作用的动作之前必须过门
5. **Execute**：每步完成后立即保存 Checkpoint、记录证据、更新进度
6. **Validate**：四层验证（L1 工具返回 → L2 步骤契约 → L3 业务逻辑 → L4 最终输出）
7. **Review & Complete**：输出结论、保存报告、清理临时状态

## 四层验证体系

| 层级 | 验证什么 | 判断依据 |
|------|---------|---------|
| L1 | 工具返回验证 | HTTP 状态码、exit code、ok 字段 |
| L2 | 步骤契约验证 | 输出是否满足 schema |
| L3 | 业务逻辑验证 | 创建后能读到、聚合值正确 |
| L4 | 最终输出验证 | 满足 Output Contract |

## Supervisor / Worker / Verifier 分工

| 角色 | 核心职责 | 最怕做错什么 |
|------|---------|-------------|
| Supervisor | 发指令、验收、纠偏、收口 | 把"还在跑"当"推进正常" |
| Worker | 执行动作、生成产物、写进度 | 跳过契约、跳过验证 |
| Verifier | 独立判断结果是否达标 | 用主观感觉代替证据 |

关键原则：验收不过时推回同一个 Worker session（不新开），只允许 `accepted` 或 `failed` 两种终态。

## 三种终态与 Fail-Closed

- `success`：关键步骤和关键字段都通过
- `partial_success`：有结果但不完整或有关键限制
- `failed`：无法产出可信结果

**必须 fail-closed 的情况**：关键字段缺失、关键验证未通过、口径不明确、汇总值与明细不一致、数据源冲突未解决。

## 最小产物集合

```
reports/<slug>/task-brief.md   # 任务简报
reports/<slug>/progress.md     # 进度追踪
reports/<slug>/review.md       # 验收记录
reports/<slug>/final-output.json  # 最终输出
```

## 相关概念

- [[Tool-vs-Skill-vs-SubAgent选型]] — 多步骤 Skill 属于 Skill 范畴，复杂到需要独立上下文时升级为 Sub-Agent
- [[Coordinator模式]] — Supervisor 角色与 Coordinator 的协调者职责类似，均负责分配与验收
- [[Skill定义规范]] — 多步骤 Skill 是 Skill 定义规范在复杂场景下的工程化延伸
- [[SubAgent-与-Fork模式]] — Supervisor/Worker 架构可借助 SubAgent 实现上下文隔离
- [[Skills-多Skill协作]] — 多步骤 Skill 的管道模式与多 Skill 协作的管道模式一脉相承
- [[Plan模式与结构化工作流]] — 模式 C 工程级工作流与 Plan 模式的结构化执行理念相通

## 来源

- [多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md) — 融合 v2.0 与 v3.0 的完整方法论，含三种模式、七阶段流程、四层验证
