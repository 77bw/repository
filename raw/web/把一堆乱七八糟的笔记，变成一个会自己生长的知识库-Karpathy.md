---
title: "把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy"
source: "https://linux.do/t/topic/1907342"
author:
  - "[[zanezhou]]"
published: 2026-04-07
created: 2026-04-26
description: "Karpathy 最近发了一篇文章，推上到处都在转发，推荐大家看一下 https://x.com/karpathy/status/2039805659525644595?s=20 大意是他在用 LLM 给自己建一个私人知识库，把你平常收藏的文章帖子统统扔进一个文件夹，然后让 AI"
tags:
  - "clippings"
---
Karpathy 最近发了一篇文章，推上到处都在转发，推荐大家看一下 [https://x.com/karpathy/status/2039805659525644595?s=20](https://x.com/karpathy/status/2039805659525644595?s=20)  
大意是他在用 LLM 给自己建一个私人知识库，把你平常收藏的文章帖子统统扔进一个文件夹，然后让 AI 把它们整理成一个结构化的 wiki

先给大家介绍下这一套是怎么运行的：  
佬们把平常收集到的帖子，文章直接爬下来直接扔到一个文件夹，然后 AI 负责整理，分类，建立联系。你只是一个采集者和提问者，AI 就像是图书管理员

这个知识库会随着时间不断复利增长，存得越多，整个知识库就越聪明

---

# 怎么搭建

## 1.建三个文件夹

建一个项目文件夹，里面放三个子文件夹：

```csharp
my-knowledge-base/
  raw/       ← 原始资料放这里
  wiki/      ← AI 整理后的 wiki 放这里
  outputs/   ← AI 回答问题的结果放这里
```

就这三个

raw/  
是你的原始素材堆。文章，帖子随手记下来的想法往里扔就行，不用整理

wiki/  
是 AI帮你整理之后的成果，里面每个话题一个

.md  
文件，互相有链接

outputs/  
是你问 AI 问题之后，它给你写的那些分析和报告

## 2.开始往raw/里扔东西

eg：  
你在微信里转发给自己的文章，复制进来变成一个.md文件就行

Obsidian有一个叫Web Clipper的浏览器插件，可以一键把网页转成markdown 存下来，非常好用，具体可以看我这篇文章 [Obsidian+Claude太强了：拯救你吃灰的收藏夹](https://linux.do/t/topic/1893419)

## 3.写一个 schema 文件

这一步很重要  
在你的根目录下，新建一个叫 CLAUDE.md 的文件（你用别的 AI 工具，叫AGENTS.md 或者 README.md之类的豆行）。这个文件是给 AI 看的说明书，告诉它你的知识库是关于什么的、wiki应该有哪些规则要遵守

给大家看看一个基础模板长啥样：

```diff
# 项目定义
[写你的主题，比如：AI产品设计、自媒体、行业研究]

# 组织结构
-raw/ 包含未处理的原始素材。切勿修改此类文件。
-wiki/ 包含整理后的维基百科。完全由 AI 进行维护。
-outputs/ 包含生成的报告、回答和分析结果。

# wiki 的规则
-每个主题在 wiki/ 目录下拥有独立的 .md 文件。
-每个维基文件开头须包含一段摘要。
-使用 [[主题名称]] 格式链接相关主题。
-在 wiki/ 目录下维护一个 INDEX.md 索引文件，列出所有主题及其单行简介。
-当有新的原始素材（raw sources）加入时，须同步更新相关的维基文章。
# 我关注的方向
[列出 3-5 个你希望该知识库重点关注的领域]
```

## 4.让 AI 编辑你的 wiki

打开cc让他读你的 raw/ 文件夹，然后按照上面规则，在 wiki/里建一套完整的 wiki

你可以这样说：  
`先阅读 raw/ 中的所有原始内容，然后按照 CLAUDE.md 中的规则在 wiki/ 目录下编译一个 wiki。首先创建 INDEX.md，然后为每个主要主题创建一个 .md 文件。链接相关主题，并对每个来源进行总结。`

然后整杯咖啡等着就行了  
有一点比较重要，不要手动编辑wiki中的内容，别碰，那是你cc小秘书的工作

## 5.开始提问

wiki 有10篇以上的文章，就可以开始提问  
比如:“用知识库里的内容，给我写一篇关于 某个话题 的500字简报”

把好的回答存到 outputs/ 或者让 AI 直接更新对应的wiki页面。你的每一次提问与回答，都将完善的的知识库

## 6.定期检查

没隔一段时间比如一周让AI 对wiki做一次检查：  
`检查整个 wiki/ 目录。找出页面之间有没有矛盾的说法；找出被提到但没有独立页面的话题；找出有哪些说法在 raw/ 里找不到来源；建议 3 个新文章方向来填补空白。`

如果有一个错误信息存进去，会一直传递，定期检查可以避免

## 关于是否用 Obsidian

不需要，但我还是想用~  
Karpathy 原话说他想让这套系统”尽量简单和扁平”  
也确实是，一堆md文件加上一个schema，比小红书那一群七八十个插件配置上坑小白的自媒体好多了

## 知识库大了之后的检索问题

一两百篇文章后 光靠INDEX.md来定位相关页面会开始吃力  
这里我还没发现很好的方法，希望各位佬给点思路~

没想到这么快就有人做出来！

[github.com](https://github.com/safishamsi/graphify)

![](https://cdn3.ldstatic.com/optimized/4X/3/5/e/35eef3abe4a7e5b49d7ba21293b4b70559dd0b86_2_690x344.png)

### [GitHub - safishamsi/graphify: AI coding assistant skill (Claude Code, Codex,...](https://github.com/safishamsi/graphify)

AI coding assistant skill (Claude Code, Codex, OpenCode, OpenClaw). Turn any folder of code, docs, papers, or images into a queryable knowledge graph

这个厉害了，它能把代码、文档、PDF、截图这些东西一起读，整理成一张关系网。可以让AI更快地读懂项目结构，还有你的项目为什么这么设计，同时能更省上下文。

## 附加价值

整个知识库就是一堆.md文件。扔进git仓库 你就有了完整的历史记录，可以对比改动、甚至还可以协作

## 最后的话

这一套就是我一直在找的知识库解法。在AI时代之前知识库的维护成本太高，现在开始逐渐有了解法

Vannevar Bush在1945年提出过一个叫 Memex 的想法：一个私人的主动策划的知识存储，文档之间有关联路径，比简单归档更接近人类联想的方式  
他当时没有解决的问题是：谁来做维护？现在这个问题有答案了

三个文件夹，一个说明文件，一个 AI。

就这些。