# 知识库操作日志（归档）

> 归档时间：2026-04-15  
> 覆盖范围：2026-04-12 ~ 2026-04-14（共 28 条操作记录）  
> 当前日志请查看 [log.md](./log.md)

---

## [2026-04-14] ingest | Agent Skills 终极指南：入门、精通、预测

- 新建：`Skills-渐进式披露.md`（Skill 三级加载机制、工程挑战）
- 更新：`Skill-vs-Prompt.md`（补充 Skills 连续体：慢=Prompt，快=Workflow，对 AI 产品设计的影响）
- 更新：`Skill定义规范.md`（补充 Skill vs MCP 区分、零代码编写视角）
- 更新：`index.md`（+1 页）
- 来源：知乎专栏 · 一泽Eze

---

## [2026-04-14] ingest | 三论Harness——Hermes Agent 完全指南

- 新建：`Hermes-Agent.md`（Nous Research 开源 Agent 运行时，三层记忆+驾驭闭环，Harness 完整落地范例）
- 更新：`Harness-Engineering.md`（补充 Hermes 案例、来源和交叉引用）
- 更新：`index.md`（+1 页）
- 来源：微信公众号 · 技术架构师视界

---

## [2026-04-14] lint | 发现 11 个问题（6类），修复 5 处

**检查范围**: 27 个 wiki 页面 + index.md + log.md

### 修复情况

- ✅ MCP.md：`[[MCP一篇就够了]]` 改为 markdown 链接（修复断链）
- ✅ Context-Engineering.md：补充 `[[MCP]]` 反向引用
- ✅ Token优化技巧.md：frontmatter 格式规范化（移除非标准 `related`/`created` 字段，新增正文"相关概念"区块）
- ✅ 横纵分析法.md：关联页面链接格式统一为 `[[Page]] — 描述`
- ⚠️ CoT "退休"矛盾：两页面适用前提不同（推理模型 vs 标准模型），非真矛盾，保留观察
- ⚠️ log.md 超 700 行：建议后续归档旧记录
- ℹ️ 数据空白（Agent 架构、Self-Consistency、ReAct 深度页）延续，待有素材补充

### 与上次 lint 对比

| 问题类型 | 上次 | 本次 | 变化 |
|---------|------|------|------|
| 矛盾 | 0 | 2 | +2（CoT 适用范围，非真矛盾） |
| 概念缺页 | 2 | 1 | -1（MCP 已补建） |
| 缺失交叉引用 | 0→0 | 4→0 | ✅ 全部修复 |
| 超大页面 | 0 | 1 | +1（log.md） |
| 格式不一致 | 0 | 2→0 | ✅ 全部修复 |
| 数据空白 | 0 | 3 | 信息性，待素材 |

---

## [2026-04-14] ingest | MCP一篇就够了（知乎）

**来源**: https://zhuanlan.zhihu.com/p/29001189476
**获取方式**: article-wiki 技能（chrome-devtools 抓取）
- 新建：`MCP.md`（MCP 定义、演进路径、架构解构、工具选择原理源码分析、Server 开发实践）
- 更新：`index.md`（新增 MCP 条目，文章数 +1，页面数 +1）
- 填补 lint 标记的 MCP 概念缺页

---

## [2026-04-14] ingest | Harness Engineering 深度解析（知乎）

**来源**: https://zhuanlan.zhihu.com/p/2014014859164026634
**获取方式**: article-wiki 技能（chrome-devtools 抓取）
- 更新：`Harness-Engineering.md`（大幅扩充：四大支柱、量化证据、实战案例、成熟度模型、六大共识/四大分歧/三大空白区）
- 更新：`Context-Engineering.md`（新增三层上下文体系、40% 甜蜜区间数据）
- 更新：`index.md`（更新 2 条摘要描述，文章数 +1）

---

## [2026-04-14] lint | 发现 6 个问题（2类），修复 4 处

**检查范围**: 24 个 wiki 页面 + index.md + log.md

### 修复情况

- ✅ 补充 `代码执行范式.md` 反向引用 `[[Skills-逆向建模]]` + `[[Skills-Github化]]` + `[[Spec-Coding]]`
- ✅ 补充 `LLM-Wiki.md` 反向引用 `[[Skills-Github化]]`
- ⚠️ 概念缺页（MCP、Agent 架构）延续，已改为纯文本引用，待有素材时补建

### 与上次 lint 对比

| 问题类型 | 上次 | 本次 | 变化 |
|---------|------|------|------|
| 概念缺页 | 2 | 2 | 持平 |
| 缺失反向引用 | 0 | 4→0 | ✅ 全部修复 |
| 破损交叉引用 | 0 | 0 | ✅ 保持 |
| 自我引用 | 0 | 0 | ✅ 保持 |
| index.md 重复 | 0 | 0 | ✅ 保持 |
| **总计** | **2** | **6→2** | ✅ 减少 4 个问题 |

---

## [2026-04-14] ingest | 别再说AI笨了！7个提示词技巧让免费模型也能输出惊艳答案

**操作类型**: 文章下载 + 新知识编译

**触及页面**:
- 新建 `raw/别再说AI笨了！7个提示词技巧让免费模型也能输出惊艳答案/article.md`（图片 10 张）
- 新建 `wiki/Prompt工程方法论全景.md` — 7 种 Prompt 技巧系统对比
- 更新 `wiki/Chain-of-Thought.md` — 新增 CoT vs ToT 对比表、补充来源
- 更新 `wiki/index.md` — 新增 1 个页面条目，计数更新至 26 页

---

## [2026-04-14] ingest | 99%的人不会用，高手都在用的12个隐藏指令

**操作类型**: 文章下载 + 新知识编译

**触及页面**:
- 新建 `raw/99%的人不会用，高手都在用的12个隐藏指令/article.md`（图片 2 张）
- 新建 `wiki/Claude-Code-系统化用法.md` — 12 种 Claude Code 进阶用法
- 更新 `wiki/AI编程行为准则.md`、`wiki/Harness-Engineering.md`、`wiki/Token优化技巧.md` — 交叉引用
- 更新 `wiki/index.md` — 计数更新至 25 页

---

## [2026-04-14] ingest | 用这招和 Claude 对话：省 75% Token，速度翻倍

**操作类型**: 文章下载 + 新知识编译

**触及页面**:
- 新建 `raw/用这招和 Claude 对话：省 75% Token，速度翻倍/article.md`（图片 8 张）
- 新建 `wiki/Token优化技巧.md` — 穴居人/文言文模式压缩 output token
- 更新 `wiki/index.md` — 计数更新至 24 页

---

## [2026-04-14] lint | 发现 5 个问题（4类），修复 3 处，补充 4 处反向引用

**检查范围**: 23 个 wiki 页面 + index.md + log.md

- ✅ 修复 2 处自我引用、1 处破损引用、补充 4 处反向引用
- ⚠️ 概念缺页（MCP、Agent 架构）延续

---

## [2026-04-14] refresh | 更新 WIKI.md 元信息

- 重新生成统计概览：17 篇文章 → 23 个知识页面，3 个分类
- 同步更新父目录 collection WIKI.md 描述行

---

## [2026-04-14] ingest | 踏马的 Agent

**操作类型**: 文章下载 + 新知识编译

**触及页面**:
- 新建 `raw/踏马的 Agent/article.md`（图片 3 张）
- 新建 `wiki/Harness-Engineering.md`、`wiki/Context-Engineering.md`
- 更新 `wiki/index.md` — 计数更新至 23 页

---

## [2026-04-13] ingest | 一文带你看懂，火爆全网的Skills到底是个啥

**操作类型**: 文章下载 + 新知识编译

**触及页面**:
- 新建 `wiki/Skills 应用场景-科普.md`、`wiki/Skills 生态时间线.md`
- 更新 `wiki/Skill-vs-Prompt.md`、`wiki/Skills-设计哲学.md`、`wiki/index.md`

---

## [2026-04-13] query | CoT 横纵分析

- 新建 `wiki/query-cot-横纵分析.md` — CoT 完整横纵分析报告
- 更新 `wiki/index.md` — 计数更新至 19 篇

---

## [2026-04-13] lint | 发现 6 个问题（3类），修复 4 处

- ✅ 修复 Skill定义规范.md 破损引用、代码执行范式.md 残留链接
- ✅ 补充双向交叉引用
- ⚠️ 概念缺页（MCP、Agent 架构）延续

---

## [2026-04-13] lint | 发现 6 个问题（3类）

- ✅ 修复 3 处缺失交叉引用
- ⚠️ 概念缺页延续

---

## [2026-04-13] ingest | 一文搞懂爆火的SKills原理及实践案例

- 新建 `wiki/Skills-设计哲学.md`、`wiki/Skills-逆向建模.md`、`wiki/AI辅助CodeReview.md`
- 更新 `wiki/index.md` — 计数更新至 18 篇

---

## [2026-04-13] ingest | AI入门——一文读懂什么是Skill以及如何开发Skill

- 新建 `wiki/Skill-vs-Prompt.md`
- 更新 `wiki/index.md` — 计数更新至 15 篇

---

## [2026-04-13] lint | 发现 9 个问题（6类）

- 首次全面检查 14 个 wiki 页面
- 发现：6 处 index.md 重复、6 处破损引用、1 处自我引用、2 个概念缺页、2 处缺失引用、3 处命名不一致

---

## [2026-04-13] ingest | 写好一个 Skill 有多难？Anthropic 踩了几百个坑之后的答案

- 新建 `wiki/Skill实战踩坑指南.md`
- 更新 `wiki/index.md` — 计数更新至 14 篇

---

## [2026-04-13] ingest | 兄弟！你真的懂 Skill 吗？

- 新建 `wiki/Skill框架源码分析.md`
- 更新 `wiki/index.md` — 计数 12→13

---

## [2026-04-13] ingest | 如何规范地定义一个 Skill · 完整指南

- 新建 `wiki/Skill定义规范.md`
- 更新 `wiki/index.md` — 计数 11→12

---

## [2026-04-13] ingest | 7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包

- 新建 `wiki/AI编程行为准则.md`
- 更新 `wiki/index.md` — 计数 10→11

---

## [2026-04-13] ingest | 我在 Prompt 里加了四个字，Claude Code 的输出质量提升了一倍

- 新建 `wiki/Chain-of-Thought.md`
- 更新 `wiki/index.md` — 计数 9→10

---

## [2026-04-13] ingest | 分享一个我用了2年的深度研究Prompt（补录 raw + 首次）

- 新建 `wiki/横纵分析法.md`
- 更新 `wiki/index.md`

---

## [2026-04-13] ingest | 一个人管不了所有事——如何让多个 Skill 协同工作

- 新建 `wiki/Skills-多Skill协作.md`
- 更新 `wiki/index.md`

---

## [2026-04-13] lint | 发现 18 个问题（6类）

- 首次 lint：7 个 wiki 页面 + index.md
- 18 个问题：6 重复 + 6 破损 + 1 自引 + 2 缺页 + 2 缺引 + 3 命名

---

## [2026-04-13] ingest | 中英混写Prompt更有创造力？我设计了个盲评实验来验证

- 新建 `wiki/Prompt语言选择.md`
- 更新 `wiki/index.md`（新增「Prompt 工程」分类）

---

## [2026-04-12] ingest | 一文快速理解 Spec 模式

- 新建 `wiki/Spec-Coding.md`
- 更新 `wiki/index.md`

---

## [2026-04-12] ingest | Skills的最正确用法，是将整个Github压缩成你自己的超级技能库。

- 新建 `wiki/Skills-Github化.md`（补录+重新编译）
- 更新 `wiki/index.md`

---

## [2026-04-12] ingest | Anthropic 又一篇 Agent 开发神文，新范式让 Token 消耗暴降 98.7%

- 新建 `wiki/代码执行范式.md`
- 更新 `wiki/index.md`

---

## [2026-04-12] ingest | LLM Wiki (Original English)

- 新建 `raw/LLM-Wiki-Original/article.md`
- 更新 `wiki/LLM-Wiki.md`，新建 `wiki/LLM-Wiki-核心思想.md`
- 更新 `wiki/index.md`

---

## [2026-04-12] ingest | 如果你还没学 RAG，可以不学了！卡帕西的 LLM Wiki 方案就很香

- **初次建仓**
- 新建 `WIKI.md`、`wiki/index.md`、`wiki/LLM-Wiki.md`、`wiki/RAG-vs-LLM-Wiki.md`
