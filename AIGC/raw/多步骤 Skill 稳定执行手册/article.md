---
title: 多步骤 Skill 稳定执行手册
source: https://mi.feishu.cn/docx/PEldd6YTMomHvwxzoxKcLVAEnQU
type: feishu
ingested: 2026-04-16
---

# 多步骤 Skill 稳定执行手册

> 这是一份多步骤SKILL稳定执行手册，目标只有一个：让第一次接触多步骤 Skill 的同学也能一步步读懂，并最终写出真正稳定可执行的多步骤 Skill。

### 这份手册适合谁
- 第一次写多步骤 Skill 的同学
- 在做 Agent / Workflow / MCP 编排的人
- 负责验收与稳定性治理的人
- 想把"能跑"提升到"稳定跑"的团队

### 读完你会掌握
- 为什么多步骤 Skill 容易失败
- 怎么拆步骤、定契约、做验证
- Supervisor / Worker / Verifier 怎么分工
- 怎么把流程做成可恢复、可审计、可复盘

### 先记住一句话
> 多步骤 Skill 的本质，不是"把很多动作串起来"，而是"把一系列可验证、可恢复、可审计的动作稳定地串起来"。

---

## 第一章：为什么多步骤 Skill 容易失败

### 1.1 五个最常见的失败根源

| # | 根源 | 典型表现 | 本质问题 | 对策 |
|---|------|---------|---------|------|
| 1 | 步骤定义太粗 | Agent 自己发明步骤 | 任务不可验证 | 原子化拆解 |
| 2 | 缺少中间验证 | 错一步，后面全错 | 错误放大 | 每步后立刻验证 |
| 3 | 上下文丢失 | 子 Agent 拿到碎片信息 | 约束没传完整 | 完整上下文注入 |
| 4 | 没有 Checkpoint | 一挂就只能重跑 | 无法恢复 | 每步保存状态 |
| 5 | 缺少监督角色 | 做完了但其实没验收 | 无闭环 | 引入 Supervisor / Verifier |

### 1.2 五个反面例子

#### 例子 1：步骤太粗
反例：`完成认证`
问题：Agent 可能跳过真正的登录流程，直接假设已经拿到 token。
正例：`调用 auth login --domain calendar，并验证返回 token 非空且未过期`

#### 例子 2：没做中间验证
Step 2 拉仓库信息返回空数组，但没人检查；Step 4 配 CI 时直接报 `branch_name is required`。
问题不在 Step 4，而是在 Step 2 没及时拦住。

#### 例子 3：上下文没传完整
Supervisor 只告诉 Worker：`去给项目写测试`。
但没有告诉它：覆盖率要 > 80%、不允许改源码、要输出 `coverage.xml`。
Worker 只能自己猜，最后结果很容易偏。

#### 例子 4：没有 Checkpoint
一个 7 步流程跑到 Step 5 挂了，因为没有中间状态保存，只能从 Step 1 全部重跑。

#### 例子 5：没有监督
Worker 说"我写完了"，但其实没跑测试、没看覆盖率、没检查最终产物。

### 1.3 先建立正确心智模型
> 单步骤任务，核心是"做出来"。多步骤任务，核心是"每一步都做对，而且后面的人知道前面的人到底做了什么"。

---

## 第二章：先把核心概念讲清楚

### 2.1 什么叫原子化步骤
定义：每个步骤只做一件事，而且必须可验证。

| 类型 | 示例 | 是否合格 |
|------|------|---------|
| 模糊步骤 | 完成认证 | 不合格 |
| 原子步骤 | 调用认证接口获取 token，并检查 token 字段存在 | 合格 |
| 模糊步骤 | 把数据整理出来 | 不合格 |
| 原子步骤 | 拉取第一页数据，校验 `items` 为数组，保存 raw JSON | 合格 |

### 2.2 什么叫 Checkpoint
定义：每步执行完成后保存的状态快照。至少包含：当前步骤、当前状态、已产生产物、关键返回值摘要、下一步依赖、产物 TTL。
作用：失败后能恢复、中断后能续跑、复盘时能追溯。

### 2.3 什么叫真实工程证据
定义：能证明任务真的推进了的客观证据。如：新生成的文件、测试结果、代码 diff、日志片段、API 返回结果、`progress.md` 的事实性更新。
不算证据："session 还在跑"、"我觉得差不多了"、"模型看起来理解了"。

### 2.4 什么叫 Output Contract
v4.0 最重要的概念之一。定义：最终输出的固定约束，至少应定义字段结构、字段类型、单位、精度、排序、时间范围、去重口径、空值语义、错误语义。
一句话理解：执行链路可以复杂，但最后产物不能"每次长得不一样"。

### 2.5 什么叫 Final Output Schema
推荐最小结构：
```json
{
  "status": "success | partial_success | failed",
  "summary": "一句话结论",
  "result": {},
  "artifacts": [],
  "warnings": [],
  "errors": [],
  "meta": {
    "run_id": "string",
    "skill_version": "string",
    "schema_version": "string",
    "contract_version": "string"
  },
  "next_action": "string"
}
```

### 2.6 什么叫 Fail-Closed
定义：当关键字段不完整、关键验证不通过时，不允许把结果包装成成功。少了非关键字段可以 `partial_success`，关键步骤失败必须 `failed`。

### 2.7 什么叫 No Invention
定义：不允许模型自己脑补缺失信息。禁止：工具没返回的字段自己补上、时间范围不明确时偷偷用默认值、结果不完整却写成确定结论。

---

## 第三章：三种执行模式怎么选

### 3.1 模式 A：单 Agent 顺序执行
适用：步骤数 < 3，每步都是确定性动作，副作用很小或没有。
流程：确认 → 执行 → 输出校验 → 完成

### 3.2 模式 B：Supervisor + Worker + Verifier
适用：步骤数 >= 3，有中间产物，路径不完全确定，需要重试、恢复、验收。
- Supervisor：负责计划、验收、纠偏
- Worker：负责执行动作、生成产物
- Verifier：负责独立校验结果是否真的达标

### 3.3 模式 C：工程级工作流
适用：软件开发、大型改造、多 issue / 多阶段推进。
流程：workflow-plan → issue/plan → issue/queue → issue/execute → verify → close

### 3.4 决策树
```
步骤数 < 3？
  → 是：模式 A
  → 否：是软件工程任务？
      → 是：模式 C
      → 否：模式 B
```

---

## 第四章：完整执行流程

### Phase 0：判断任务类型
- 查询型（获取数据并展示，无副作用）→ 可走简化路径
- 构建型（写代码、建系统，有持久化副作用）→ 走完整流程
- 管道型（多步 API 调用 + 数据转换）→ 重点在每步验证

### Phase 1：Confirm，确认意图
防止一开始就做错。推荐格式：
```
我理解你要做的是：1... 2...
约束：... 输出：... 验收：... 失败条件：...
请确认。
```

### Phase 2：Freeze Contract，冻结契约
v4.0 最强调的一步，冻结两份契约：
- **Execution Contract**：步骤清单、步骤依赖、工具边界、超时预算、重试预算、并发限制
- **Output Contract**：输出字段、单位、精度、时间范围、排序规则、去重口径、空值语义

### Phase 3：Create Plan，拆计划
好计划至少包含：步骤、依赖、每步验证方式、每步超时、是否可并行。

### Phase 4：Approval Gate，获得批准
有副作用的动作之前必须过门（改文件、写系统、发消息、部署、更新文档）。

### Phase 5：Execute，执行并持续保存状态
每完成一步，立刻做 5 件事：更新进度、保存 Checkpoint、记录证据、标记当前状态、进入下一步。

### Phase 6：Validate，四层验证

| 层级 | 验证什么 | 怎么看 |
|------|---------|--------|
| L1 | 工具返回验证 | HTTP 状态码、exit code、ok 字段 |
| L2 | 步骤契约验证 | 输出是否满足 schema |
| L3 | 业务逻辑验证 | 创建后能读到、聚合值是否正确 |
| L4 | 最终输出验证 | 最终输出是否满足 Output Contract |

### Phase 7：Review 与 Complete
收口动作：输出最终结论、保存最终报告、清理临时状态、关闭 Worker、保留可追溯产物。

---

## 第五章：三个完整例子

### 例子 A：查天气（模式 A）
调天气接口 → 读取结果 → 输出。不需要 Supervisor，不需要复杂 Checkpoint。

### 例子 B：生成日报（模式 B，管道型）
确认范围 → 冻结 Output Contract（金额单位/日期格式/排序/空值）→ 拉多源数据 → 做 join → 生成结果 → 四层验证。

### 例子 C：给项目补单测（模式 B/C）
Supervisor 写 task-brief.md 明确验收标准（覆盖率 > 80%，不改源码）→ Worker 写测试跑测试 → Verifier 独立检查覆盖率。
Correction Loop：覆盖率不达标时推回同一个 Worker session 补齐，不新开 session。

---

## 第六章：Supervisor / Worker / Verifier 怎么协作

### 6.1 职责分工

| 角色 | 核心职责 | 最怕做错什么 |
|------|---------|-------------|
| Supervisor | 发指令、验收、纠偏、收口 | 把"还在跑"当"推进正常" |
| Worker | 执行具体动作、生成产物、写进度 | 跳过契约、跳过验证 |
| Verifier | 独立判断结果是否达标 | 用主观感觉代替证据 |

### 6.2 Supervisor 核心原则
- 看真实工程进度，不看假象
- 验收不过时，推回同一个 Worker session
- 不要高频轮询（默认 5 分钟）
- 只允许两种终态：`accepted` 或 `failed`

### 6.3 Worker 核心原则
- 每一步都要可验证
- 每一步都要留下产物和证据
- 不要自己改目标
- 不要在缺关键输入时硬跑

### 6.4 Verifier 核心原则
- 不替 Worker 补作业
- 不看 session 是否活着，只看产物是否达标
- 不接受缺关键字段的结果
- 决策必须绑证据

### 6.5 最小产物集合
```
reports/<slug>/task-brief.md
reports/<slug>/progress.md
reports/<slug>/review.md
reports/<slug>/final-output.json
```

### 6.6 每轮 review 标准格式
```markdown
## Review <timestamp>
### 已完成 - <本轮确认完成的事实>
### 未完成 - <仍缺失的验收项>
### Decision - accepted | continue | failed
### Next Action - <下一步修正方向>
```

---

## 第七章：输出稳定性

### 7.1 执行稳定 ≠ 输出稳定
流程能跑完但输出不稳定：今天少字段、明天排序变了、后天单位变了、warnings 有时是字符串有时是数组。

### 7.2 Final Output Schema 必须固定
最终结果会被用户看、下游系统读、下一轮 Agent 再消费、审计和复盘时回看。

### 7.3 三种终态
- `success`：关键步骤和关键字段都通过
- `partial_success`：有结果，但不完整或有关键限制
- `failed`：无法产出可信结果

### 7.4 必须 fail-closed 的情况
关键字段缺失、关键验证未通过、口径不明确、汇总值与明细不一致、数据源冲突未解决。

### 7.5 输出前 6 个检查
结构对不对、字段齐不齐、单位和精度对不对、汇总值是否可复算、summary 是否准确、artifacts 是否真实存在。

---

## 第八章：错误与对策

### 执行类错误
| 错误 | 后果 | 正确做法 |
|------|------|---------|
| 步骤定义太粗 | Agent 自己发明步骤 | 步骤原子化 |
| 跳过中间验证 | 错误逐级放大 | 每步后立刻验证 |
| 没有 checkpoint | 失败后全量重来 | 每步保存状态 |
| 超时没设定 | 永久等待 | 每步设 timeout |

### Supervisor 类错误
| 错误 | 后果 | 正确做法 |
|------|------|---------|
| 只看"还活着" | 假性正常 | 看真实证据 |
| 验收不过就新开 session | 上下文丢失 | 推回同一个 Worker |
| 轮询过于频繁 | 资源浪费、误判 | 默认 5 分钟 |
| 完成后不收口 | 资源泄漏 | 关闭 Worker、Supervisor 自停 |

### 外部依赖类错误
| 错误类型 | 正确做法 |
|---------|---------|
| HTTP 500 | 指数退避重试 |
| 401 / 403 | 进入认证恢复流程 |
| 空结果 | 区分没数据还是参数错 |
| 参数类型错误 | 执行前 schema 校验 |
| 网络超时 | 超时后 abort 并记录 |

---

## 第九章：模板与检查清单

### 最小 Skill 模板
```markdown
## 任务目标
<一句话描述>

## 步骤清单
| # | 步骤 | 依赖 | 验证方式 | 超时 |
|---|---|---|---|---|
| 1 | 确认输入 | 无 | 输入完整 | 10s |
| 2 | 获取数据 | 1 | 返回非空 | 30s |
| 3 | 处理数据 | 2 | schema 正确 | 20s |
| 4 | 输出结果 | 3 | 最终结构正确 | 10s |
```

### Output Contract 模板
```markdown
## Output Contract
- Schema Version: 1.0.0
- Timezone: Asia/Shanghai
- Date Format: YYYY-MM-DD
- Number Precision: ratio=2, amount=0
- Sort Order: region asc
- Null Policy: missing=null, empty=[]
- Dedupe Key: record_id
```

### 执行前检查清单
- 步骤数是否真的 >= 3
- 步骤是否原子化
- 每步是否可验证
- 是否定义了 Output Contract
- 是否定义了 Checkpoint
- 是否定义了 timeout / retry / 并发预算
- 是否定义了终态与失败条件

### 执行中检查清单
- 每步是否已写入 Checkpoint
- 是否有真实证据
- 是否发生口径漂移
- 是否命中 Stop-the-line 条件
- 是否需要触发 Correction Loop

### 完成后检查清单
- 四层验证已完成
- Final Output Schema 校验通过
- 最终报告已落盘
- 临时状态已清理
- 结果已告知用户

---

## 第十章：实操路径

1. 先判断任务属于模式 A / B / C 哪一种
2. 把任务拆成原子步骤
3. 在执行前先冻结 Output Contract
4. 给每一步配上验证方式
5. 给每一步配上 Checkpoint
6. 对有副作用的动作加 Approval Gate
7. 用 Supervisor / Worker / Verifier 做分工
8. 用四层验证判断结果是否达标
9. 用 `success / partial_success / failed` 明确终态
10. 最后保留产物、报告、证据和复盘结论

> 一句话收尾：不要追求"Agent 看起来很聪明"，要追求"这条链路即使换个人来看，也知道每一步做了什么、为什么对、哪里失败、怎么恢复"。

---

## 附录：参考文档
本融合版整合并吸收了以下两篇文档的优势：
- 多步骤 Skill 稳定执行手册 v2.0
- 多步骤 Skill 稳定执行手册 v3.0
