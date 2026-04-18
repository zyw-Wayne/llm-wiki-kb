---
title: Checkpoint 机制（多步骤执行状态快照）
tags: [Checkpoint, 状态恢复, 断点续跑, Skill, 执行稳定性]
sources: [raw/多步骤 Skill 稳定执行手册/article.md]
updated: 2026-04-16
---

# Checkpoint 机制（多步骤执行状态快照）

## 定义

Checkpoint 是多步骤 Skill 执行过程中，每步完成后保存的状态快照。使失败后能恢复、中断后能续跑、复盘时能追溯。

## 核心观点

- 没有 Checkpoint 的 7 步流程跑到 Step 5 挂了，只能从 Step 1 全部重跑（来源：[多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md)）
- 每步完成后的必做动作：更新进度、保存 Checkpoint、记录证据、标记当前状态、进入下一步

## Checkpoint 必须包含的字段

| 字段 | 含义 |
|------|------|
| 当前步骤 | 执行到第几步 |
| 当前状态 | success / in_progress / failed |
| 已产生产物 | 生成了哪些文件或数据 |
| 关键返回值摘要 | 工具调用的核心返回数据 |
| 下一步依赖 | 后续步骤需要什么前置条件 |
| 产物 TTL | 产物有效期 |

## Checkpoint 示例

```json
{
  "run_id": "2026-04-13-001",
  "skill": "daily-report",
  "current_step": 4,
  "total_steps": 6,
  "step_results": {
    "step_1": {"status": "success"},
    "step_2": {"status": "success", "artifacts": {"raw": "/tmp/source_a.json"}},
    "step_3": {"status": "success", "artifacts": {"raw": "/tmp/source_b.json"}},
    "step_4": {"status": "in_progress"}
  },
  "last_updated": "2026-04-13T18:00:00+08:00"
}
```

## 相关概念

- [[多步骤Skill稳定执行]] — Checkpoint 是 Phase 5 Execute 的核心机制
- [[Output-Contract]] — Checkpoint 中应记录 Output Contract 的当前验证状态

## 来源

- [多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md) — Checkpoint 定义、字段、示例与使用场景
