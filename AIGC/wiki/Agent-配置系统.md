---
title: Agent 配置系统
tags: [claude-code, 配置, 安全, 功能开关, 状态管理, 六层优先级]
sources: [raw/御舆：解码 Agent Harness/article.md]
updated: 2026-04-15
---

# Agent 配置系统

## 定义

Agent 配置系统是 Claude Code 的"基因"——在 Agent 启动前就已写定，决定了 Agent 能做什么、不能做什么、以及以何种方式去做。它由六层配置源逐层合并而成，并通过信任半径递减原则设计安全边界。

## 核心观点

- 配置是"基因"：在启动时加载，在运行时逐层生效，与生物基因的表达类比精确（来源：[御舆：解码 Agent Harness](../raw/御舆：解码%20Agent%20Harness/article.md)）
- 安全边界核心：`projectSettings` 在所有安全敏感检查中被系统性排除，防止供应链攻击
- 双层功能开关：编译时 `feature()` 消除死代码，运行时 GrowthBook 提供动态控制
- 状态管理极简哲学：34 行 Store 代码实现完整状态管理，"少即是多"

## 六层配置源优先级

完整优先级链（从低到高）：

```
pluginSettings → userSettings → projectSettings → localSettings → flagSettings → policySettings
```

| 配置源 | 文件路径 | 是否入 Git | 用途 |
|--------|----------|-----------|------|
| pluginSettings | 插件注册 | N/A | 基础默认值 |
| userSettings | `~/.claude/settings.json` | N/A | 个人全局默认值 |
| projectSettings | `.claude/settings.json` | 是 | 团队共享设置 |
| localSettings | `.claude/settings.local.json` | 否 | 个人本地覆盖 |
| flagSettings | CLI `--settings` 参数 | N/A | 一次性覆盖 |
| policySettings | 平台相关 managed-settings.json | N/A | 企业级锁定 |

## 合并规则

- **数组类型**：拼接并去重（如 `permissions.allow`），确保权限规则只增不减
- **对象类型**：深度合并，嵌套属性逐层覆盖
- **标量类型**：后者直接覆盖前者

**关键例外**：`policySettings` 使用"首个非空源胜出"（first source wins）而非深度合并，来源优先级：远程 API > MDM > managed-settings.json > HKCU 注册表。这保证了企业策略的确定性和可审计性。

## 安全边界设计

### 供应链攻击的威胁模型

`projectSettings` 是风险最大的配置源，因为它同时具备三个高风险属性：
1. **来源不可信**：可能来自克隆的第三方仓库
2. **自动加载**：进入目录后自动生效，无需确认
3. **可执行代码**：hooks 配置可执行任意 shell 命令

### 信任半径递减原则

```
信任级别：policySettings ★★★★★ > flagSettings/localSettings/userSettings ★★★★ > projectSettings ★★ > pluginSettings ★
```

在所有安全敏感检查中（`skipDangerousModePermissionPrompt`、`hasAutoModeOptIn` 等），系统注释一致标注：**projectSettings is intentionally excluded -- a malicious project could otherwise auto-bypass the dialog (RCE risk)**。

### 企业安全控制

- `allowManagedHooksOnly: true`：只允许托管 hooks 运行，阻止所有用户/项目/本地 hooks
- `strictPluginOnlyCustomization`：可选择性锁定 skills、agents、hooks、mcp 四个定制面
- 路径安全校验：拒绝相对路径、根路径、UNC 路径、Null 字节注入、用户目录外的路径

## 功能开关系统

### 编译时功能标志（feature()）

通过 bundler 死代码消除（dead code elimination）实现：当 `feature('FEATURE_NAME')` 为 false 时，整个代码分支从构建产物中移除。

主要功能标志及架构意义：

| 功能标志 | 架构方向 |
|---------|---------|
| KAIROS | 长驻会话，从"按需调用"到"持续运行" |
| DAEMON | 后台守护进程能力 |
| TEAMMEM | 从"个人工具"到"团队基础设施" |
| EXTRACT_MEMORIES | 后台记忆提取（详见[[Agent-记忆系统]]） |
| TRANSCRIPT_CLASSIFIER | 自动模式分类器 |

### 运行时实验框架（GrowthBook）

函数名 `getFeatureValue_CACHED_MAY_BE_STALE` 直接编码行为约束——值来自缓存，可能已过时。使用随机命名的实验标志（如 `tengu_passport_quail`）避免语义偏见。

**决策树**：功能发布时确定 → 编译时 feature()；需要运行时动态控制 → GrowthBook；需要 A/B 测试 → GrowthBook + 随机分组。

## 状态管理系统（AppState）

### Store 设计哲学

34 行泛型 Store，灵感来自 Zustand，三个核心方法：`getState`、`setState`、`subscribe`。

三个关键设计决策：
1. **不可变更新**：`setState` 接受 `(prev: T) => T`，通过 `Object.is` 比较引用
2. **DeepImmutable**：`AppState` 类型在编译时阻止直接修改状态字段
3. **Set-based 监听者**：自动去重，subscribe 返回取消函数（React cleanup 模式）

### AppState 结构分层

| 层 | 字段 | 变化频率 |
|----|------|---------|
| 配置层 | settings, verbose, mainLoopModel | 启动时确定，很少变化 |
| 权限层 | toolPermissionContext | 随工具调用动态变化 |
| 集成层 | mcp.*, plugins.* | 由子系统初始化流程管理 |
| 通信层 | replBridge*, teamContext | 跨进程/跨 Agent 状态 |
| 执行层 | speculation, agentDefinitions | 运行时动态创建和销毁 |

### React Context 封装

- `useAppState(selector)`：使用 `useSyncExternalStore` 精细化订阅，只有 selector 返回值变化才重渲染
- `useSetAppState()`：只获取 setState 函数，不订阅任何状态，永远不触发重渲染

**最小订阅原则**：分别订阅具体字段，避免 `useAppState(s => s)` 这种反模式。

## 配置策略模式

| 模式 | 配置分布 |
|------|---------|
| 个人-团队分离 | userSettings 放个人偏好，projectSettings 放团队规则，localSettings 放个人覆盖 |
| CI/CD 专用 | flagSettings 注入一次性配置，不污染持久化文件 |
| 企业统一管控 | policySettings 锁定安全项，projectSettings 定制非安全行为 |

## 相关概念

- [[Agent-钩子系统]] — hooks 配置是配置系统的重要组成部分，遵循相同的合并规则
- [[Agent-记忆系统]] — 记忆路径的 projectSettings 排除机制是配置安全策略的具体应用
- [[Harness-Engineering]] — 配置系统是 Harness 的核心基础设施之一
- [[Claude-Code-系统化用法]] — 用户视角的配置实践
