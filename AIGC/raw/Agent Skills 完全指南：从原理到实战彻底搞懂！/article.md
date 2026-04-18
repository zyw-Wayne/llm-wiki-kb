---
title: "Agent Skills 完全指南：从原理到实战彻底搞懂！"
author: "ConardLi"
account: "code秘密花园"
date: "2026年1月27日 08:38"
source: "https://mp.weixin.qq.com/s/VQSRPTf5bOyA1bjS2JH5Kw"
---

# Agent Skills 完全指南：从原理到实战彻底搞懂！

**作者**：ConardLi  
**公众号**：code秘密花园  
**发布时间**：2026年1月27日 08:38  
**原文链接**：[Agent Skills 完全指南：从原理到实战彻底搞懂！](https://mp.weixin.qq.com/s/VQSRPTf5bOyA1bjS2JH5Kw)

---
Agent Skills 最近非常的火，它是既 MCP 后 Anthropic 推出的又一个 Agent 领域的行业标准。

它的成长路线和 MCP 也非常像，25 年 10 月份发布时只有 Anthropic 自家产品支持，后来 Cursor、Codex、Opencode、Gemini CLI 等产品看到了 Skills 的优势于是纷纷开始支持。

再后来社区开始涌现大量的开源 Skills 以及 Skills 开放市场，当下大家已经默认 Skills 成为了又一个扩展 Agent 能力的标准实践。

简单来说，Skills 的作用就是将那些重复性的、专业的流程进行打包封装。当你需要使用某种能力时，不再需要像过去那样每次都去查阅手册或重新输入冗长的提示词，而是像调用工具一样直接使用。

在本篇文章中，我们将从浅入深，和大家一起学习以下知识：

1. Skills 入门理解：Skills 到底是什么？长什么样？怎么工作的？
2. Skills VS MCP：Skills 和 MCP 的区别是什么，MCP 会被淘汰吗？
3. Skills 初步尝试：去哪里找 Skill？怎么使用 Skill？怎么自己创建一个 Skill？
4. Skills 实战使用：如何用 Skills 实现外部知识检索？比传统 RAG 的优势在哪？
5. Skills 安全分析：Skills 的安全性如何？使用它有哪些风险？

## 一、 Skills 入门理解

### 1.1 Skills 到底是什么？
在传统的 AI 聊天模式中，AI 的能力取决于：

- 它原本学过什么（训练数据）
- 你临时在对话框里告诉它什么（提示词、工具、记忆）
这就像你招了个什么都懂一点的实习生，每次干活你都得重新教一遍。

而 Agent Skills 带来了一种全新的玩法：模块化能力插件。

你可以把 Claude（支持 Skills 的客户端）想象成一个超级大脑，而 Agent Skills 就是给这个大脑安装的外接工具箱。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgc2HP2egY8Q4CvoqWknJ3YD6op1WvJuqkbtevTNdjWI0QoYDYsokL4zw/640?wx_fmt=jpeg&from=appmsg&watermark=1)
这个工具箱里不仅有工具本身，还包含了详细的 “官方使用说明书”，大脑不需要理解具体有哪些工具以及工具的用法是什么，只需要在需要使用某个工具时查看工具说明书，再把工具拿出来用。

### 1.2 Skills 长什么样？
Agent Skills 的官方文档中强调了一个核心关键词：File-system based（基于文件系统）。

如果你写过代码，可能很容易理解。

要编写一个程序，并不一定所有代码都是我们自己写的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcyuSic0SxicalbYraxicrNSm8BnwsthnLYfwMWzEPTyd6Z6m0l1JYMFPwA/640?wx_fmt=jpeg&from=appmsg&watermark=1)
我们可能会通过 import xxx 来引入一些外部包，这些包存放在固定的位置（如 node_modules）。

当程序需要调用这些包的能力时，就会从指定文件夹取出对应的代码然后执行。

Agent Skills 也是类似的逻辑，每个 Skill 都是一个实实在在存在的文件夹，它存放在一个固定的位置（如 .claude/skills）这个文件夹里装着下面几样东西：

1. 指令（SKILL.md）： 告诉 AI 怎么干活的 SOP。
2. 参考（reference）： 更详细的参考文档（可选）。
3. 脚本（scripts）： 比如 Python 代码，让 Skill 也能调用外部能力（可选）。
4. 资源（assets）：图片、模版等可能使用到的资源（可选）。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcnicibP8xmLu606If4jhemgkGB9cJjib5wlL2dviaZlyEEwjukoXibr89S0A/640?wx_fmt=jpeg&from=appmsg&watermark=1)
如果你在你的 Agent（如 Claude Code）执行目录（如你的项目代码目录）下放了这个文件夹，

那下次和 Agent 对话的时候就能自动根据你的需求匹配到这个 Skill，不需要再进行任何额外的配置。

比如，你希望 Agent 帮你润色文章，就可以编写一个下面这样的 Skill：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgc3nib6OfqFTGiaHodsIiaicZiaeOiaBclicFlQkvb1balOpibXUmibr5sibfBTXDQ/640?wx_fmt=png&from=appmsg&watermark=1)
上面的三根短横线部分相当于 Skill 的「身份证」：

- name 是它的唯一标识，起个简单好记的英文名字就行
- description 则决定什么时候会触发这个 Skill，描述这个 Skills 是做什么的、遇到什么样的用户请求应该用它、提醒读者：描述越具体，越容易在正确场景被调用
下面就是 Skill 的正文部分：

- 目标：简单描述清楚这个 Skill 要做的事情
- 使用步骤：列出 Skill 的操作流程（先搞清楚想要什么风格、再读原文、再改写、最后规定输出格式）
- 注意事项：告诉模型「什么不要做」（不要乱加内容、不要替用户做决定、有歧义要提醒）
看起来挺普通的？似乎很多能力都可以做这件事？

- 可以把这段文字和要润色的文章直接发给大模型？
- 可以把这段文字放到系统提示词？
- 可以把这段固定的流程封装为一个 Workflow？
- 可以把这段文字编写为一个 Agent.md 或者项目级的 Rules？
这些方式看似不同，但本质上只是把提示词放在了不同的位置，你给 AI 的每次对话都会带上这些提示词。

在真实的业务场景中，一个 Agent 不可能只干一件这么简单的事。大家试想一下，如果你要给 AI 装 50 个技能，每个技能都有几千字的说明书，要是系统一启动就把这些全塞进 AI 的脑子（Context Window）里，那么就会：

- 成本爆炸，每次对话可能都会消耗几万 Token。
- AI 的注意力也会被分散，变得“这也想干，那也想干”。
Skill 的出现就是为了解决这种问题，它有一个非常核心的机制，叫渐进式披露（Progressive Disclosure）。说人话就是：按需加载，用多少拿多少。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcnm1Zia3kvED4JgpED84UTY8aWa02qFKuVoyd0OzCmH8J98icQ4y5ttQA/640?wx_fmt=jpeg&from=appmsg&watermark=1)

### 1.3 Skills 的核心机制
这是我觉得 Agent Skills 设计得最聪明的地方。你可以把它想象成我们在图书馆查资料的三个步骤，非常直观：

第一层：先看目录（元数据 Metadata）![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgccoL4D9mMulNKwXgaf83t0PB0umlmfYcVv3ibZqxRIGiap9MuxvUBaBUQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)

- 什么时候加载？ 系统刚启动的时候。
- 加载什么？ 只加载每个技能的名字和一段简短的描述。
- 有什么用？ 这一层占用的资源极少，可能就几百个 Token。它的作用就是告诉 Claude：“嘿，你的工具箱里有‘查周报’、‘处理 Excel’ 这几个工具哦。”
- 结果： Claude 知道自己 “会什么”，但还不知道 “具体怎么做”。
第二层：翻开手册（指令 Instructions）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgciavJWQTtLYlh7MDKia7jAnRpFcGM8LIXb3QHukShXyKDGYFemKohBWHg/640?wx_fmt=png&from=appmsg&watermark=1)

- 什么时候加载？ 当你说 “帮我把这个 Excel 处理一下” 的时候。
- 加载什么？ Claude 发现这事儿归 “Excel 处理” 这个技能管，于是它才会通过后台命令，去读取那个文件夹里的 SKILL.md 文件。
- 有什么用？ 只有在这个时候，那些详细的操作步骤、注意事项才会进入 AI 的脑子。
第三层：动手干活（运行时资源 Runtime Resources）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcVxlk68VHIeSlnNLzQiboMJj7tsvOP4h5T5iau7KBqILEGP6yrTLxaUvg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

- 什么时候加载？ 真正执行具体步骤的时候。加载什么？
- 参考（reference）： 用户下达的任务可能是分析 Excel，也可能是创建 Excel，这两个操作可能有完全不同的处理步骤，详细的步骤不一定都在 SKILL.md 中，可以分开放在不同的参考文献（reference）下，当 Claude 识别到你要做的是分析 Excel 时，才会去查阅分析 Excel 的 reference。
- 脚本（scripts）：Skill 中可以内置一些可执行的 Excel 处理脚本，在 SKILL.md 或者具体的参考文献（reference）下会告诉你应该调用以及如何调用这些脚本。还有最重要的一点，Claude 只需要按照指引执行脚本，而脚本本身的代码是不会塞给 AI 去读的，你完全不用担心一个超大代码文件会消耗 Token。

> 这意味着：一个 Skill 可以打包整套说明文档、大量的执行脚本，但只要任务不需要，这些内容就永远不会占用上下文。

## 二、Skills VS MCP
看到这，你可能会觉得 Skills 和 MCP 有点像？

它们似乎都可以做到按需加载、给 AI 扩展外部能力？

这也是很多同学可能会弄混的问题。

### 2.1 MCP 有什么问题
在 [全网最细，一文带你弄懂 MCP 的核心原理！](https://mp.weixin.qq.com/s?__biz=Mzk0MDMwMzQyOA==&mid=2247503721&idx=1&sn=19e1b701cb8ba16a10a1d8f58e68711b&token=1267907119&lang=zh_CN&scene=21#wechat_redirect) 中，我们介绍了 MCP 出现的意义和执行原理：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcDaLgScZ9K0g61nBv0u4YAQ2pCWGiabD4MwPX3kJOTJ3ca23MEpkE3LQ/640?wx_fmt=png&from=appmsg&watermark=1)

> MCP（Model Context Protocol，模型上下文协议）是由 Anthropic 公司推出的一个开放标准协议，它就像是一个 “通用插头” 或者 “USB 接口”，制定了统一的规范，不管是连接数据库、第三方 API，还是本地文件等各种外部资源，都可以通过这个 “通用接口” 来完成，让 AI 模型与外部工具或数据源之间的交互更加标准化、可复用。
所以 MCP 的本质，还是在做 “标准化”，它让给 AI 扩展外部能力这件事更 “标准化”。

假如你的 Agent 连接了多个 MCP，它似乎也能实现 “按需加载”（根据用户的意图决定调用哪个工具）。

但这个 “按需加载” 背后的代价是非常巨大的，在 MCP 的架构下，仅仅是“连接”这个动作，就已经在透支你的额度了。

这是由 LLM 的工具调用机制决定的。为了让 AI 知道它有哪些能力可用，每一个连接的 MCP Server 必须在对话开始前，将其所有工具的完整定义（名称、详细描述、参数 Schema、使用示例）一次性注入 LLM 的上下文中。

每个 MCP Server 一般都会包含大量的工具，比如 Github MCP ，它自己就包含了 30 多个工具：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcsMAYicxONgh6Cm2lHzJeDjKPLTkYibpvRRH5aRRjWRhOoVkwzibp8hFdQ/640?wx_fmt=png&from=appmsg&watermark=1)
假如每个工具消耗 500 个 Token，那只链接这一个工具就需要消耗将近 20000 Token。

在真实环境下，一个 Agent 不会仅链接一个 MCP Server。

假如你只问了 AI 一个非常简单的问题（1+1=？），Agent 已经烧掉了大几万的 Token，这个成本是非常恐怖的。

更深层的问题在于链接过多的 MCP Server 可能导致 LLM 的 “注意力” 下降，从而降低工具调用的准确性。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcmYH310y4E28oRz88Q3jBaLFvRJ8RIeibQQuwptm92ou8ac8Riawwkzkw/640?wx_fmt=png&from=appmsg&watermark=1)
我在之前的文章中有讲过一个专门测试 MCP Server 调用准确度的基准：MCP Atlas（[世界最顶级的大模型，都在 PK 些啥？ （大模型评估完全指南）](https://mp.weixin.qq.com/s?__biz=Mzk0MDMwMzQyOA==&mid=2247505276&idx=1&sn=d57b06af2f091ae00bc9ef1d723cb9d4&token=1267907119&lang=zh_CN&scene=21#wechat_redirect)），在这个基准中包含了 40 多个不同服务器、300 多个工具的复杂环境。

模型必须自己发现合适的工具、正确调用，并把多步结果汇总成最终答案。目前最强的 Claude Opus 4.5 也只能拿到 62% 的准确率，这个值还会随着工具的增多而进一步下降。

而我们上面讲到的 Skills 的核心机制：渐进式披露 ，恰好可以解决这两个问题：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcfkG30ibVd5jbKSVJaPhCewdxWeiasE8At0CJQ0ha9HIyzXJrwUZRoAfA/640?wx_fmt=jpeg&from=appmsg&watermark=1)

- 节省 Token：首次链接时，相比 MCP 需要将 40 多个 MCP Server 下 300 个工具全部塞进模型上下文（消耗数万 Token），模型只需要加载 40 个 Skills 的元数据（几千 Token）。
- 提升注意力： 面对几百个工具，AI 很容易分心。Skills 采用的是“漏斗式”引导：先通过目录判断大方向，确认要干活了，再加载具体的说明，最后通过说明找到详细的文档和脚本最后再执行。让 AI 每次只专注于当前任务。即使是能力稍弱的模型，在这种机制下也能保持极高的调用准确率。

### 2.2 MCP 会被淘汰吗？
看到这你可能会问了，Skills 看起来更智能、更节省资源，那 MCP 会被淘汰吗？

> 结论是：MCP 不会被完全淘汰，但对它的需求会大幅减少！

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcBUAwyJhGnogaurib2BC0Y07EKR2Jq0jO2YNmX0UTiaMPMFe0hRGDIbyA/640?wx_fmt=jpeg&from=appmsg&watermark=1)
首先，MCP 协议层的价值不可替代： MCP 的真正价值不在于它如何把文本塞进 Prompt，而在于它制定了一套标准接口。

它统一了 AI 连接世界的方式。如果你是一个通用的三方平台（高德地图、Notion 等），想发布一个工具让其他 Agent 都能用上你的能力，那首先选择的还是 MCP。

但是，如果你有一些重复性的工作流，比如要以固定的流程读写本地文件、要用一个标准的范式来 Review 代码、有一套固定的风格来编写文章，这些场景都推荐使用 Skill 来实现。

在过去这几个需求中的本地文件读写、链接 Github、给文章生成图片这些需要链接外部世界的能力都得通过 MCP 去实现，但现在你可以都把它们打包到 Skill 里。

未来的格局可能是这样的（来自宝玉老师）：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcTozcbbdmbUgwZ6k1icMaldcPY17icacED83oJFrraAg3fSJuhic6bATGg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

- Agent 本身内置部分核心能力（bash、read、edit、write）
- 少数通用 MCP Server 负责远程连接（数据库、云 API、SaaS 集成）
- 大量 Skills 负责封装标准工作流、连接本地知识库
- 两者在必要时协作，但 Skills 会承担绝大部分 “教 AI 怎么做事” 的工作（这其中也包含教 AI 怎么用 McpServer、怎么用其他 Skills、怎么更好的调用核心能力）

## 三、Skills 的初步尝试

### 3.1 去哪找 Skills？
和 MCP 一样，Skills 成了开放标准后开始爆发式增长，社区出现了大量的开源 Skills，很多 Skills 开放市场也应运而生，之前大部分 MCP Market 也都增加了 Skills 的分类：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcxBLAS2QkqzycYDjfuK3t6ghvONx4Q9B9T8ib0WfBiaQBywL7MXBb7KqQ/640?wx_fmt=png&from=appmsg&watermark=1)
https://skillsmp.com/

我们可以看到 skillsmp 中的 Skills 数量在最近经历着爆发式增长，这个增长速度要比之前的 MCP 爆火的时候还要快。这就不得不提 Skills 的另一大优点：编写门槛低！

MCP 虽然有一套标准的规范，但终究还是要靠代码编写的，即便有了 AI 辅助，对于小白来讲还是有一定的门槛的。

而 Skills 就不一样了，只要你会写提示词，就能写 Skill，可以预见的是，之前那些大量的固定工作流在未来可能都会被编写为 Skill，这也意味着 Agent 的编写门槛被再一次大幅降低了！

### 3.2 怎么使用 Skills？
我们随便进入一个 MCP 市场，然后搜索我们要使用的 Skills，比如这里我们还以绘图软件 Excalidraw 为例：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcnicicFmicDCLVicVU1aGPicBFC5SgO3LKphz4ibxd2gou5vKyaUgppCeqkWQ/640?wx_fmt=png&from=appmsg&watermark=1)
可以看到社区已经有大量 Excalidraw 的 Skill 了，我们这里选择 Star 最多的一款：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcmShzKQUbbIDecDggJP0ynhmxXHFkZ6YEn2ZTtkibXRYLdsJLCKic6Xjw/640?wx_fmt=png&from=appmsg&watermark=1)
进入详情后，我们选择一个最简单的安装方式，直接把这个 Skill 下载到本地，点击 wget skill.zip：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcaqZWqia0o5ED4cAS51paOibWMl0BnOwPKOhK5QDGHWQH9zNXvx3vbAJg/640?wx_fmt=png&from=appmsg&watermark=1)
然后我们把这个压缩包解压，你就会看到熟悉的目录。接下来，你只需要把这个目录下载到指定的位置：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcFtlwhKtNE3BwBHibog4PsXOCU1GOWJofBuGZbCC1nhGNDpdkMa14hSw/640?wx_fmt=jpeg&from=appmsg&watermark=1)
不同客户端的目录大同小异，基本上都是  .agentName/skills 目录，这里我们使用最近比较火的 OpenCode 进行演示（大家看可以自行选择 Cusor、Codex 等支持 skills 的客户端），所以我们创建一个新的文件夹，然后把刚刚下载的文件夹放到 .opencode/skills 目录下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgchgFtl8oDCxMbibVHxlGjtFHG5TAu3SL8nKhDu0rINnM9yeherW1WLuw/640?wx_fmt=png&from=appmsg&watermark=1)
接下来，我们在这个目录下打开 opencode 客户端，输入下面的提示词：

> 帮我绘制一个架构图，讲解什么是 5W2H 分析法，直接帮我在当前目录下生成一个 excalidraw 文件。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgc8XiccmwuIWnzrqibrn36zTHY9xlssJv5nu7IplMWiax68HYkRlFc9MC5Q/640?wx_fmt=png&from=appmsg&watermark=1)
你不需要手动去 “安装” 或 “运行” Skill，只要文件放对位置了，OpenCode 的 AI 就会自动根据用户的需求判断要调用这个 Skill，然后帮我生成了代码：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcpAOcwXyEwVWj8QlujJpIicFuMvZo8sSGGhialiaRAZbJ0x1qayiaeGd11w/640?wx_fmt=png&from=appmsg&watermark=1)
我们将生成的代码粘贴到 https://excalidraw.com/，就可以看到已经生成好的架构图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcIY4D5xWgmlYeicmvOQTXYBsrN8KRNkAMZMFfcEC7icTObuIW0Nn8lRLw/640?wx_fmt=png&from=appmsg&watermark=1)

### 3.3 创建你的第一个 Skill
下面，我们一起来尝试做第一个 Skill，虽然 Skill 的开发门槛低，但这不意味着我们就要自己写！

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcJ8aU8AicfaicQiac2GibHXoa5BnZq2LlV93lqKicfvs0gAx7LHAID14F37A/640?wx_fmt=png&from=appmsg&watermark=1)
Anthropic 官方直接给我们提供了一个 生产 Skills 的 Skill：Skill Creator。你不需要写一行代码或配置文件，只需要用自然语言告诉它你想做什么，它就会自动为你生成一个符合标准的 Skill 包。

接下来，按照刚才的流程，我们把这个 Skill 下载下来，放到 .opencode/skills 目录下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcwhTSHJ23n7C022K0Bc8eIBtRL5mEZD7hyuvVcRj8H4LGaYggWTCYiaw/640?wx_fmt=png&from=appmsg&watermark=1)
然后我们给出下面的提示词：

> 帮我创建一个可以准确获取当前系统时间的 Skill，描述使用中文，脚本使用 Node.js。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgceLNyIXxrq2ScXryQ2kWJBiasFJwb2eQKgLOvYx65Tia2h69MR9DCicvTg/640?wx_fmt=png&from=appmsg&watermark=1)
然后，opencode 识别到我们的需求，开始调用 skill-creator：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcd1FmkakDFqRuiaa4qmVwpKw8VkZKry484EqK5DhzSNYxiaOf0VU7vh2A/640?wx_fmt=png&from=appmsg&watermark=1)
然后我们打开本地的 .opencode/skills  目录，发现多了一个 current-time-node skill，包含一个 SKILL.md 加一个获取准确时间的 Node.js 脚本：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcWRPMUA0tkUFvhSBd5N8WMZA9CiawtLyZEVUD5oxf5xqpa0Enj4t7CNQ/640?wx_fmt=png&from=appmsg&watermark=1)
接下来，我们询问 opencode：“获取当前系统时间”，然后它就会自动找到刚刚生成的 Skill 并调用里面的脚本：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/e5Dzv8p9XdThn0BIDxEwF89DOYia2bzgcciaicFiae7UMMgfISWkODx0whibtXAAF1NPm0pAHUretUjJOials3IJEQFQ/640?wx_fmt=png&from=appmsg&watermark=1)

## 最后
本期教程我们就先讲到这。

大家已经了解了 Agent Skill 的基本原理，以及如何使用和创建一个 Skill。

如果本期教程对你有所帮助，希望得到一个免费的三连和关注。

下一期，我们会进入实战章节，一期来使用 Agent Skill 实现一个知识库检索的功能，相比传统的 RAG ，它的效果究竟怎么样呢，我们下期见。

关注《code秘密花园》从此学习 AI 不迷路，相关链接：

- AI 教程完整汇总：https://rncg5jvpme.feishu.cn/wiki/U9rYwRHQoil6vBkitY8cbh5tnL9
- 相关学习资源汇总在：https://github.com/ConardLi/easy-learn-ai
如果本期对你有所帮助，希望得到一个免费的三连，感谢大家支持！

---

> ⚠️ 以下图片未能从正文 HTML 中定位，按下载顺序追加：

![图片](./images/img_001.jpg)

![图片](./images/img_002.jpg)

![图片](./images/img_003.jpg)

![图片](./images/img_004.png)

![图片](./images/img_005.jpg)

![图片](./images/img_006.jpg)

![图片](./images/img_007.png)

![图片](./images/img_008.jpg)

![图片](./images/img_009.png)

![图片](./images/img_010.png)

![图片](./images/img_011.png)

![图片](./images/img_012.jpg)

![图片](./images/img_013.jpg)

![图片](./images/img_014.jpg)

![图片](./images/img_015.png)

![图片](./images/img_016.png)

![图片](./images/img_017.png)

![图片](./images/img_018.png)

![图片](./images/img_019.jpg)

![图片](./images/img_020.png)

![图片](./images/img_021.png)

![图片](./images/img_022.png)

![图片](./images/img_023.png)

![图片](./images/img_024.png)

![图片](./images/img_025.png)

![图片](./images/img_026.png)

![图片](./images/img_027.png)

![图片](./images/img_028.png)

![图片](./images/img_029.png)