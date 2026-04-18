---
topic: Skills 体系
score: 92
last_evaluated: 2026-04-17
wiki_pages: 18
---

# Skills 体系 · 进化档案

## 覆盖度评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 广度 | ⭐⭐⭐⭐⭐ | 18 页覆盖从哲学、分类学、定义规范到源码分析、官方工作流、评估体系的全谱系 |
| 深度 | ⭐⭐⭐⭐½ | 源码分析+官方 Eval 架构+六种 JSON Schema 达到工程级深度，仅安全维度缺深潜 |
| 实用性 | ⭐⭐⭐⭐⭐ | 五种设计模式+六步打磨法+测试三原则+官方创建闭环，从零到生产全链路可操作 |
| 时效性 | ⭐⭐⭐⭐⭐ | 全部内容更新至 2026-04-17，含最新 Anthropic 官方实践 |
| 交叉引用 | ⭐⭐⭐⭐⭐ | 页面间互联紧密，形成完整知识网络 |

## 现有页面（18 页）

- Skill定义规范 · Skill实战踩坑指南 · Skill-设计模式与打磨法 · Skills-设计哲学 · Skills-分类学
- Skills-系统架构与插件 · Skill框架源码分析 · Skills-渐进式披露 · Skills-多Skill协作
- Skills-Github化 · Skills-逆向建模 · Skill-vs-Prompt · Skills 应用场景-科普 · Skills 生态时间线
- Skill测试与质量保障 · Skill-Creator-官方工作流 · Skill-Eval-系统架构
- MCP

**关联页面**（归属其他分类但强相关）：
- Tool-vs-Skill-vs-SubAgent选型（Agent 高级模式）
- 多步骤Skill稳定执行 · Output-Contract · Checkpoint机制（多步骤 Skill 工程）

## 进化向量

### → Skill 测试与调试方法论
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🟢 成熟
**价值**: 从 88 → 92（+4），已全部实现
**线索**: ✅ 已覆盖：JSON 测试集格式、三种断言类型、Description 优化四步法、发布前 10 项检查清单（Skill测试与质量保障）；官方评估驱动迭代闭环、并行盲评对比（Skill-Creator-官方工作流）；六种 JSON Schema、Grader/Comparator/Analyzer 三种评估智能体、Eval Viewer（Skill-Eval-系统架构）。剩余缺口：`claude --debug` 排查链路详解、回归测试自动化脚本——属于锦上添花，不影响成熟判定。

### → Skill 安全与 Prompt Injection 防护
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🔴 空白
**价值**: 从 92 → 95（+3）
**线索**: 第三方 Skill 信任评估框架；恶意 references/scripts 注入攻击面分析；MCP 来源 Skill 安全隔离深入分析；企业环境 Skill 审计与合规流程。目前仅 Skills-系统架构与插件 中提到"MCP 技能不执行 Shell 命令内联"这一安全边界，缺系统性安全分析。

### → 跨平台 Skill 生态对比
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🔴 空白
**价值**: 从 92 → 95（+3）
**线索**: Cursor Rules vs Claude Skills vs Windsurf vs Codex 对比；Skill 概念在不同 AI 编程工具中的异同；可移植性与迁移策略；各平台 Skill 市场/分发机制对比

### → Skill 效果度量与 ROI
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🟡 有基础覆盖
**价值**: 从 92 → 95（+3），已实现 +1
**线索**: 部分覆盖：Skill-Eval-系统架构 提供定量评估框架（timing.json 记录 token 消耗）；Skill-设计模式与打磨法 含售前工时评估实战案例展示 ROI 思维。仍缺：团队级采用率追踪体系、Skill 退役判断标准、长期 Token 成本收益分析模型

### → Skill 版本管理与生命周期
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🔴 空白
**价值**: 从 92 → 94（+2）
**线索**: 版本迁移策略（v1→v2 平滑过渡）；多人协作 Skill 冲突解决；废弃（deprecation）流程；Skill 发布/分发/更新机制

## 进化日志

| 日期 | 触发 | 事件 | 变化 |
|------|------|------|------|
| 2026-04-17 | evolve 主动 | 全面审计：+3 新页面（测试质量保障、官方工作流、Eval 架构） | 88→92 分，测试向量 🟡→🟢，度量向量 🔴→🟡 |
| 2026-04-17 | ingest 被动 | ingest「Claude Skill 实战：测试、评估与迭代优化」填补测试盲区 | 85→88 分，测试向量 🔴→🟡 |
| 2026-04-17 | query 被动 | 查询"如何写好合规高效的 Skill"时系统评估覆盖度 | 建立基线 85 分，识别 5 个进化向量 |
