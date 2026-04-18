---
title: "中英混写Prompt更有创造力？我设计了个盲评实验来验证"
author: "问答"
account: "AI 方寸山"
date: "2026年4月13日 07:57"
digest: "中英文混着写Prompt真的更有创造力吗？我设计了两轮盲评实验，纯中文、纯英文、中英混合三组对决，三个AI独立评分。结果出人意料：两轮实验排名完全相反，任务类型才是决定性变量。"
source: "https://mp.weixin.qq.com/s/Ur2HVaf2O8a1HL5ZLHjfEA"
---

# 中英混写Prompt更有创造力？我设计了个盲评实验来验证

今天看到个说法：说你写Prompt的时候，中英文混着来，比如「帮我brainstorm一个极具cyberpunk风格的marketing方案」这种，AI给出来的东西会更有创造力。  

原因嘛，据说是中英混合会迫使模型在两个不同的语义空间之间跳跃，在碰撞中产生火花。

听起来特别有道理对吧。

说真的，我身边很多做AI的朋友，包括我自己，写Prompt的时候也确实习惯性地中英混着来。但要说为什么这么写，好像也从来没有真正想清楚过。就是一种直觉，一种「感觉这样好像厉害一点」的模糊信念。

然后我就想，这玩意到底是真的还是玄学？

于是，手痒病又犯了，我设计了一个实验来验证下。

准确的说，两轮实验。

思路很简单，写三组Prompt，内容完全一样，唯一的区别就是语言形式，一组纯中文，一组纯英文，一组中英混合。然后让Claude Sonnet 4.6跑，跑出来的结果匿名打乱，交给三个完全不同厂商的AI模型盲评打分。评分的时候，它们完全不知道哪份输出对应哪种语言。

变量唯一，盲评打分，多模型交叉验证。

## Prompt 示例
给大家看一眼三组Prompt长什么样，以第二轮的任务为例，让AI发明10个「现代社会中存在但还没有词来描述的感觉」。

纯中文版：请发明10个「现代社会中确实存在、但还没有词来描述」的感觉或现象。每个发明一个自造的名字+一句话描述。越让人觉得「对对对我就是这样但从来没说出口」越好。不要鸡汤，不要励志，越意外越好。

纯英文版：Invent 10 feelings or phenomena that genuinely exist in modern life but don't have a word for them yet. Each gets a made-up name + a one-sentence description. The more "yes yes yes that's exactly it but I never had a word" the better. No chicken soup, no inspirational stuff. The more unexpected the better.

中英混合版：请invent 10个「现代社会中确实存在、但还没有word来describe」的feeling或phenomenon。每个invent一个自造的name+一句话description。越让人觉得「对对对that's exactly it但从来没说出口」越好。不要chicken soup，不要inspirational，越意外越好。

## 试验
第一轮：品牌营销方案，纯英文组均分53.4，中英混合组48.1，纯中文组45.8。纯英文碾压。

第二轮：发明未命名现代感觉，中英混合组30.8，纯中文组29.3，纯英文组26.7。混合语言翻盘。

稳定性：中英混合组标准差0.7（极差2.0），纯英文组标准差3.2（极差8.7），纯中文组标准差2.2（极差6.0）。

## 为何？
结构化任务：英文训练数据多，纯英文Prompt如铺好的高速公路。混合语言相当于放减速带。

发散任务：混合语言强制跨语义空间跳跃。中文情感词汇（缘分、怅然、乡愁）+英文现代概念（FOMO、liminal、doomscroll）同时激活两个联想网络，边缘区域错位产生创意火花。

正则化效果：混合语言迫使模型在两种语言结构间协商，减少单语言路径的「舒适区重复」。

## 意义和思考
结论：任务越开放发散，混合语言越可能有用；任务越结构化精确，越应用最流畅的单一语言。

对应语言学「代码转换」（code-switching）概念：双语者切换语言时大脑前额叶活跃度提高，认知灵活性更强。

局限：样本量小（3-5个），仅测Claude Sonnet 4.6，AI评分可能存在偏差，需要更多验证。