---
title: AI 编程行为准则（Karpathy 四原则）
tags: [AI编程, CLAUDE.md, 行为约束, Karpathy, Claude-Code, 人机分工]
sources: [raw/7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包，普通人也能写出大神级代码/article.md, raw/Karpathy-Inspired Claude Code Guidelines/article.md]
updated: 2026-04-15
---

# AI 编程行为准则（Karpathy 四原则）

## 背景

Andrej Karpathy 吐槽 AI 编程的四个顽疾：
> "模型会自作主张做出错误假设，然后一路错下去。它们不会管理困惑，不会寻求澄清，该反驳的时候不反驳，100行能解决的问题，要写1000行。"

项目 `forrestchang/andrej-karpathy-skills` 将这一观察直接转化为 `CLAUDE.md` 行为规则，上线几天获得 7.9k Star。

## 四个原则

### 1. Think Before Coding — 不猜，不装懂

- 不确定就问，不要默默选一个解释运行
- 有歧义就把几种解释都列出来，让人拍板
- 发现更简单的方案就直接说出来
- 哪里不懂就直接指出来，停下来澄清

**解决的问题**：猜错需求被迫返工（每次约半小时）

### 2. Simplicity First — 能少写就不多写

- 只做需求内的功能，不提前加"灵活性"
- 单次调用不做抽象，不搞过度工程
- 不可能发生的场景，不做错误处理
- 200行能写完，绝不写201行

**解决的问题**：读懂和维护过度工程代码（每次约半小时）

### 3. Surgical Changes — 只改该改的

- 不动相邻的代码、注释、格式
- 没坏的地方不重构
- 原有风格是什么就是什么，哪怕不喜欢
- 发现无关死代码，只提醒不删除

**解决的问题**：还原被误修改的"无辜代码"（每次十几分钟）

### 4. Goal-Driven Execution — 先想清楚"做好了"是什么样

- 把模糊需求转成可验证的目标
- 修 bug 先写测试复现，再改
- 多步任务要写出每一步验收标准
- 目标越具体，AI 越少瞎折腾

**解决的问题**：反复沟通"这样对不对"（每次约十分钟）

## CLAUDE.md 完整内容

```markdown
# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes.

## 1. Think Before Coding
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them.
- If a simpler approach exists, push back.
- If unclear, stop and ask clarification.

## 2. Simplicity First
- No features beyond what was asked.
- No abstractions for single-use code.
- No unrequested "flexibility" or "configurability".
- If 200 lines can be 50, rewrite it.

## 3. Surgical Changes
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing project style.
- Mention dead code but don't delete it unless asked.

## 4. Goal-Driven Execution
- Transform tasks into verifiable success criteria.
- State a brief plan for multi-step tasks with checks.
```

## 三个反常识结论

### 1. 行为约束 > 更大的模型

GitHub issue 区反馈几乎一模一样："加了这个文件后，Claude 不再自作主张了"。  
实测数据：相同任务的 code review **一次通过率从 ~40% 提升到 80%+**。

大部分时候不是 AI 能力不够，而是行为不对——太"热情"，喜欢过度发挥，喜欢帮你做决定。  
给它一套清晰边界规则，**效果比换更贵的大模型好，成本为零**。

### 2. 提示"落地" vs 提示"工程"

| 维度 | 聊天框提示词 | CLAUDE.md 项目规则 |
|------|------------|-----------------|
| 生命周期 | 关窗即消失 | 项目存在就生效 |
| 版本管理 | 无 | 可 git 管理 |
| 团队共享 | 无 | 新人自动继承 |
| 可迭代性 | 每次重写 | 持续演化 |

把沉淀下来的经验变成**项目级的固定规则**，这才是可复用、可传承的提示工程。

### 3. 人机分工明确

- **人负责**：定需求、拍方案、划清边界（做不做、做成什么样）
- **AI 负责**：怎么做，不多做也不少做，不越界

最好的工程师不是写代码最多的，而是**最克制**的。以前这道理用来管人，现在用来管 AI 一样好使。

## 权衡取舍

这些准则偏向**谨慎优先于速度**。对于简单任务（修错别字、明显的一行改动），可灵活判断——不是每次修改都需要完整流程。目标是减少非简单任务中的高代价失误，而非拖慢简单操作。（来源：CLAUDE.md 原文 Tradeoff Note）

## 使用方法

**Plugin 安装（推荐）**：
```
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills
```

**Per-project 安装**：
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

**适用范围**：不绑定 Claude Code，Cursor / GPT-4o / Gemini Code Assist 等均可使用。

## 相关概念

- [[Skill定义规范]] — Skill 定义、SKILL.md 标准写法与安全规范
- [[Spec-Coding]] — 同属"动手前先建立清晰共识"方法论，Spec 是更完整的项目级规范版本
- [[Skills-Github化]] — 将能力固化为可复用文件，CLAUDE.md 是最轻量的固化形式
- [[Chain-of-Thought]] — Think Before Coding 原则与 CoT 异曲同工：让推理过程显式化
- [[约束先行]] — 建立规则的方法论，行为准则是其框架下的具体内容
- [[Harness-Engineering]] — 行为准则的系统化升级：从个人约束到 Harness 级约束体系
- [[Claude-Code-系统化用法]] — CLAUDE.md 是用法 02/06 的具体落地形式，本文是系统化使用全景

## 来源

- [7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包，普通人也能写出大神级代码](../raw/7.9k%20Star%20一夜爆火！把%20Andrej%20Karpathy%20做成了%20Claude%20技能包，普通人也能写出大神级代码/article.md) — 结构派 AI，2026-04-09
- [Karpathy-Inspired Claude Code Guidelines (CLAUDE.md 原文)](../raw/Karpathy-Inspired%20Claude%20Code%20Guidelines/article.md) — forrestchang/andrej-karpathy-skills 仓库原文
