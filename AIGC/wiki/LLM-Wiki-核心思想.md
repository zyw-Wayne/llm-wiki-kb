# LLM Wiki 核心思想

**分类**: 模式  
**标签**: 知识库、LLM、个人知识管理、RAG

---

## 核心理念 (The Core Idea)

**EN**: Most people's experience with LLMs and documents looks like RAG: you upload files, the LLM retrieves chunks at query time, generates an answer. The LLM is **rediscovering knowledge from scratch on every question**. There's no accumulation.

**CN**: 大多数人与 LLM 和文档的体验类似 RAG：上传文件，查询时检索片段，生成答案。LLM **每次从零重新发现知识**。没有积累。

---

**EN**: The wiki is a **persistent, compounding artifact**. The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read.

**CN**: Wiki 是**持久的、累积的知识制品**。交叉引用已建立、矛盾已标注、综合已反映你读过的所有内容。

---

## 分工 (Division of Labor)

| 角色 | 职责 |
|------|------|
| **人类 (You)** | 筛选来源、探索方向、提问、思考意义 |
| **LLM** | 摘要、交叉引用、归档、记账、维护一致性 |

**EN**: You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it.

**CN**: 你从不（或很少）亲自写 wiki——LLM 负责所有书写和维护工作。

---

## 为什么有效 (Why This Works)

> **EN**: The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping.
>
> **CN**: 维护知识库最枯燥的不是阅读或思考，而是记账。

**EN**: Humans abandon wikis because the maintenance burden grows faster than the value. **LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass.**

**CN**: 人类放弃 wiki 是因为维护负担增长快于价值。**LLM 不会无聊、不会忘记更新交叉引用、可以一次修改 15 个文件。**

---

## 思想渊源 (Related Ideas)

**EN**: Related in spirit to **Vannevar Bush's Memex (1945)** — a personal, curated knowledge store with associative trails between documents.

**CN**: 精神上是 **Vannevar Bush 的 Memex (1945)** 的延续——个人化的、经过策展的知识库，文档之间有联想式轨迹。

**EN**: Bush's vision was private, actively curated, with connections between documents as valuable as the documents themselves. The part he couldn't solve was **who does the maintenance**. The LLM handles that.

**CN**: Bush 的愿景是私人的、主动策展的，文档间的连接与文档本身一样有价值。他没解决的问题是：**谁来做维护**。LLM 解决了这个问题。

---

## 关键对比 (Key Contrasts)

| 特性 | RAG | LLM Wiki |
|------|-----|----------|
| 知识积累 | ❌ 每次重新检索 | ✅ 回答存回 wiki |
| 维护者 | 人类 | LLM |
| 检索效率 | 向量相似度 | 索引 + 交叉引用 |
| 技术栈 | 重（向量 DB） | 轻（Markdown 文件） |

---

## 相关概念

- [[LLM-Wiki]] — LLM Wiki 完整介绍（架构、工作流、适用场景）
- [[RAG-vs-LLM-Wiki]] — RAG 与 LLM Wiki 对比选型

---

**来源**:
- [如果你还没学 RAG 可以不学了！卡帕西的 LLM Wiki 方案就很香](../raw/如果你还没学 RAG 可以不学了！卡帕西的 LLM Wiki 方案就很香/article.md) (中文解读)
- [LLM Wiki 英文原文](../raw/LLM-Wiki-Original/article.md) (英文原文)
