---
title: "7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包，普通人也能写出大神级代码"
author: "结构派AI"
account: "结构派 AI"
date: "2026年4月9日 20:00"
digest: "几天前Andrej Karpathy 吐槽:\x26quot;模型会自作主张做出错误假设，然后一路错下去。"
source: "https://mp.weixin.qq.com/s/RB5h5BQ-x3c78JgbjxKBtA"
---

# 7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包，普通人也能写出大神级代码

**作者**：结构派AI  
**公众号**：结构派 AI  
**发布时间**：2026年4月9日 20:00  
**原文链接**：[7.9k Star 一夜爆火！把 Andrej Karpathy 做成了 Claude 技能包，普通人也能写出大神级代码](https://mp.weixin.qq.com/s/RB5h5BQ-x3c78JgbjxKBtA)

---
几天前Andrej Karpathy 吐槽:

> "模型会自作主张做出错误假设，然后一路错下去。它们不会管理困惑，不会寻求澄清，该反驳的时候不反驳，100行能解决的问题，要写1000行。"

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/csdHwSTog2xOtTpnPDFicg0V0noh3IKribg0dIpquAvjgMAdh1ibJV32SzzYKTDJHWZWeD4iamOkCKrYkpuzu3fSJacBgxkLFAhAuKia8GVHb8Wk/640?wx_fmt=png&from=appmsg&watermark=1)

于是，有人就是基于Karpathy的观察，变成skills项目，做成Claude Code 可执行的行为准则。

为什么这个项目突然爆火
用Claude Code写代码的同学，多半遇到过这些糟心事：

你让改一个导出函数，它顺便把整个配置文件重新格式化了。

你说清楚"只加一个参数"，它给你抽出三层抽象，把简单问题复杂化。

需求有歧义，它不问，直接选一个错的理解往下跑，最后你还要返工。

仅仅一个 `CLAUDE.md` 文件，上线几天狂揽 7.9k Star，目前还在涨📈。

![图片](https://mmbiz.qpic.cn/mmbiz_png/csdHwSTog2x5WFfR33pibsJWVapPTArBOt3RI7YdYaNlZgPnvb0VckGx8XxWvZDBLOws4L0icGdKJdZWGTIdLY3XXAy0AgrYhnHbBxKjadjdA/640?wx_fmt=png&from=appmsg&watermark=1)
很多人说，放了这个文件之后，Claude写代码"正常多了"。

四个原则，每个都帮你省时间
项目核心就是四个原则，精准命中AI写代码的四个常见病，每个都直接对应你的时间账单：

| 1. Think Before Coding — 不猜，不装懂帮你省：猜错需求返工的半小时• 不确定就问，不要默默选一个解释运行• 有歧义就把几种解释都列出来• 发现更简单的方案就直接说出来• 哪里不懂就直接指出来，停下来澄清 |
| --- |

| 2. Simplicity First — 能少写，就不多写帮你省：读懂和维护垃圾代码的半小时• 只做需求内的功能，不提前加"灵活性"• 单次调用不做抽象，不搞过度工程• 不可能发生的场景，不做错误处理• 200行能写完，绝不写201行 |
| --- |

| 3. Surgical Changes — 只改你该改的帮你省：还原无辜代码的十几分钟• 不动相邻的代码、注释、格式• 没坏的地方不重构• 原有风格是什么就是什么，哪怕你不喜欢• 发现无关死代码，只提醒不删除 |
| --- |

| 4. Goal-Driven Execution — 先想清楚什么是"做好了"帮你省：反复沟通"这样对不对"的十分钟• 把模糊需求转成可验证的目标• 修bug先写测试复现，再改• 多步任务要写出每一步验收标准• 目标越具体，AI越少瞎折腾 |
| --- |

实测对比：变化真的很明显
我把这个文件放到了当前项目里测试，前后变化超出预期：

**之前的Claude：** 

- 我让改一个导出路径，它顺便把整个配置文件的缩进重新格式化了 

- 说好了只加一个判断，它给我抽出一个基类，再搞两个实现类 

- 对需求有疑问，它不说，直接按照错的来，最后我要回滚改半天

**加上文件后的Claude：** 

- 它会先问我："我理解的需求是XXX，如果理解错了请纠正我" 

- 真的只改了我说的那几行，无关代码碰都不碰 

- 发现我要的方案太复杂，它会主动说："这里可以更简单，你看行不行"

**从"热情过度帮倒忙的实习生"变成"手稳不越界的资深工程师"**。

这个改变，每天帮你省出至少1小时——你不用改AI犯下的低级错误，可以专注在真正要解决的问题上。

结合你的日常工作流，效果会更好
我自己日常用AI写代码的流程，其实和这四个原则完美契合：

我一般是这么干的：

 1. 我先想清楚需求，把要解决的问题写在注释里，明确验收标准

 2. 让AI先出方案，我来拍板选哪个，有歧义直接问清楚 

3. 改bug先让AI写测试复现，确认问题再动手改 

4. 写完让AI自己检查有没有多改东西，我只review功能

加上这个 `CLAUDE.md` 之后，相当于把我每次要说的话，提前变成项目规则，AI不用我每次提醒，自然就少犯错。

说白了，这就是**把你和AI的分工白纸黑字写下来**： 

- 你负责：定需求、拍方案、把边界划清楚 

- AI负责：按照规则干活，不越界、不多嘴

之前很多人用AI，反过来了：AI帮你定方案，你帮AI擦屁股。现在掰过来，效率立刻上去。

为什么这招有用？背后是三个反常识结论
这个项目之所以爆火，不是因为它发明了什么新东西，是它戳破了三个行业迷信：

1. 不是模型越大越好，是行为约束越好
现在大家都在追更大的模型，参数从7B涨到70B再涨到405B。

但实际上，GitHub 上这个项目的 issue 区里，排在前面的反馈几乎一模一样——"加了这个文件后，Claude 不再自作主张了""同样的模型，感觉聪明了一倍"。

有人测了一下，加上规则前后，相同任务的 code review 一次通过率从大约 40% 提升到了 80% 以上。

这说明什么？大部分时候不是 AI 能力不够，而是行为不对。

它不是写不出来，是太"热情"了，喜欢过度发挥，喜欢帮你做决定。

给它一套清晰的边界规则，效果比换个更贵的大模型好得多，成本是零。

2. 不需要提示工程，需要"提示落地"
很多人还在玩"顶级提示词"，找各种1000字的万能提示词。

这个项目给了另一个思路：**把沉淀下来的经验，变成项目级的固定规则**。

区别在哪，放进 `CLAUDE.md`，它就变成了代码仓库的一部分。可以 git 版本管理，可以团队共享，新人加入项目自动继承，不用每个人重新摸索一遍"怎么跟 AI 说话"。你在聊天框里写的提示词，关了窗口就没了；写在项目文件里的规则，只要项目在就一直生效。

这才是可复用、可传承、可迭代的提示工程。

3. AI coding的终极方向：人和AI分工明确

人提需求，定边界；AI干活，不越界。

人负责拍板"做不做"、"做成什么样"；AI负责"怎么做"，不多做也不少做。

这其实是软件工程几十年的老道理——最好的工程师不是写代码最多的那个，而是最克制的那个。只不过以前这个道理是用来管人的，现在我们发现用来管 AI 一样好使。

Karpathy 的原话本质上就是在说这件事：模型的问题不在于"不够聪明"，而在于"不够克制"。

这个项目用一页纸把克制变成了可执行的规则。

不止Claude，所有AI coding工具都能用
这个方法不绑定Claude Code，你用Cursor、GPT-4o、Gemini Code Assist，一样能用。

做法一样简单：在项目根目录放这个文件，AI一般都会自动读取。就算不自动读，你每次提问的时候加一句"参考项目根目录的CLAUDE.md"就行。

就算你不用AI写代码，把这四个原则放在你自己的开发流程里，也能少写很多垃圾代码：

- 想清楚再动手 → 少返工
- 能简单就不复杂 → 好维护
- 别碰不该碰的 → 少出bug
- 先想验收标准 → 少走歪路
这其实就是资深工程师的基本素养，现在我们把它教给AI。

一分钟用上
使用方法极其简单，不需要装任何依赖，不需要改任何配置：

1. 打开你的项目根目录
2. 新建一个文件 `CLAUDE.md`
3. 把下面这段内容粘进去保存
4. 重启 AI 编辑器，搞定

```
# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes.

## 1. Think Before Coding
-State your assumptions explicitly. If uncertain, ask.
-If multiple interpretations exist, present them.
-If a simpler approach exists, push back.
-If unclear, stop and ask clarification.

## 2. Simplicity First
-No features beyond what was asked.
-No abstractions for single-use code.
-No unrequested "flexibility" or "configurability".
-If 200 lines can be 50, rewrite it.

## 3. Surgical Changes
-Don't "improve" adjacent code, comments, or formatting.
-Don't refactor things that aren't broken.
-Match existing project style.
-Mention dead code but don't delete it unless asked.

## 4. Goal-Driven Execution
-Transform tasks into verifiable success criteria.
-State a brief plan for multi-step tasks with checks.
```
项目地址：forrestchang/andrej-karpathy-skills

最后说一句
这个项目最有意思的地方是：

它**没有写任何一行功能代码**，只是把聪明人已经验证的开发原则，整理出来交给AI。

但就是这一页纸，解决了无数人每天都在遇到的痛点。

现在大家都在卷大模型，都在等下一个"更强"的模型。

但这个项目告诉我们：

**把现有的模型用好，把边界定清楚，比等一个更大的模型，回报快得多。**

反正只需要一分钟，不妨试试——看看你的AI写代码，会不会"正常"很多。

你用AI写代码遇到过哪些哭笑不得的问题？欢迎留言交流。

关注我们，获取最新AI资讯和深度解读 🤖

**![图片](https://mmbiz.qpic.cn/mmbiz_gif/SGWPQ5J9CreIqlzzBGookRWXYbI8yyyZvicXzuRJTNTO0qgiaj3AmujJDOjhPvX2SnF61oy9Z2VtZCWjzic1UmBwQ/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp)**

---

> ⚠️ 以下图片未能从正文 HTML 中定位，按下载顺序追加：

![图片](./images/img_001.png)

![图片](./images/img_002.png)

![图片](./images/img_003.gif)