# AIGC 知识库 Schema

**类型**: wiki  
**主题**: AIGC（人工智能生成内容）技术、工具与实践  
**创建时间**: 2026-04-12  
**维护者**: LLM Wiki

---

## 统计概览

| 指标 | 数值 |
|------|------|
| 原始文章数 | 42 篇 |
| 知识页面数 | 71 个 |
| 分类数 | 14 个 |
| 最近更新 | 2026-04-17 |
| 累计操作 | ingest 42 · query 1 · lint 11 · refresh 3 |

## 知识分类

| 分类 | 页面数 | 包含页面 |
|------|--------|---------|
| 知识管理 | 3 | LLM-Wiki · LLM-Wiki-核心思想 · RAG-vs-LLM-Wiki |
| Claude Code 实战 | 1 | Claude-Code-会话管理实战 |
| Agent 架构与设计 | 10 | AI编程范式三次浪潮 · Harness-Engineering · Agent-Harness-设计哲学五大原则 · Agent-Harness-构建指南 · Claude-Code-架构全景 · Claude-Code-源码架构 · Hermes-Agent · Agent-类型分类 · Agent构建四阶段 · 多智能体协作五种模式 |
| Agent 核心系统 | 8 | 对话循环-AsyncGenerator · 工具系统五要素协议 · Agent-权限管线四阶段 · Agent-配置系统 · Agent-记忆系统 · Agent-钩子系统 · 流式架构与性能优化 · Plan模式与结构化工作流 |
| Agent 高级模式 | 3 | SubAgent-与-Fork模式 · Coordinator模式 · Tool-vs-Skill-vs-SubAgent选型 |
| Context 工程 | 5 | Context-Engineering · Context-Rot · Context-压缩 · 上下文管理策略 · 上下文四级压缩策略 |
| Skills 体系 | 13 | Skills-系统架构与插件 · Skill定义规范 · Skills-渐进式披露 · Skill实战踩坑指南 · Skill框架源码分析 · Skills-Github化 · Skills-多Skill协作 · Skills-设计哲学 · Skills-逆向建模 · Skill-vs-Prompt · Skills 应用场景-科普 · Skills 生态时间线 · MCP |
| 多步骤 Skill 工程 | 3 | 多步骤Skill稳定执行 · Output-Contract · Checkpoint机制 |
| 工程实践 | 7 | 约束先行 · 代码执行范式 · Spec-Coding · AI编程行为准则 · AI辅助CodeReview · Claude-Code-系统化用法 · Token优化技巧 |
| 参考速查 | 1 | Claude-Code-功能标志与术语表 |
| Prompt 工程 | 5 | Prompt语言选择 · 横纵分析法 · Chain-of-Thought · query-cot-横纵分析 · Prompt工程方法论全景 |
| 思维方法与创造力 | 1 | AI时代创造力方法论 |
| 技术写作与创作 | 2 | 花叔写作Skill逆向方法论 · 技术写作风格DNA |
| 知识梳理存档 | 1 | 有效的-Context-工程-见知录004 |

## 最近活动

| 日期 | 操作 | 摘要 |
|------|------|------|
| 2026-04-17 | ingest | Claude Code 会话管理实战 → +1 页, ~2 页（Anthropic 官方博客） |
| 2026-04-17 | ingest | Skill 最佳实践：五种设计模式+六步打磨法 → +1 页, ~2 页 |
| 2026-04-17 | ingest | Anthropic 多智能体协作五种模式 → +1 页, ~1 页 |
| 2026-04-16 | ingest | 多步骤 Skill 稳定执行手册 → +3 页（飞书来源） |
| 2026-04-15 | lint | 发现 4 个问题（3 类），修复 4 处 |
| 2026-04-15 | ingest | 花叔不公开的写作 Skill → +2 页, ~1 页 |
| 2026-04-15 | ingest | Agent Skills 完全指南 → ~3 页 |
| 2026-04-15 | ingest | 万字干货！Agent Skills从入门到精通 → ~4 页 |

## 健康状态

最近一次 lint（2026-04-17）：**0 个遗留问题**。全部 71 页在 index 中，无孤儿页面、无破损链接、无超大页面。

---

## 知识库结构

```
AIGC/
├── raw/               # 原始文章（LLM 只读不写）
│   └── <文章标题>/
│       └── article.md
├── wiki/              # 编译生成的知识页面
│   ├── index.md       # 内容目录
│   ├── log.md         # 操作日志
│   └── *.md           # 概念/主题/分析页面
├── evolve/            # 知识进化追踪（LLM 维护）
│   ├── index.md       # 总控面板：所有分类评分 + 下一步建议
│   └── <分类名>.md    # 每个主题的进化档案（覆盖度评估 + 进化向量 + 日志）
└── WIKI.md            # 本文件
```

## 工作流程

### Ingest（摄入）
1. 新文章放入 `raw/`
2. LLM 读取文章，提取核心概念、观点、实体
3. 创建或更新 `wiki/*.md` 页面
4. 更新 `index.md`
5. 追加 `log.md`

### Query（查询）
1. 读取 `wiki/index.md` 定位相关页面
2. 读取相关 wiki 页面，综合给出答案
3. 有价值的回答存为新页面 `query-<主题>.md`
4. 追加 `log.md`

### Lint（健康检查）
1. 检查矛盾、过时内容、孤儿页面
2. 检查缺失的交叉引用
3. 输出健康报告
4. 追加 `log.md`

### Evolve（进化追踪）

对指定分类或全部分类进行覆盖度审计，识别进化向量（知识可生长的方向），追踪知识库的成熟度变化。

- `wiki evolve <分类名>`：审计指定分类
- `wiki evolve`：更新所有已评估分类
- `wiki evolve --all`：全量评估（含未评估分类）
- Query 后被动触发（保守策略）：仅在发现明确知识盲区时自动更新

结果写入 `evolve/` 目录，同时追加 `wiki/log.md`。

### Refresh（刷新元信息）
扫描实际状态，重新生成本文件的统计概览、分类、最近活动、健康状态。

### Log（聚合日志）
读取 `wiki/log.md`，输出精简聚合视图（不修改原文）。

### List（精简概览）
读取 `wiki/index.md` + 扫描目录，输出一屏可读的知识库全貌。

## 页面格式约定

- 使用 `[[页面名称]]` 进行交叉引用
- 矛盾用 `> ⚠️ 矛盾：` 标注
- 过时内容标记为 `[STALE]`
- 来源引用使用 markdown 链接 `[文章标题](../raw/文章标题/article.md)`

## 知识分类体系

- **知识管理**: LLM Wiki 方法论、RAG 对比、核心思想
- **Agent 架构与设计**: 编程范式演进、Harness 工程、设计哲学、架构全景、源码分析、Agent 类型分类
- **Agent 核心系统**: 对话循环、工具系统、权限管线、配置/记忆/钩子系统、流式架构、Plan 模式
- **Agent 高级模式**: SubAgent/Fork、Coordinator、Tool/Skill/SubAgent 选型
- **Context 工程**: 上下文管理策略、上下文腐烂、压缩策略、四级压缩
- **Skills 体系**: 架构与插件、定义规范、渐进式披露、实战踩坑、源码分析、设计哲学、MCP 协议
- **工程实践**: 约束先行、代码执行范式、Spec-Coding、行为准则、Code Review、Token 优化
- **参考速查**: 功能标志与术语表
- **Prompt 工程**: 语言选择、分析方法、思维链、方法论全景
- **思维方法与创造力**: AI 时代创造力恢复框架
- **技术写作与创作**: 写作 Skill 逆向方法论、风格 DNA
- **知识梳理存档**: 万字精读、综合分析报告

## 知识积累原则
好的综合性回答应存回 wiki/，成为新的知识节点，供后续查询直接复用。知识库越用越聪明。
