---
title: Skill 定义规范
tags: [Skill, Claude Code, SKILL.md, 规范, AgentSkills]
sources: [raw/如何规范地定义一个 Skill · 完整指南/article.md, raw/兄弟！你真的懂 Skill 吗？/article.md, raw/Agent Skills 终极指南：入门、精通、预测/article.md, raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Skill 定义规范

## 定义

Skill 是一个以文件夹为单位的、可复用的 AI 能力包。完整公式：**Skill = 可发现的元数据 + 可执行/可操作的过程性指令 + 可选资源（references/assets/scripts）+ 渐进式加载策略 +（可选）权限/环境门控**。

它更像是给 AI 的一份操作说明书——写清楚什么时候用、按什么步骤走、输出什么格式，AI 读到就知道该怎么处理。

## 核心观点

- Skill 不是插件，没有后台程序在跑，本质是结构化的元指令文件（来源：[如何规范地定义一个 Skill · 完整指南](../raw/如何规范地定义一个 Skill · 完整指南/article.md)）
- **description 是成败关键**——AI 靠它判断要不要调用，必须写成"可检索的触发器"（来源：同上）
- 渐进式信息披露：启动时仅加载元数据（name/description），触发后加载正文，执行时再按需读取资源（来源：同上）
- 六个核心原则：单一职责、描述精准、结构清晰、渐进加载、安全透明、版本管理（来源：同上）

## 标准文件结构

```
my-skill/
├── SKILL.md          # 核心文件，必须有
├── references/       # 可选：补充知识/规范文档
├── assets/           # 可选：模板/静态资源
└── scripts/          # 可选：可执行脚本
```

**关键规则**：
- `SKILL.md` 必须直接放在 `skills/your-skill-name/` 目录下，不能再嵌套
- 目录名全小写，连字符分隔

## SKILL.md 标准写法

### Frontmatter（元数据）

| 字段 | 是否必填 | 说明 |
|------|---------|------|
| name | ✅ 必填 | 与目录名保持一致 |
| description | ✅ 必填 | AI 用来判断是否调用此 Skill 的核心依据 |
| version | 推荐 | 便于审计与回滚 |
| author | 推荐 | 便于溯源 |
| requires | 按需 | 声明依赖的其他 Skill 或工具 |
| platforms | 按需 | 限定运行平台 |

### 正文三段式结构

1. **触发条件**：明确说明什么时候用、什么时候不用
2. **执行步骤**：分步列出具体操作
3. **输出格式**：定义输出结构、格式、语言风格

### Description 写法三原则

1. 包含用户可能说的原话，不是你认为准确的术语
2. 明确写出不适用场景，防止 AI 误调用
3. 控制在5行以内，过长反而降低检索精度

> **❌ 错误**：`帮助你更高效地完成任务`
> **✅ 正确**：`当用户询问某个话题的最新趋势、热点动态、行业动向时使用。触发词：趋势分析、热点、最新动态。不适用于：历史事件、代码问题。`

## 常见错误

- **description 太模糊**：AI 不知道该不该用，通常选择不用
- **文件夹结构嵌套错误**：`skills/my-skill/src/SKILL.md` ❌ → `skills/my-skill/SKILL.md` ✅
- **改完没重启**：改完 SKILL.md 需要重启 gateway 才能生效
- **元数据与行为不一致**：声明"只读"却执行写操作
- **硬编码敏感信息**：API Key、服务器 IP 必须用环境变量

## 安全自检清单

- ✅ description 与实际执行行为一致
- ✅ 无硬编码 API Key、密码、路径
- ✅ scripts/ 中的脚本逻辑透明可读
- ✅ 权限声明最小化
- ✅ 不包含任何外部数据外传逻辑
- ✅ 敏感操作有明确的用户确认步骤

## 框架源码视角的补充

Anthropic 16 个官方 Skill 的实践验证了本文规范的核心观点（来源：[[Skill框架源码分析]]）：

- **SKILL.md body 质量是一切的基础**——它是"给 LLM 看的使用手册"，决定 LLM 能否理解任务、选对方法
- **所有官方 Skill 都没用 function calling**——通过 SKILL.md 文本 + skill_run 沙箱驱动，description 触发加载，body 指导执行
- **五种模式**：纯 Prompt 注入 → 参考文档渐进加载 → 库调用 → 脚本执行 → 编排，由轻到重
- description 在框架中的作用：LLM 通过 `skill_list()` 看到所有技能的 name + description（~30 Token/个），以此判断是否加载

## Skill vs MCP

两者常被混淆，但定位完全不同（来源：Agent Skills 终极指南）：

- **MCP**：开放标准的**协议**，关注 AI 如何以统一方式调用外部工具、数据和服务，本身不定义任务逻辑或执行流程
- **Skill**：教 Agent 如何完整处理特定工作，将执行方法、工具调用方式以及相关知识材料封装为完整的「能力扩展包」

通俗比喻：MCP 是门禁卡（进不了数据库、打不开文件柜就是白搭），Skills 是大脑里的知识，Prompt 是嘴巴。

## 源码级补充：完整 SKILL.md Frontmatter 字段（来自 Claude Code 源码）

以下字段来自 Claude Code 第11章源码分析，补充了官方文档未覆盖的配置项（来源：raw/御舆：解码 Agent Harness/article.md）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 省略时使用目录名，建议 kebab-case |
| `description` | string | **必填**，AI 用于判断是否调用 |
| `when_to_use` | string | 帮助模型自动选择合适技能 |
| `arguments` | string | 命名参数列表（空格分隔） |
| `argument-hint` | string | 参数提示格式 |
| `allowed-tools` | list | 遵循最小权限原则 |
| `model` | string | 只读任务用 haiku，复杂任务用 inherit |
| `effort` | string | 推理努力程度 |
| `user-invocable` | bool | 是否用户可直接调用 |
| `disable-model-invocation` | bool | 禁用模型调用（纯脚本模式） |
| `context` | string | `fork`（耗时任务）或 `inline`（快速操作） |
| `agent` | string | 关联的智能体类型 |
| `version` | string | 版本号 |
| `paths` | list | **条件激活**：gitignore 风格路径模式，匹配时才激活技能 |
| `hooks` | object | pre/post 生命周期钩子命令 |

**条件技能（paths 字段）**：一旦激活，在整个会话期间永久保持可用状态（防止缓存清理导致行为不一致）。

**安全边界**：来自 MCP 服务器的技能（`loadedFrom === 'mcp'`）不执行 Shell 命令内联，防止远端注入攻击。

## Description "黄金结构"公式

高质量 description 遵循以下结构（来源：[万字干货！Agent Skills从入门到精通](../raw/万字干货！Agent Skills从入门到精通/article.md)）：

> **[一句话核心功能] + [具体执行动作] + [明确的触发关键词/场景]**

核心原则"触发即正义"——description 首要任务是给 AI 的路由机制看，明确回答两个问题：
1. 这个 skill 是做什么的？（功能定义）
2. 用户在什么场景/说什么话时应该使用它？（触发条件）

写好 description 的秘诀：**模拟用户的提问方式**，把用户可能说的原话关键词都塞进去。

优秀示例：
```
name: security-code-review
description: Reviews code for security vulnerabilities and best practices.
Use when the user asks to "review code", "check for bugs", "analyze security",
or mentions specific issues like SQL injection, XSS, or performance bottlenecks.
```

## Skills 三个"魔法机关"

1. **智能开关**（YAML 元数据）：始终加载的控制面板，决定何时触发
2. **随用随取的小抄**（渐进式披露）：平时不占上下文，需要时才加载 → 详见 [[Skills-渐进式披露]]
3. **呼叫外援与影分身**（行动导向与子代理）：Skills 不只让 AI 读说明书，还能召唤 SubAgent 处理复杂任务

## Skills 创建四阶段流程

| 阶段 | 关键动作 |
|------|---------|
| 1. 明确需求与边界 | 单一职责、触发关键词、所需资源清单 |
| 2. 构建 skill 文件夹 | 选择路径（个人 ~/.claude/skills/ | 项目 .claude/skills/ | 插件）、创建 SKILL.md |
| 3. 编写核心指令 | 职责边界 + 编号步骤 + 输入输出规范 + 硬性约束（≥3 条 + 1 个示例 → 稳定性提升 60%） |
| 4. 测试调试迭代 | 路径检查 → YAML 校验 → 触发测试 → 执行验证；未被触发 90% 是 description 不够具体 |

> `claude --debug` 可查看详细加载日志，用于调试 Skill 触发问题。

## 零代码编写

Agent Skills 开放标准的一个关键优势：**入门门槛极低，智能上限极高。** 非技术出身的领域专家，离自己做专业 Agent 只剩隔着一层窗户纸——把专业经验和工作流程用文档形式写清楚，Agent 就能照着执行。

最简单的 Skill 可以只有一个 SKILL.md，纯自然语言写成；复杂的可以包含向量数据库构建指南、脚本、Persona 模板等。

## 相关概念

- [[Skill框架源码分析]] — 框架架构、skill_run 沙箱、五种执行模式的源码级分析
- [[Skills-多Skill协作]] — 多个 Skill 如何协同工作，单一职责拆分
- [[Skill实战踩坑指南]] — Anthropic 数百个 Skill 实战踩坑经验，9 类分类与九大改进原则
- [[Skills-Github化]] — 将 GitHub 开源项目 Skill 化的实战工作流
- [[Skill-vs-Prompt]] — Skill 与 Prompt 的 12 项详细对比与选型指南
- [[Skills-渐进式披露]] — Skill 三级加载机制（元数据/指令/资源）
- [[MCP]] — MCP 协议与 Skill 的关系与区分
- [[AI编程行为准则]] — Karpathy 四原则，行为约束比更大模型更有效
- [[Skills-设计哲学]] — Skills 从"中台思维"到"恰好而非更多"的推导逻辑与设计哲学
- [[Skills-逆向建模]] — 需求迭代场景下的建模实战方法（实体/流程/规则三维度）
- [[Tool-vs-Skill-vs-SubAgent选型]] — Tool/Skill/Sub-Agent 三种能力单元的选型决策树

## 相关概念（补充）

- [[Skills-系统架构与插件]] — 完整的 Skills 加载引擎、动态发现、条件技能、插件系统源码级分析
- [[Skill-Creator-官方工作流]] — Anthropic 官方完整创建迭代流程，含 Description 写法的官方立场（"稍微激进"）
- [[Skill-Eval-系统架构]] — 官方评估体系，断言打分与盲评系统

## 实战案例：花叔写作 Skill 的架构设计

花叔的写作 Skill 提供了一个高质量 Skill 设计的典型案例（来源：[[花叔写作Skill逆向方法论]]）：

- **Skill 拆分原则**：创作 Skill 只管内容生成，格式转换单独做成 md2book Skill，各司其职
- **复杂度控制**：创作 Skill 已有 8 个参考文件、3 个脚本、2 个模板、2 套测试，不宜再塞入导出逻辑
- **TDD 式质量铁律**：12 项章节 QC + 10 项全书 QC，每项都是 checkbox，必须逐项过
- **Rationalization Table**：专门对付 Agent 找借口跳过规则的机制
- **Agent Protocol**：写作前必须先研究确认事实，不凭训练数据编造

## 打磨方法论速览

一套评测驱动的六步打磨法（来源：[[Skill-设计模式与打磨法]]）：① 建立基线发现问题 → ② 定义验收标准 → ③ 写最小版本 → ④ 逐步扩展（每条新规则对应一个测试用例）→ ⑤ 加记忆和验收标准 → ⑥ 上线后持续校准。核心纪律：不要猜该写什么，让失败告诉你该写什么。

配套微观测试工程（来源：[[Skill测试与质量保障]]）：三原则框架（Description 决定触发 → 正文决定质量 → 测试决定可靠性）、JSON 触发测试集、三种断言评估、Description 优化四步法、发布前 10 项检查清单。

## 来源

- [如何规范地定义一个 Skill · 完整指南](../raw/如何规范地定义一个 Skill · 完整指南/article.md) — Skill 定义、SKILL.md 标准写法、description 写法、常见错误与安全规范的完整指南
- [Agent Skills 终极指南：入门、精通、预测](../raw/Agent Skills 终极指南：入门、精通、预测/article.md) — 一泽Eze，Skill vs MCP 区分、零代码编写、渐进式披露机制
- [御舆：解码 Agent Harness](../raw/御舆：解码 Agent Harness/article.md) — 第11章：完整 SKILL.md frontmatter 字段、条件技能、MCP 技能安全边界
- [花叔不公开的写作 Skill，我逆向出来了](../raw/花叔不公开的写作 Skill，我逆向出来了/article.md) — 花叔写作 Skill 的逆向拆解、五层架构、质量控制机制
- [Skill 最佳实践：五种设计模式+六步打磨法](../raw/Skill 最佳实践：五种设计模式+六步打磨法（含完整实战过程和源码）/article.md) — 熊猫Jay，评测驱动打磨法、售前工时评估实战案例
