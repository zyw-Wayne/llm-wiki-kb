---
title: Skill 实战踩坑指南
tags: [Skill, Claude Code, 实战, 踩坑, Anthropic, 最佳实践]
sources: [raw/写好一个 Skill 有多难？Anthropic 踩了几百个坑之后的答案/article.md, raw/Lessons from Building Claude Code - How We Use Skills/article.md]
updated: 2026-04-15
---

# Skill 实战踩坑指南

## 定义

Anthropic 内部数百个 Skill 的实战经验系统总结，从"写出来容易，写好很难"的角度，梳理 Skill 开发中最容易踩的坑和最有效的改进策略。

## 核心观点

- Skill 本质是**文件夹**，不是单个 Markdown 文件——可以放脚本、数据、模板、配置，支持动态注册 Hook
- Anthropic 内部用得最好的 Skill，恰恰是充分利用了文件夹结构和配置选项的
- Skill 的价值不在初始版本写得多完整，而在于**使用过程中不断补充 Claude 犯过的错**
- 真正该写进 Skill 的是那些 Claude 不知道的、你的团队特有的知识（内部约定、踩过的坑、用什么工具查什么问题）

## 9 类 Skill 分类体系

Thariq 将 Anthropic 内部所有 Skill 归为 9 类，最好的 Skill 干净利落地属于某一类：

| 类型 | 说明 | 典型场景 |
|------|------|---------|
| 库和 API 参考 | 教 Claude 正确使用某个库/CLI/SDK | 内部计费库边界情况、CLI 子命令用法 |
| 产品验证 | 描述怎么测试和验证代码 | 配合 Playwright/tmux 做验证，录制测试视频 |
| 数据获取与分析 | 连接数据和监控系统 | 事件表查询、Grafana dashboard 对应关系 |
| 业务流程自动化 | 一键化重复性工作流 | 自动聚合 ticket/GitHub/Slack 生成 standup |
| 代码脚手架 | 为特定功能生成框架代码 | 新建带标准认证、日志和部署配置的内部应用 |
| 代码质量与审查 | 强制执行代码质量标准 | 启动子 Agent 挑刺，迭代到只剩 nitpick |
| CI/CD 与部署 | 拉代码、推代码和部署 | 监控 PR、自动重试 flaky CI、灰度发布回滚 |
| Runbook | 症状→多工具调查→结构化报告 | 把 on-call 经验沉淀为可执行标准流程 |
| 基础设施运维 | 日常维护和运维操作 | 查找孤立 Pod/Volume，先确认再清理 |

> 体感：验证类和业务流程自动化类最容易见效，因为它们解决的是每天都在重复的痛点。

## 九大实战原则

### 1. 别写 Claude 已经知道的（续：参见 [[Skills-设计哲学]] 的"恰好而非更多"理念）

Claude 本身就懂很多编程知识。Skill 应该重点放在那些能把 Claude 推出它默认思维模式的信息上。

**反例**："请写出高质量代码"——Claude 本来就会。
**正例**：frontend-design Skill 教 Claude 避免经典 AI 审美（Inter 字体配紫色渐变）。

### 2. 好好写 Gotchas 部分

任何 Skill 里信息量最高的部分就是 Gotchas（踩坑清单）。把 Claude 使用 Skill 时经常犯的错记下来，随着使用不断补充。

> 一开始写的 Skill 可能只有二三十行，用了一个月之后 Gotchas 部分比正文还长。因为每次 Claude 犯一个新错，你就加一条，这些积累才是 Skill 真正的价值所在。

### 3. 用好文件系统和渐进式披露

把整个文件系统当作上下文工程和渐进式披露的手段：
- SKILL.md 里告诉 Claude 这个 Skill 里有哪些文件
- 详细的函数签名和用法示例拆到 `references/api.md`
- 脚本、示例、参考文档分门别类放进 Skill 文件夹
- Claude 知道它们在那儿，需要的时候自己会去找

### 4. 别把 Claude 钉死

Skill 会被反复使用，指令太具体在某些场景下会变得死板。

**反例**："必须按 A→B→C→D 四个步骤执行"
**正例**："通常的流程是 A→B→C→D，但可以根据实际情况调整顺序或跳过某些步骤"

### 5. 想清楚初始化流程

有些 Skill 需要用户先提供上下文才能用。好的做法：
- 配置信息存到 `config.json` 里
- 第一次运行时问用户，配置好后直接用
- 用 `AskUserQuestion` 工具提结构化选择题

### 6. description 字段是写给模型看的

description 不是给人看的摘要，而是给模型看的触发条件。应该写清楚"**什么时候应该触发**"，而不是"这个 Skill 做了什么"。

**反例**："用于格式化代码"——触发率低
**正例**："当用户要求格式化代码、或代码风格不一致时触发"——触发率高

### 7. 给 Skill 加记忆

通过存储数据实现记忆功能：
- 简单：往文本日志追加内容
- 复杂：用 SQLite 数据库
- 数据应存到稳定位置（`${CLAUDE_PLUGIN_DATA}`），避免升级时被删

### 8. 放脚本，让 Claude 组合

给 Claude 代码是你能做的最有力的事之一。把辅助脚本和库函数放到 Skill 里，Claude 就可以把精力花在组合和决策上，而不是从头写重复的样板代码。

### 9. 按需 Hook

Skill 可以包含只在被调用时才激活的 Hook，持续到会话结束。适合那些"有态度"的 Hook：
- `/careful`：拦截 `rm -rf`、`DROP TABLE`、`force-push` 等危险操作
- `/freeze`：阻止编辑特定目录之外的文件

## 分发与统计

- 小团队：直接提交到仓库的 `.claude/skills` 目录
- 规模大了：搭建内部 Plugin Marketplace
- Anthropic 内部没有集中团队决定上架——有人做了好用的 Skill，先放 sandbox 试用，有口碑再提 PR
- ⚠️ 容易创建质量差或重复的 Skill，上架前需有筛选机制（来源：Thariq 英文原帖）
- 用 `PreToolUse` Hook 记录 Skill 使用情况，发现高频使用和**触发率低于预期（undertriggering）**的 Skill
- Anthropic 近期发布了 **Skill Creator** 工具，降低 Skill 创建门槛（来源：Thariq 英文原帖）

## Skill 之间的组合依赖

目前 Marketplace 没有原生依赖管理，但可以在 Skill 里直接引用其他 Skill 的名字，Claude 会自动调用已安装的。

## 与现有知识的交叉验证

> ⚠️ 矛盾：本文强调 Skill 是文件夹而非单个文件，[[Skill定义规范]] 中将 Skill 定义为"结构化的元指令文件"，两者在"文件 vs 文件夹"的表述上有差异。实际上两者并不矛盾——Skill 的核心确实是 SKILL.md 这个元指令文件，但最佳实践是充分利用文件夹结构来组织补充资源。

与 [[Skill定义规范]] 的互补关系：
- Skill定义规范 聚焦**定义和标准写法**（frontmatter、三段式结构、description 写法三原则）
- 本文聚焦**实战踩坑和改进策略**（9 类分类、九大原则、分发统计）

与 [[Skills-多Skill协作]] 的互补关系：
- Skills-多Skill协作 聚焦**架构设计**（单一职责、三种协作模式、共享状态）
- 本文提供了更多**微观层面的打磨技巧**（Gotchas、渐进式披露、description 写法）

与 [[Skill测试与质量保障]] 的互补关系：
- 本文的"好好写 Gotchas"是经验层面的质量保障
- Skill测试与质量保障 将其系统化为可操作的测试工程（JSON 测试集、三种断言、Description 优化四步法）

## 相关概念

- [[Skill定义规范]] — Skill 定义、SKILL.md 标准写法与安全规范
- [[Skills-多Skill协作]] — 多个 Skill 如何协同工作，单一职责拆分
- [[Skill框架源码分析]] — 框架架构、skill_run 沙箱、五种执行模式的源码级分析
- [[Skills-Github化]] — 将 GitHub 开源项目 Skill 化的实战工作流
- [[AI辅助CodeReview]] — 代码质量与审查类 Skill 的工程化落地
- [[Skills-设计哲学]] — "恰好而非更多"的设计哲学与踩坑经验互补
- [[Skills-分类学]] — 分类学视角的颗粒度控制，与 description 触发设计互补

## 来源

- [写好一个 Skill 有多难？Anthropic 踩了几百个坑之后的答案](../raw/写好一个 Skill 有多难？Anthropic 踩了几百个坑之后的答案/article.md) — 中文转述，Anthropic 工程师 Thariq 的 Skill 实战踩坑经验
- [Lessons from Building Claude Code: How We Use Skills](../raw/Lessons from Building Claude Code - How We Use Skills/article.md) — Thariq (@trq212) 英文原帖，2026-03-17，含完整示例和细节
