# Anthropic 又一篇 Agent 开发神文，新范式让 Token 消耗暴降 98.7%

**来源**: AI 寒武纪 (微信公众号)  
**URL**: https://mp.weixin.qq.com/s/XHuQghz8bHXqxes_0dS3FQ  
**原始发布时间**: 2026-04-12 (摄入日期)

---

## 核心问题：Agent 的两大"隐形税"

### 1. 工具定义过载

传统做法是将所有可用工具定义一次性加载到模型上下文中。当 Agent 连接数千个工具时，仅仅这些定义就可能消耗数十万 Token。

示例：
```
gdrive.getDocumentDescription: Retrieves a document from Google Drive
Parameters:
  documentId (required, string): The ID of the document to retrieve
  fields (optional, string): Specific fields to return
Returns: Document object with title, body content, metadata

salesforce.updateRecordDescription: Updates a record in Salesforce
Parameters:
  objectType (required, string): Type of Salesforce object
  recordId (required, string): The ID of the record to update
  data (required, object): Fields to update with their new values
```

### 2. 中间结果消耗

更致命的是，工作流中的每一个中间结果都必须经过模型上下文。

示例任务："从 Google Drive 下载会议纪要，并将其附加到 Salesforce 的潜在客户记录中"

传统流程：
1. 第一次工具调用：`gdrive.getDocument(documentId: "abc123")`
2. 结果返回：完整的会议纪要文本（可能 5 万 Token）全部加载进模型上下文
3. 第二次工具调用：`salesforce.updateRecord(...)`，在 data 字段中再次写入完整纪要

**结果**: 一份 5 万 Token 的纪要被模型处理了两次，可能直接超出上下文窗口限制。

---

## 解决方案：代码执行新范式

### 核心思想

Anthropic 提出名为**"代码执行"(Code Execution)**的新范式，建立在模型上下文协议（MCP）之上：

> **别再让模型直接调用工具了，让它写代码来调用工具**

### 实现方式

将 MCP 服务器呈现为代码 API，而不是直接的工具调用接口。

文件树结构示例：
```
servers/
├── google-drive/
│   ├── getDocument.ts
│   └── ... (other tools)
├── salesforce/
│   ├── updateRecord.ts
│   └── ... (other tools)
...
```

每个工具文件（如 `getDocument.ts`）内部封装了对 MCP 工具的实际调用。

### 新范式工作流

对于同样的"会议纪要"任务，Agent 生成的代码：

```typescript
// 从 Google Docs 读取纪要并添加到 Salesforce
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

### 颠覆性变化

| 特性 | 传统方式 | 代码执行方式 |
|------|----------|-------------|
| 工具加载 | 开局加载所有定义 | 按需浏览文件系统发现 |
| 数据流转 | 经过模型上下文两次 | 在代码变量中本地流转 |
| Token 消耗 | 15 万 | 2000 |
| 效率 | 基准 | 提升 98.7% |

---

## 代码执行带来的五大核心优势

### 1. 渐进式披露 (Progressive Disclosure)

模型无需预知一切。它们可以像人类程序员一样：
- 通过探索文件系统（`ls ./servers/`）发现可用服务
- 使用 `search_tools` 工具按需发现和学习工具的用法

### 2. 上下文高效的工具结果

处理海量数据时，Agent 在代码执行环境中进行过滤、转换和聚合，只将最终的小规模结果返回给模型。

示例：处理 10000 行数据的电子表格

```typescript
// 传统方式：返回 10000 行数据到上下文
TOOL CALL: gdrive.getSheet(sheetId: 'abc123')

// 代码执行方式：在环境中过滤，只返回摘要
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(row => row["Status"] === 'pending');
console.log(`发现 ${pendingOrders.length} 个待处理订单`);
console.log(pendingOrders.slice(0, 5)); // 只记录前 5 个供模型审查
```

### 3. 更强大的控制流

循环、条件判断、错误处理等复杂逻辑，用标准代码模式实现：

```typescript
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({ channel: 'C123456' });
  found = messages.some(m => m.text.includes('deployment complete'));
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('部署完成通知已收到');
```

这比"调用工具 - 休眠 - 调用工具"的循环更高效，减少了模型的"首个 Token"延迟。

### 4. 保护隐私的操作

默认情况下，所有中间数据都保留在代码执行环境中。

执行环境可以自动识别并"令牌化"敏感数据：
- Agent 写的代码处理 `row.email` 和 `row.phone`
- 如果尝试打印这些数据，模型看到的是 `[EMAIL_1]` 和 `[PHONE_1]`
- 真实数据在执行环境中安全地从 Google Sheets 流向 Salesforce，全程不经过模型

### 5. 状态持久化与技能 (Skills)

**状态持久化**：Agent 可以将中间结果写入文件，实现任务的中断和恢复。

```typescript
const leads = await salesforce.query(...);
const csvData = leads.map(l => ...).join('\n');
await fs.writeFile('./workspace/leads.csv', csvData);
```

**技能积累**：Agent 可以将成功的代码保存为可复用的函数。

```typescript
// In ./skills/save-sheet-as-csv.ts
export async function saveSheetAsCsv(sheetId: string) { ... }

// Later, in any agent execution:
import { saveSheetAsCsv } from './skills/save-sheet-as-csv';
const csvPath = await saveSheetAsCsv('abc123');
```

通过不断积累这样的技能，Agent 可以构建起强大的、可复用的高级能力工具箱。

---

## 结语

Anthropic 认为，上下文管理、工具组合、状态持久化这些问题在传统软件工程中都有成熟的解决方案。

**代码执行范式**正是将这些经过时间检验的工程模式应用于 AI Agent，让 Agent 以其最擅长的方式——编写代码——来更高效地与世界互动。

### 新的挑战

运行 Agent 生成的代码需要：
- 安全的沙箱环境
- 资源限制
- 监控机制

### 权衡收益

- Token 成本大幅降低
- 延迟缩短
- 工具组合能力提升

---

**参考**: https://www.anthropic.com/engineering/code-execution-with-mcp
