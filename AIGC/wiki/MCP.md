---
title: MCP (Model Context Protocol)
tags: [MCP, 协议, 工具调用, Anthropic, Agent, Bridge, IDE集成]
sources: [raw/MCP一篇就够了/article.md, raw/御舆：解码 Agent Harness/article.md, raw/Agent Skills 完全指南：从原理到实战彻底搞懂！/article.md]
updated: 2026-04-15
---

# MCP (Model Context Protocol)

## 定义

MCP（Model Context Protocol，模型上下文协议）是 Anthropic 于 2024 年 11 月发布的开放协议标准，定义了应用程序和 AI 模型之间交换上下文信息的方式。类比 USB-C 接口，MCP 让开发者以一致的方式将各种数据源、工具和功能连接到任意 AI 模型。

## 核心观点

- **MCP 是 prompt engineering 发展的产物**：手工将信息引入 prompt 越来越困难 → function call 出现但平台依赖性强 → MCP 提供统一的中间协议层（来源：[MCP一篇就够了](../raw/MCP一篇就够了/article.md)）
- **MCP 的三大优势**：生态（大量现成插件）、统一性（不绑定特定模型）、数据安全（敏感数据留在本地）
- **模型通过 prompt engineering 选择工具**：所有工具的名称、描述、参数以文本形式注入 system prompt，模型据此决定调用哪些工具

## 演进路径

```
手工 prompt → function call → MCP
（人工粘贴）  （平台依赖强）  （统一协议标准）
```

- **手工 prompt**：用户手动将数据库信息、文件内容粘贴到 prompt 中，随问题复杂度增加不可持续
- **function call**：OpenAI/Google 等平台引入，允许模型调用预定义函数；但各平台 API 实现差异大，切换模型需重写代码
- **MCP**：Anthropic 提出的统一标准，充当 AI 模型的"万能转接头"，任何支持 MCP 的模型都可以灵活切换

## 架构：Host / Client / Server

MCP 由三个核心组件构成：

| 组件 | 职责 | 示例 |
|------|------|------|
| **Host** | 接收用户提问，与 AI 模型交互 | Claude Desktop、Cursor |
| **Client** | Host 内置，负责与 MCP Server 建立连接 | MCP Client SDK |
| **Server** | 执行实际操作（读文件、查数据库等），返回结果 | filesystem、PostgreSQL server |

**调用流程**：用户提问 → Host → 模型分析 → 需要外部数据 → Client 连接 → Server 执行 → 结果返回 → 模型生成最终回答

## 工具选择原理

模型如何决定使用哪些工具？核心机制是 **prompt engineering**：

1. **工具描述注入**：所有已注册 MCP Server 的名称、功能描述（来自函数 docstring）和参数 schema 被格式化为文本，注入到 system prompt
2. **模型决策**：模型根据用户问题和工具描述，输出结构化 JSON 格式的工具调用请求
3. **执行与反馈**：客户端解析 JSON 执行对应工具，将结果重新发送给模型生成最终回复
4. **幻觉防护**：如果模型输出了无效的工具调用 JSON，客户端会 skip 掉

> 工具文档至关重要——模型通过工具描述文本来理解和选择工具，因此精心编写工具的名称、docstring 和参数说明至关重要。

## MCP Server 开发

MCP Server 可提供三种功能类型：
- **Resources（资源）**：类似文件的数据，可被客户端读取
- **Tools（工具）**：可被 LLM 调用的函数（需用户批准）
- **Prompts（提示）**：预编写模板，帮助完成特定任务

**最小开发实践**（Python + FastMCP）：

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("服务名")

@mcp.tool()
def my_tool() -> str:
    """工具描述（会被注入到模型的 prompt 中）"""
    return "result"

if __name__ == "__main__":
    mcp.run()
```

**开发流程**：环境配置（uv） → 编写 Server → `mcp dev` 测试（Inspector） → 配置 `claude_desktop_config.json` 接入 → 实际使用

**最佳实践**：向 LLM 提供 domain knowledge（SDK README + 示例代码）作为 context，描述需求让模型生成 MCP Server 代码。

## 生态资源

- [MCP 官方文档](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Official MCP Servers](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [MCP Chatbot DEMO](https://github.com/keli-wen/mcp_chatbot)

## MCP 在工具注册管线中的位置（来源：raw/御舆：解码 Agent Harness/article.md，Ch1/Ch3）

### 渐进式能力扩展第四级

Claude Code 的四级扩展模型中，MCP 是最外层的扩展机制：

```
Tool（核心开发者）→ Skill（高级用户）→ Plugin（生态开发者）→ MCP 服务器（第三方开发者）
```

MCP 的意义：Agent 的能力边界不是由原始开发者预设的，而是可以在运行时动态扩展的。

### 延迟工具发现（Deferred Tool Discovery）

当 MCP 服务器注册了大量工具时，全部发送给 API 会消耗大量 token（每个工具 schema 包含名称、描述和参数定义，50+ 工具时可能消耗数千 token）。

Claude Code 的解决方案：**ToolSearchTool 延迟发现机制**
- 不在初始系统提示中发送所有工具的完整 schema
- 只发送工具名称列表
- 模型通过 `ToolSearchTool` 按需加载工具的完整 schema

规则：MCP 工具**总是**延迟发现；`alwaysLoad` 标记的工具（如 AgentTool）不延迟。

### 工具过滤管线中的 MCP 处理

MCP 工具在发送给 API 之前经过以下处理：
1. **工具池组装**：内建工具 + MCP 工具合并，**按名称排序**去重（排序目的：确保 prompt 缓存稳定性）
2. **权限过滤**：`mcp__server__*` 通配符可以 deny 整个 MCP 服务器的所有工具
3. **延迟决策**：MCP 工具默认进入延迟发现队列

> 最佳实践：如果 MCP 服务器注册了 50+ 工具，启用延迟发现可以显著减少初始 prompt 的 token 消耗。见 [[工具系统五要素协议]]。

## Claude Code 中的 MCP 深度集成（源码级）

以下内容来自对 Claude Code 源码的深度分析，补充了协议实现细节。

### 八种传输协议

| 协议类型 | 延迟特征 | 适用场景 |
|---------|---------|---------|
| `stdio` | 最低（进程间管道） | 本地开发工具、文件系统操作（**首选**） |
| `sse` | 网络延迟 | 远程 HTTP 服务、云端部署 |
| `sse-ide` | 本地网络 | IDE 扩展专用（含 ideName 标识） |
| `http` | 网络延迟 | MCP 规范新协议，支持流式响应 |
| `ws` | 网络延迟 | 需要实时双向通信 |
| `ws-ide` | 本地网络 | IDE 扩展专用，低延迟双向通信 |
| `sdk` | 近零延迟 | SDK 进程内调用，豁免企业策略检查 |
| `claudeai-proxy` | 网络延迟 | Claude.ai 平台代理服务器 |

### 三段式工具命名：mcp__{server}__{tool}

- **命名空间隔离**：防止不同服务器同名工具冲突
- **权限独立**：`mcp__github__*` 权限规则不会影响内置工具或其他服务器
- **SDK 前缀跳过**：设置 `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` 后，MCP 工具可按名称覆盖内置工具
- **解析规则**：按第一个双下划线拆分，若服务器名含 `__` 会产生歧义（实践中极少见）

### 四层权限模型（深度防御）

1. **企业策略**：`deniedMcpServers`（黑名单，绝对优先）+ `allowedMcpServers`（白名单门控）
2. **IDE 工具白名单**：IDE 类型服务器只允许 `executeCode` 和 `getDiagnostics` 两个工具
3. **用户权限配置**：`.claude/settings.local.json` 中的 allow/deny 规则
4. **运行时确认**：默认 passthrough，每次弹窗确认

> ⚠️ 反模式：不要使用 `"mcp__*": "allow"` 一键全开，任何新连接服务器的工具都会被自动允许。

### 七个配置作用域

| 作用域 | 配置来源 | 管理级别 |
|--------|---------|---------|
| `local` | `.claude/settings.local.json` | 项目-个人（不提交版本控制） |
| `project` | `.claude/settings.json` | 项目-共享（提交版本控制） |
| `user` | `~/.claude/settings.json` | 用户全局 |
| `dynamic` | 运行时动态添加 | 会话级（临时） |
| `enterprise` | 企业管理配置 | 组织级（硬性约束） |
| `claudeai` | Claude.ai 平台连接器 | 平台级 |
| `managed` | 托管策略配置 | 管理级（硬性约束） |

### 插件去重签名机制

| 服务器类型 | 签名 |
|-----------|------|
| stdio | `stdio:${JSON.stringify([command, ...args])}` |
| 远程服务器 | `url:${原始URL}` |
| sdk | null（不做去重） |

去重优先级：手动配置 > 插件配置 > Claude.ai 连接器

### Bridge 系统（IDE 集成与远程控制）

Bridge 是 Claude Code 与外部世界双向通信的核心层（30+ 个模块）：

- **v1 适配器**：WebSocket 读取 + HTTP POST 写入（历史遗留）
- **v2 适配器**（推荐）：SSETransport 读取 + CCRClient 写入
  - **SSE 序列号延续**：重连时携带 lastSeq，服务器只发增量而非完整历史，避免"消息风暴"
  - **Epoch 管理和心跳**：区分旧连接延迟消息与新连接新消息
  - **多会话安全认证**：闭包提供每实例认证，互不干扰

**控制协议（5 种子类型）**：`initialize`（能力握手）、`set_model`、`set_max_thinking_tokens`、`set_permission_mode`、`interrupt`

**outbound-only 模式**：除 `initialize` 外的所有可变请求被拒绝，防止远端控制 CLI 会话。

**Bridge 权限四层门控**：订阅类型（必须 claude.ai 订阅）→ Profile 完整性 → 组织 UUID → 功能标志（`tengu_ccr_bridge`）

### IDE 工具白名单设计哲学

只允许 `mcp__ide__executeCode` 和 `mcp__ide__getDiagnostics`，原因：
- IDE 扩展是第三方代码，可能暴露危险工具（如 `deleteFiles`）
- 最小权限原则：IDE 集成只需执行代码和获取诊断两个核心能力
- 白名单在**工具发现阶段**执行，不在白名单的工具根本不进入工具注册表，模型甚至不知道它们的存在

## MCP 规模化局限性

MCP 的"按需调用"背后有隐性代价——仅"连接"就已在透支上下文额度（来源：[Agent Skills 完全指南](../raw/Agent%20Skills%20完全指南：从原理到实战彻底搞懂！/article.md)）。

### Token 开销量化

为让模型知道可用能力，每个 MCP Server 必须在对话开始前将所有工具的完整定义（名称、描述、参数 Schema、使用示例）一次性注入上下文：

- 每个工具约消耗 ~500 Token
- GitHub MCP 含 30+ 工具 → 仅此一个 Server 就消耗 ~15000-20000 Token
- 真实 Agent 连接多个 Server → 即使用户只问"1+1=?"，已烧掉数万 Token

### 注意力退化与 MCP Atlas 基准

工具过多导致 LLM"注意力"下降，降低工具调用准确性。MCP Atlas 基准测试结果：

| 条件 | 数据 |
|------|------|
| 服务器数 | 40+ |
| 工具总数 | 300+ |
| 最强模型（Claude Opus 4.5）准确率 | **62%** |

准确率会随工具数量增加进一步下降。对比 [[Skills-渐进式披露]]：40 个 Skills 的元数据仅需几千 Token，且模型每次只聚焦当前任务的指令。

### 未来格局预测

MCP 不会被淘汰，但需求会大幅减少（来源：宝玉）：

- **Agent 内置**少量核心能力（bash、read、edit、write）
- **少数通用 MCP Server** 负责远程连接（数据库、云 API、SaaS 集成）
- **大量 Skills** 封装标准工作流、连接本地知识库
- Skills 内部可编排 MCP 工具和其他 Skills，承担绝大部分"教 AI 怎么做事"的工作

> MCP 的不可替代价值在协议层——它统一了 AI 连接外部世界的标准接口；但"教 AI 干活"这件事正在向 Skills 迁移。

## 相关概念

- [[Context-Engineering]] — MCP 是上下文工程的具体实现手段，提供"按需加载"外部数据的标准化方式
- [[Harness-Engineering]] — MCP 作为 Agent 约束工程的工具调用层，是 Harness 的一个关键组件
- [[代码执行范式]] — MCP Server 可作为代码执行的载体
- [[Agent构建四阶段]] — MCP 在 Agent 构建第四阶段（Agent Loop）之上提供标准化工具对接
- [[Skills-系统架构与插件]] — MCP 技能的特殊安全策略（不执行 Shell 内联）
- [[SubAgent-与-Fork模式]] — MCP 工具在子智能体中的使用与权限

## 来源

- [MCP一篇就够了](../raw/MCP一篇就够了/article.md) — MCP 完整概念、架构、原理源码分析、开发实践与 Chatbot DEMO
- [御舆：解码 Agent Harness](../raw/御舆：解码 Agent Harness/article.md) — 第12章：Claude Code 中 MCP 的源码级实现，传输协议、安全架构、Bridge 系统
- [Agent Skills 完全指南：从原理到实战彻底搞懂！](../raw/Agent%20Skills%20完全指南：从原理到实战彻底搞懂！/article.md) — ConardLi，2026-01-27，MCP Atlas 基准（62%）、Token 开销量化、未来格局预测
