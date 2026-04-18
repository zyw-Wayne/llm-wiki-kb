---
title: Skill Creator 官方工作流
tags: [Skill, Claude Code, 官方文档, 创建流程, 迭代, 评估]
sources: [~/.claude/skills/skill-creator/SKILL.md]
updated: 2026-04-17
---

# Skill Creator 官方工作流

## 定义

Anthropic 官方 skill-creator 工具定义的完整 Skill 创建方法论。核心是一个**评估驱动的迭代闭环**：草稿 → 并行测试（有Skill vs 无Skill基线）→ 定量+定性评估 → 人工反馈 → 改进 → 重复，直到用户满意。

与社区方法论的关键区别：官方流程强调**子智能体并行执行**、**盲评对比**和**自动化描述优化**，工程化程度远高于手工迭代。

## 核心闭环

```
意图捕获 → 访谈研究 → 撰写草稿 → 并行测试运行
    ↑                                    ↓
    ←── 改进 Skill ← 读取反馈 ← 人工评审 ← 评估打分
```

## 阶段一：意图捕获（Capture Intent）

四个核心问题：

1. **做什么**：这个 Skill 要让 Claude 能做什么？
2. **何时触发**：用户会说什么话/在什么场景下触发？
3. **输出格式**：期望的输出是什么样的？
4. **是否需要测试集**：客观可验证的输出（数据提取、代码生成）适合测试；主观输出（写作风格）通常不需要

如果当前对话已包含工作流（用户说"把这个变成 Skill"），优先从对话历史提取答案：使用的工具、步骤顺序、用户的修正、输入输出格式。

## 阶段二：访谈与研究（Interview & Research）

主动询问：边界情况、输入输出格式、示例文件、成功标准、依赖项。

**先完成访谈，再写测试用例。** 检查可用 MCP 服务器，研究相关文档和最佳实践。

## 阶段三：撰写 SKILL.md

### 目录结构

```
skill-name/
├── SKILL.md          # 必须，核心指令
└── Bundled Resources  # 可选
    ├── scripts/       # 确定性/重复性任务的可执行脚本
    ├── references/    # 按需加载的补充文档
    └── assets/        # 输出模板、图标、字体等
```

### 渐进式披露三层模型

| 层级 | 内容 | 加载时机 | 预算 |
|------|------|---------|------|
| L1 元数据 | name + description | 始终在上下文 | ~100 词 |
| L2 指令 | SKILL.md body | 触发时加载 | <500 行（理想值） |
| L3 资源 | references/scripts/assets | 按需加载 | 无限制，脚本可不加载直接执行 |

**关键规则**：SKILL.md 接近 500 行时，增加层级并写清"下一步去哪里找"的指针。大参考文件（>300 行）需加目录。

### 多领域/多框架组织

```
cloud-deploy/
├── SKILL.md          # 工作流 + 选择逻辑
└── references/
    ├── aws.md        # Claude 只读相关的那个
    ├── gcp.md
    └── azure.md
```

### Description 写法原则

官方立场：**Claude 当前倾向于"不触发"（undertrigger）**，因此 description 应该写得"稍微激进一些"（a little bit pushy）。

> 不要写：`How to build a simple fast dashboard to display internal Anthropic data.`
> 要写：`How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'`

### 写作风格哲学（四条核心原则）

1. **解释 Why 而不是堆 MUST**：今天的 LLM 足够聪明，理解了原因比遵守死规则更有效。如果发现自己在写全大写的 ALWAYS/NEVER，这是黄旗——试着重构为"解释为什么这件事重要"
2. **用理论-of-mind**：让 Skill 通用化，不要过度拟合到特定示例
3. **写完草稿后用新鲜眼光重审**
4. **使用祈使语气**（imperative form）

## 阶段四：测试运行

### 并行双轨测试

每个测试用例**同时**派出两个子智能体：

| 轨道 | 新建 Skill 场景 | 改进 Skill 场景 |
|------|----------------|----------------|
| **实验组** | 带 Skill 运行（with_skill） | 带新版 Skill 运行 |
| **基线组** | 不带 Skill 运行（without_skill） | 带旧版 Skill 运行（先快照） |

**关键**：在同一轮（same turn）启动所有运行，让它们差不多同时完成。不要先跑实验组再回来跑基线。

### 工作区目录结构

```
<skill-name>-workspace/
└── iteration-1/
    ├── eval-descriptive-name/
    │   ├── eval_metadata.json
    │   ├── with_skill/
    │   │   ├── outputs/
    │   │   ├── grading.json
    │   │   └── timing.json
    │   └── without_skill/
    │       ├── outputs/
    │       ├── grading.json
    │       └── timing.json
    └── benchmark.json
```

### 时间利用

运行期间不空等——**立刻起草断言（assertions）**。好断言的标准：
- 客观可验证
- 名字描述性强（在 benchmark viewer 里一眼看懂）
- 主观维度（写作风格、设计质量）留给人工评审，不强塞断言

### 捕获 timing 数据

子智能体完成时通知中包含 `total_tokens` 和 `duration_ms`。**必须立即保存**——这是唯一的捕获窗口，之后无法恢复。

## 阶段五：评估

### 四步评估流程

1. **打分**：Grader 智能体逐条评估断言（详见 [[Skill-Eval-系统架构]]）
2. **聚合**：`aggregate_benchmark` 脚本生成 benchmark.json（含均值±标准差、delta）
3. **分析**：Analyzer 智能体发现聚合统计隐藏的模式（非区分性断言、高方差评估、时间/token 权衡）
4. **可视化**：`generate_review.py` 生成浏览器评审工具，包含 Outputs（逐案例查看+反馈文本框）和 Benchmark（定量对比）两个标签页

### 评审工具交互

- 左右箭头键导航测试用例
- 每个用例可写文字反馈（自动保存）
- 第 2 轮迭代起可折叠查看上一轮输出和反馈
- 点击"Submit All Reviews"导出 `feedback.json`

### 读取反馈

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "图表缺少坐标轴标签", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."}
  ]
}
```

**空反馈 = 用户觉得没问题。** 聚焦有具体意见的用例。

## 阶段六：改进（核心）

官方四条改进哲学：

### 1. 从反馈中泛化

Skill 要被使用无数次，但迭代只在少量示例上进行。不要做过拟合的微调或堆叠 MUST，而是尝试不同的表达隐喻、推荐不同的工作模式。**试错成本低，可能碰到好方案。**

### 2. 保持精简

删掉没有产出的部分。读转录记录（transcript），不只看最终输出——如果 Skill 让模型浪费大量时间在无效操作上，砍掉导致这种行为的指令段落。

### 3. 解释 Why

理解用户反馈背后的真实意图，把这个理解传达到指令中。比起写死规则，解释原因更人性、更强大、更有效。

### 4. 发现重复劳动

阅读所有测试运行的转录记录——如果多个子智能体独立写了类似的辅助脚本（比如都写了 `create_docx.py`），就把它提取到 `scripts/` 目录，避免每次调用都重新发明轮子。

### 迭代终止条件

- 用户明确满意
- 所有反馈为空
- 改进不再有实质进展

## 阶段七：描述优化（Description Optimization）

Skill 功能完善后，用自动化流程优化触发准确率。

### 流程

1. **生成 20 条触发测试查询**（8-10 正例 + 8-10 负例）
2. **用户在 HTML 评审工具中审核**（可编辑、增删、切换 should_trigger）
3. **运行自动优化循环**：`run_loop.py` 自动 60/40 划分训练/测试集，每条跑 3 次取可靠触发率，Claude 提出改进，迭代最多 5 轮
4. **应用最佳描述**：按测试集得分（非训练集）选最佳版本，防止过拟合

### 测试查询设计原则

**必须写得像真实用户**：包含文件路径、个人背景、列名、公司名、URL、口语缩写、拼写错误。

> ❌ `"Format this data"`, `"Extract text from PDF"`
> ✅ `"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage..."`

**负例要是"近似未命中"**：与 Skill 共享关键词但实际需要不同处理。`"Write a fibonacci function"` 作为 PDF Skill 的负例太简单——没有测试价值。

### 触发机制原理

Skill 以 name + description 出现在 Claude 的 `available_skills` 列表中。**Claude 只对自己无法轻松处理的任务才查阅 Skill**——简单的一步操作（如"读这个 PDF"）即使 description 匹配也可能不触发。测试查询要足够复杂，让 Claude 觉得"查阅 Skill 会有帮助"。

## 高级：盲评对比

当需要严格验证"新版是否真的更好"时使用。

1. **Blind Comparator**：给两个输出标记为 A/B，不告知来源，纯粹基于输出质量判断胜者
2. **Post-hoc Analyzer**："揭盲"后分析 WHY 胜者赢了——指令清晰度差异、脚本使用差异、错误处理差异

可选步骤，大多数场景人工评审足够。

## 不同环境适配

| 环境 | 子智能体 | 浏览器 | 基线对比 | 描述优化 |
|------|---------|--------|---------|---------|
| Claude Code | ✅ 并行 | ✅ | ✅ | ✅ `run_loop.py` |
| Claude.ai | ❌ 串行自跑 | ❌ 对话内展示 | ❌ 跳过 | ❌ 需 CLI |
| Cowork | ✅ 并行 | `--static` 导出 HTML | ✅ | ✅ |

## 与社区方法论的对比

| 维度 | 官方 skill-creator | 社区六步打磨法 |
|------|-------------------|---------------|
| **基线对比** | 必须，每轮都跑有/无 Skill 两组 | 可选，裸跑一次建基线 |
| **测试执行** | 子智能体并行自动执行 | 手动或对话模拟 |
| **评估工具** | 浏览器 viewer + benchmark.json | 人工 1-5 分评分 |
| **断言系统** | JSON 格式 + Grader 智能体自动打分 | 三种断言类型（contains/not_contains/format） |
| **描述优化** | 自动化训练/测试划分 + 5 轮迭代 | 四步诊断法手工优化 |
| **盲评** | 可选的 A/B 盲评 + Analyzer 归因 | 无 |
| **适用人群** | 需要 Claude Code CLI 环境 | 零代码门槛，对话即可 |

两者不矛盾：社区方法论是轻量级起步方案，官方工具是工程化进阶方案。

## 相关概念

- [[Skill-Eval-系统架构]] — 官方评估系统的完整 JSON Schema 与三种评估智能体
- [[Skill定义规范]] — SKILL.md 标准写法、description 三原则、完整 frontmatter 字段
- [[Skill测试与质量保障]] — 社区测试方法论，与官方流程互补
- [[Skill-设计模式与打磨法]] — 社区五种设计模式 + 六步打磨法
- [[Skill实战踩坑指南]] — Anthropic 内部九大改进原则
- [[Skills-渐进式披露]] — 三级加载机制的详细分析

## 来源

- `~/.claude/skills/skill-creator/SKILL.md` — Anthropic 官方 skill-creator 内置 Skill，Claude Code 自带
