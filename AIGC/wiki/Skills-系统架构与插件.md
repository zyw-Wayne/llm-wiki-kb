---
title: Skills 系统架构与插件
tags: [Skill, 插件, 加载引擎, 动态发现, 条件技能, 安全, Claude Code]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Skills 系统架构与插件

## 定义

Claude Code 的技能系统是一个多层次的扩展机制，允许用户通过 Markdown 文件定义可复用的 prompt 模板，也允许开发者通过 TypeScript 代码注册编译时内置的技能。设计目标：**零配置可用，有配置强大**。

## 核心观点

- 内置技能文件采用**惰性单例**提取：首次调用时才提取到磁盘，并发调用共享同一 Promise（`??=` 操作符）（来源：raw/御舆：解码 Agent Harness/article.md）
- **MCP 技能不执行 Shell 命令内联**：MCP 来自不可信远端，Shell 内联是关键安全边界
- 去重策略用 `realpath` 解析符号链接，按首次出现保留，防止重复加载
- 循环依赖通过"发布-查找"中间注册模块打破（而非动态 import，因为打包二进制中非字面量动态 import 路径解析失败）

## 内置技能清单

| 技能名称 | 用途 | 分层 |
|---------|------|------|
| `verify` | 验证代码变更的正确性 | 代码质量层 |
| `simplify` | 代码简化与重构审查 | 代码质量层 |
| `debug` | 调试辅助，提供诊断思路 | 代码质量层 |
| `batch` | 批量文件处理 | 工作流层 |
| `stuck` | 帮助模型走出困境 | 工作流层 |
| `skillify` | 将 prompt 转换为可复用技能 | 工作流层 |
| `update-config` | 配置 settings.json | 配置层 |
| `remember` | 记忆管理（CLAUDE.md 条目） | 配置层 |
| `keybindings-help` | 键盘快捷键自定义帮助 | 配置层 |
| `loop` | 循环执行（需 AGENT_TRIGGERS） | Feature-gated |
| `claude-api` | Claude API 应用（需 BUILDING_CLAUDE_APPS） | Feature-gated |
| `schedule` | 定时远程任务（需 AGENT_TRIGGERS_REMOTE） | Feature-gated |

## 技能注册关键字段

| 字段 | 说明 |
|------|------|
| prompt 生成器 | 延迟执行，仅在技能被调用时才运行 |
| `files` | 引用文件集合，首次调用时提取到磁盘 |
| `isEnabled` | 运行时门控回调 |
| `context: 'fork'` | 标记技能在独立子进程中执行 |

### 文件提取安全设计

| 安全措施 | 防护目标 |
|---------|---------|
| `O_NOFOLLOW` | 符号链接劫持 |
| `O_EXCL` | TOCTOU 竞态 |
| `0o700` 目录 | 目录遍历 |
| `0o600` 文件 | 文件泄露 |
| `??=` 单例 | 并发竞态 |

## 加载路径（五个来源）

加载引擎并行扫描五个来源，合并后按 `realpath` 去重：

```
[1] managedSkillsDir     管理策略级（企业，最高优先级）
[2] userSkillsDir        用户级 ~/.claude/skills/
[3] projectSkillsDirs    项目级 .claude/skills/
[4] additionalDirs       --add-dir 额外目录
[5] legacyCommands       旧版 /commands/ 目录（兼容）
```

分层覆盖优先级（高 → 低）：managed > project > user > plugin > bundled

各层详情：

- **管理策略级**：企业管理员集中下发，`CLAUDE_CODE_DISABLE_POLICY_SKILLS` 可禁用
- **用户级**：`~/.claude/skills/`，跨所有项目可用
- **项目级**：`.claude/skills/`，仅当前项目生效
- **旧版命令目录**：`.claude/commands/`，单文件 `.md` 和目录格式均支持，标记为 `commands_DEPRECATED`

## SKILL.md Frontmatter 完整字段

```yaml
---
name: my-skill-display-name
description: 技能描述（必填）
when_to_use: 使用场景描述
arguments: arg1 arg2 arg3
argument-hint: "[arg1] [arg2] [arg3]"
allowed-tools: [Bash, Read, Write]
model: claude-sonnet-4-20250514
effort: high
user-invocable: true
disable-model-invocation: false
context: fork          # 或 inline
agent: code-builder
version: "1.0"
paths:                 # 条件激活路径
  - "src/**/*.ts"
hooks:
  pre:
    - command: npm run lint
  post:
    - command: echo "done"
---
```

## 参数替换机制

执行顺序：命名参数 → 索引参数 → 完整参数 → 无占位符时追加

```
/my-skill hello world

$greeting → "hello"（命名参数，需 arguments: "greeting name"）
$name     → "world"
$1        → "hello"（索引参数）
$2        → "world"
$ARGUMENTS[0] → "hello"
$ARGUMENTS    → "hello world"（完整参数）
```

环境变量替换：
- `${CLAUDE_SKILL_DIR}` → 技能目录绝对路径
- `${CLAUDE_SESSION_ID}` → 当前会话 ID

## LoadedFrom 类型与安全策略

| 来源 | 信任等级 | Shell 内联 |
|------|---------|-----------|
| bundled | 最高 | 允许 |
| managed | 高 | 允许 |
| skills | 中 | 允许 |
| plugin | 中 | 允许 |
| commands_DEPRECATED | 中 | 允许 |
| **mcp** | **低** | **禁止** |

## 动态技能发现

`discoverSkillDirsForPaths`：当模型通过 Read/Write/Edit 等工具访问文件时，沿文件路径向上查找 `.claude/skills/` 目录。

```
Read(src/sub/deep/file.ts)
  → 从 src/sub/deep/ 开始向上遍历至 cwd
  → 检查每层是否存在 .claude/skills/
  → 跳过 gitignored 目录（防止 node_modules 中的技能被加载）
  → 收集新发现目录（按深度排序，最深优先）
  → addSkillDirectories() 加载并合并到 dynamicSkills Map
```

**安全要点**：被 `.gitignore` 忽略的目录中的技能自动跳过。

**Monorepo 场景**：每个子项目的 `.claude/skills/` 只在操作该目录下的文件时才激活，实现"按位置激活"。

## 条件技能

带有 `paths` frontmatter 的技能不立即激活，存入 `conditionalSkills` Map：

```
启动时加载 → 有 paths 字段? 
  → 存入 conditionalSkills Map
  → 模型操作文件时检查路径是否匹配
  → 匹配则激活，存入 dynamicSkills + 永久标记已激活
```

**永久标记原因**：防止缓存清理后技能"消失"，导致会话中行为不一致。

## 插件系统

### 目录结构

```
my-plugin/
├── plugin.json       # 插件清单（可选）
├── commands/         # 自定义 slash 命令（技能）
├── agents/           # 自定义 AI agents
└── hooks/
    └── hooks.json    # Hook 配置
```

### 缓存优先加载策略

```
收集所有插件来源 → 检查缓存
  命中 → 直接返回
  未命中 → 解析 plugin.json → 加载 commands/agents/hooks
         → 收集错误（不中断加载）→ 存入缓存 → 返回 PluginLoadResult
```

**容错设计**：即使某个组件加载失败，其他组件仍正常加载。错误收集到 `errors` 字段，不中断整体启动。

### MCP 技能的间接注册

通过"发布-查找"（publish-lookup）中间注册模块打破 skills ↔ mcp 模块的循环依赖：

```
skills module → 向 registry module publish(buildFn)
mcp module    → 从 registry module lookup(buildFn)
registry module 是依赖图的叶子节点（无自身依赖）
```

## 技能组合模式

### 模式一：技能链

```
/gen-api POST orders  →  /verify  →  /remember "约定"
```

### 模式二：条件技能 + 智能体

条件技能（按文件类型激活）→ 自定义智能体引用该技能 → 附属参考文件

### 模式三：技能 + Hook

```yaml
hooks:
  pre:
    - command: npm run type-check   # 执行前类型检查
  post:
    - command: npm run test --related  # 完成后运行相关测试
```

## 反模式

- **"万能技能"**：超过 200 行的技能应拆分，越大越不确定，越难调试和复用
- **在 node_modules 中放置技能**：gitignore 过滤是防线，但应明确将 `**/.claude/skills/` 加入 `.gitignore`

## 相关概念

- [[Skill定义规范]] — SKILL.md 标准写法、description 写法原则
- [[Skill实战踩坑指南]] — 实战踩坑经验
- [[Skills-渐进式披露]] — 三级加载机制（元数据/指令/资源）
- [[Skills-多Skill协作]] — 多 Skill 协同工作模式
- [[Skills-设计哲学]] — Skills 设计哲学
- [[SubAgent-与-Fork模式]] — 智能体中的 skill 预加载
- [[MCP]] — MCP 技能的特殊安全策略
