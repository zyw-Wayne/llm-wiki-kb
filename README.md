# llm-wiki-kb

基于 [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 方法论构建的个人知识库 — 由 LLM 全自动维护的持久化、可复合的 Markdown Wiki。

> 本知识库使用 [wei-llm-wiki](https://github.com/zyw-Wayne/wei-llm-wiki) 技能驱动 ingest / query / lint / evolve 等全部工作流。

## 什么是 LLM Wiki？

传统 RAG 每次查询都从原始文档重新检索和推理；LLM Wiki 则让 LLM **增量构建并维护一个结构化的知识库**，知识在每次摄入和查询中持续编译、交叉引用、自我修正，越用越聪明。

| 层 | 角色 | 谁维护 |
|----|------|--------|
| **Raw Sources** | 不可变的原始文章 | 人类投喂 |
| **Wiki** | 编译后的知识页面 | LLM 全权维护 |
| **Schema** | 工作流约定与配置 | 人机共同演进 |

## 仓库结构

```
.
├── AIGC/                    # AIGC 主题知识库
│   ├── raw/                 # 原始文章（LLM 只读）
│   ├── wiki/                # LLM 编译的知识页面（78 篇）
│   │   ├── index.md         # 内容目录
│   │   └── log.md           # 操作日志
│   ├── evolve/              # 知识进化追踪
│   └── WIKI.md              # Schema（分类、统计、工作流）
└── WIKI.md                  # 顶层导航（多知识库索引）
```

## 当前知识库

### AIGC — 人工智能生成内容

42 篇原始文章编译为 71+ 个知识页面，覆盖 14 个分类：

| 分类 | 页面数 | 核心主题 |
|------|--------|----------|
| Agent 架构与设计 | 10 | Harness 工程、架构全景、多智能体协作 |
| Skills 体系 | 13 | 系统架构、定义规范、渐进式披露、MCP |
| Agent 核心系统 | 8 | 工具系统、权限管线、记忆系统、流式架构 |
| 工程实践 | 7 | 约束先行、Spec-Coding、Code Review |
| Context 工程 | 5 | 上下文管理、Context Rot、四级压缩 |
| Prompt 工程 | 5 | 语言选择、思维链、方法论全景 |
| 知识管理 | 3 | LLM Wiki 核心思想、RAG 对比 |
| 多步骤 Skill 工程 | 3 | 稳定执行、Output Contract、Checkpoint |
| Agent 高级模式 | 3 | SubAgent、Coordinator、选型指南 |
| 技术写作与创作 | 2 | 写作 Skill 逆向方法论、风格 DNA |
| 其他 | 4 | 创造力方法论、术语表、综合分析等 |

## 核心工作流

| 操作 | 说明 |
|------|------|
| **Ingest** | 投喂原始文章 → LLM 自动提取概念、创建/更新 wiki 页面、维护索引和日志 |
| **Query** | 基于已编译知识回答问题，有价值的回答存为新页面 |
| **Lint** | 健康检查：检测矛盾、孤儿页面、破损链接、过大页面 |
| **Evolve** | 覆盖度审计，识别知识可生长的方向 |

## 使用方式

本知识库通过 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) + [LLM Wiki Skill](https://github.com/zyw-Wayne/wei-llm-wiki) 驱动：

```bash
# 摄入一篇文章
wiki ingest <url-or-file>

# 查询知识
wiki query "Agent 权限管线设计"

# 健康检查
wiki lint

# 知识进化追踪
wiki evolve
```

## 设计原则

- **LLM 全权维护** — 人类只投喂源文档和提问，所有 wiki 页面由 LLM 生成和维护
- **知识复合增长** — 每次 ingest 和 query 都让知识库更完整
- **交叉引用** — 使用 `[[页面名称]]` 语法构建知识网络
- **零向量嵌入** — 纯关键词 + 标签匹配，可读、可审计、可 git 管理

## License

MIT
