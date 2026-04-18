---
title: Output Contract（输出契约）
tags: [Output Contract, 契约, 输出稳定性, schema, Fail-Closed, Skill]
sources: [raw/多步骤 Skill 稳定执行手册/article.md]
updated: 2026-04-16
---

# Output Contract（输出契约）

## 定义

Output Contract 是多步骤 Skill 中对**最终输出的固定约束**，确保执行链路再复杂，最后产物也不会"每次长得不一样"。是 v4.0 执行方法论中最核心的概念之一。

## 核心观点

- 必须定义：字段结构、字段类型、单位、精度、排序、时间范围、去重口径、空值语义、错误语义（来源：[多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md)）
- 不冻结 Output Contract 的后果：今天按"元"输出，明天别人按"万元"输出，结果必然不稳定
- 执行前冻结 Output Contract 是稳定执行的第一道防线

## Final Output Schema

推荐最小结构：
```json
{
  "status": "success | partial_success | failed",
  "summary": "一句话结论",
  "result": {},
  "artifacts": [],
  "warnings": [],
  "errors": [],
  "meta": {
    "run_id": "string",
    "skill_version": "string",
    "schema_version": "string",
    "contract_version": "string"
  },
  "next_action": "string"
}
```

## Output Contract 模板

```yaml
output_contract:
  schema_version: 1.0.0
  timezone: Asia/Shanghai
  date_format: YYYY-MM-DD
  amount_unit: yuan
  ratio_precision: 2
  sort_order: region asc
  null_policy:
    missing: null
    empty_list: []
  time_range: <具体范围>
```

## 输出前 6 个检查

1. 结构对不对
2. 字段齐不齐
3. 单位和精度对不对
4. 汇总值是否可复算
5. summary 是否准确
6. artifacts 是否真实存在

## 相关概念

- [[多步骤Skill稳定执行]] — Output Contract 是多步骤 Skill 七阶段流程中 Phase 2 的核心产物
- [[Checkpoint机制]] — Checkpoint 中应记录 Output Contract 的当前验证状态；验证不通过时必须 fail-closed
- [[Skill定义规范]] — Output Contract 可嵌入 SKILL.md 作为执行约束

## 来源

- [多步骤 Skill 稳定执行手册](../raw/多步骤 Skill 稳定执行手册/article.md) — Output Contract 定义、模板与验证流程
