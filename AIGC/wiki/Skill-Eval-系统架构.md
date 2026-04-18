---
title: Skill Eval 系统架构
tags: [Skill, 评估系统, benchmark, grading, 盲评, JSON Schema]
sources: [~/.claude/skills/skill-creator/references/schemas.md, ~/.claude/skills/skill-creator/agents/grader.md, ~/.claude/skills/skill-creator/agents/comparator.md, ~/.claude/skills/skill-creator/agents/analyzer.md]
updated: 2026-04-17
---

# Skill Eval 系统架构

## 定义

Anthropic 官方 skill-creator 内置的完整评估工程体系。包含六种 JSON 数据格式、三种专用评估智能体（Grader/Comparator/Analyzer）和一个可视化评审工具。核心设计原则：**定量断言与人工定性判断并行，自动打分与盲评归因互补**。

## 六种数据格式全景

| 文件 | 位置 | 职责 | 生产者 |
|------|------|------|--------|
| `evals.json` | `evals/evals.json` | 测试用例定义 | 人工 + AI |
| `eval_metadata.json` | 每个 eval 目录 | 单个测试用例的元数据+断言 | AI |
| `timing.json` | 每个运行目录 | 时间和 token 消耗 | 从子智能体通知捕获 |
| `metrics.json` | `outputs/metrics.json` | 工具调用统计 | 执行器智能体 |
| `grading.json` | 运行目录（outputs 同级） | 断言评估结果 | Grader 智能体 |
| `benchmark.json` | 迭代目录 | 聚合统计对比 | 聚合脚本 + Analyzer |

## evals.json — 测试用例定义

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "用户的任务提示",
      "expected_output": "预期结果描述",
      "files": ["evals/files/sample1.pdf"],
      "expectations": [
        "输出包含 X",
        "使用了脚本 Y"
      ]
    }
  ]
}
```

- `files`：可选，输入文件路径（相对于 Skill 根目录）
- `expectations`：可验证的断言列表，初始可为空，测试运行期间起草

## grading.json — 评分结果（核心）

Grader 智能体的输出，是整个评估体系的核心数据结构。

### 关键字段

| 字段 | 说明 |
|------|------|
| `expectations[]` | 逐条断言评分，必须用 `text`/`passed`/`evidence` 三字段（viewer 依赖精确字段名） |
| `summary` | 聚合统计：passed/failed/total/pass_rate |
| `execution_metrics` | 从 metrics.json 复制的工具调用统计 |
| `timing` | 从 timing.json 复制的时间数据 |
| `claims` | 从输出中提取并验证的隐式声明（事实/过程/质量三类） |
| `user_notes_summary` | 执行器标记的不确定性、需审查项、变通方案 |
| `eval_feedback` | 对评估本身的改进建议（仅在发现明显不足时出现） |

### 评分标准

**PASS**：有明确证据且反映真正的任务完成（非表面合规）

**FAIL**：无证据、证据矛盾、无法验证、或仅表面满足（如文件名正确但内容为空）

> 原则：**举证责任在 PASS 方。** 不确定时判 FAIL。无部分分数。

## benchmark.json — 聚合统计对比

包含三个核心区段：

### metadata
```json
{
  "skill_name": "pdf",
  "executor_model": "claude-sonnet-4-20250514",
  "runs_per_configuration": 3
}
```

### runs[] — 逐次运行结果

```json
{
  "eval_id": 1,
  "eval_name": "Ocean",
  "configuration": "with_skill",  // 必须精确匹配，viewer 靠此分组
  "run_number": 1,
  "result": {
    "pass_rate": 0.85,
    "passed": 6,
    "total": 7,
    "time_seconds": 42.5,
    "tokens": 3800
  },
  "expectations": [...]
}
```

### run_summary — 统计聚合

```json
{
  "with_skill": {
    "pass_rate": {"mean": 0.85, "stddev": 0.05},
    "time_seconds": {"mean": 45.0, "stddev": 12.0},
    "tokens": {"mean": 3800, "stddev": 400}
  },
  "without_skill": { ... },
  "delta": {
    "pass_rate": "+0.50",
    "time_seconds": "+13.0",
    "tokens": "+1700"
  }
}
```

**Schema 严格性警告**：viewer 精确读取字段名。`config` 代替 `configuration`、`pass_rate` 放在 run 顶层而非嵌套在 `result` 下，都会导致 viewer 显示空值。

## 三种评估智能体

### 1. Grader（评分员）

**职责**：评估断言 + 批评评估本身

**八步流程**：
1. 完整阅读转录记录
2. 检查输出文件（不只信转录记录的描述）
3. 逐条评估断言，引用具体证据
4. 提取并验证隐式声明（事实/过程/质量三类）
5. 读取执行器的 user_notes
6. **批评评估本身**：通过性太容易的断言、重要结果无断言覆盖、无法从输出验证的断言
7. 写入 grading.json
8. 合并 metrics.json 和 timing.json

**核心设计**：Grader 有"双重任务"——既打分又审查评估体系。一个弱断言上的 PASS 比什么都不测更危险，因为它制造虚假信心。

### 2. Blind Comparator（盲评对比员）

**职责**：在不知道哪个 Skill 产出了哪个输出的情况下，纯粹基于输出质量判断胜者。

**评分框架**（双维度 × 1-5 分）：

| 维度 | 指标 |
|------|------|
| **内容** | 正确性、完整性、准确性 |
| **结构** | 组织、格式、可用性 |

两个维度平均后映射到 1-10 综合分。如果提供了 expectations，作为辅助证据（非主要判据）。

**判决规则**：
- 主要依据：综合打分
- 次要依据：断言通过率
- 平局极少出现——总有一个更好，哪怕只好一点

### 3. Post-hoc Analyzer（事后分析员）

**职责**："揭盲"后分析 WHY 胜者赢了，生成可执行的改进建议。

**输出结构**：

```json
{
  "comparison_summary": { "winner": "A", ... },
  "winner_strengths": ["清晰的分步指令...", "包含验证脚本..."],
  "loser_weaknesses": ["模糊的指令导致不一致行为...", "无验证脚本..."],
  "instruction_following": {
    "winner": { "score": 9, "issues": [...] },
    "loser": { "score": 6, "issues": [...] }
  },
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "将'适当处理文档'替换为具体步骤",
      "expected_impact": "消除导致不一致行为的歧义"
    }
  ]
}
```

**改进建议六大类别**：

| 类别 | 含义 |
|------|------|
| `instructions` | 指令文本修改 |
| `tools` | 添加/修改脚本或模板 |
| `examples` | 增加示例 |
| `error_handling` | 错误处理指导 |
| `structure` | 内容重新组织 |
| `references` | 补充参考文档 |

优先级分三级：high（可能改变胜负）、medium（提升质量但未必改变胜负）、low（锦上添花）。

## Analyzer 在 Benchmark 场景的角色

Analyzer 在 benchmark 场景下不做改进建议，而是**发现统计聚合隐藏的模式**：

### 五种断言模式

| 模式 | 含义 | 启示 |
|------|------|------|
| 双组全 PASS | 非区分性断言 | 可能不测 Skill 价值 |
| 双组全 FAIL | 超出能力范围 | 断言本身可能有问题 |
| 有 Skill PASS / 无 Skill FAIL | Skill 明确有价值 | 核心价值断言 |
| 有 Skill FAIL / 无 Skill PASS | Skill 可能有害 | 需要调查 |
| 高方差 | 非确定性行为 | 可能是 flaky 断言 |

### 资源分析

- Skill 是否显著增加执行时间？
- token 使用方差是否过高？
- 是否有离群运行拉偏聚合值？

## 评审工具（Eval Viewer）

`generate_review.py` 生成浏览器端评审界面。

### 两个标签页

| 标签 | 内容 |
|------|------|
| **Outputs** | 逐案例查看提示词、输出、（可折叠）上轮输出、打分结果、反馈文本框 |
| **Benchmark** | 通过率/时间/token 对比图，逐评估分解，Analyzer 观察笔记 |

### 关键参数

| 参数 | 用途 |
|------|------|
| `--skill-name` | Skill 名称 |
| `--benchmark` | benchmark.json 路径 |
| `--previous-workspace` | 第 2 轮起，指向上一轮工作区 |
| `--static <path>` | 无浏览器环境时导出为独立 HTML |

## timing.json 的特殊性

子智能体任务完成通知是 `total_tokens` 和 `duration_ms` 的**唯一来源**。通知到达后不持久化到任何其他地方。

**必须**在收到通知时立即写入 timing.json，否则数据永久丢失。这是整个评估体系中唯一有"一次性窗口"约束的数据。

## comparison.json — 盲评结果

```json
{
  "winner": "A",
  "reasoning": "Output A 提供了完整方案...",
  "rubric": {
    "A": { "content_score": 4.7, "structure_score": 4.3, "overall_score": 9.0 },
    "B": { "content_score": 2.7, "structure_score": 2.7, "overall_score": 5.4 }
  },
  "output_quality": {
    "A": { "score": 9, "strengths": [...], "weaknesses": [...] },
    "B": { "score": 5, "strengths": [...], "weaknesses": [...] }
  }
}
```

## history.json — 版本演进追踪

```json
{
  "skill_name": "pdf",
  "current_best": "v2",
  "iterations": [
    { "version": "v0", "parent": null, "expectation_pass_rate": 0.65, "grading_result": "baseline" },
    { "version": "v1", "parent": "v0", "expectation_pass_rate": 0.75, "grading_result": "won" },
    { "version": "v2", "parent": "v1", "expectation_pass_rate": 0.85, "grading_result": "won", "is_current_best": true }
  ]
}
```

每个版本记录父版本、通过率和胜负结果，形成完整的 Skill 演化谱系。

## 设计理念总结

1. **断言 + 人工双轨**：可客观验证的用断言自动化，主观质量留给人
2. **Grader 自省**：评分员同时审查评估本身，避免"弱断言通过 = 虚假信心"
3. **盲评去偏**：Comparator 不知道哪个是新版，防止确认偏误
4. **揭盲归因**：Analyzer 在盲评后才看 Skill 源码，确保先有判断再做归因
5. **timing 一次性窗口**：强调数据的时效性约束，工程纪律的典型体现

## 相关概念

- [[Skill-Creator-官方工作流]] — 评估系统所服务的完整创建迭代流程
- [[Skill测试与质量保障]] — 社区测试方法论（三种断言类型、Description 优化四步法）
- [[Skill-设计模式与打磨法]] — 评测驱动打磨法的宏观框架
- [[多步骤Skill稳定执行]] — 多步骤 Skill 的四层验证体系，可与断言评估配合

## 来源

- `~/.claude/skills/skill-creator/references/schemas.md` — 完整 JSON Schema 定义
- `~/.claude/skills/skill-creator/agents/grader.md` — Grader 智能体规范
- `~/.claude/skills/skill-creator/agents/comparator.md` — Blind Comparator 规范
- `~/.claude/skills/skill-creator/agents/analyzer.md` — Post-hoc Analyzer 规范
