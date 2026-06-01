---
title: 提示词导师 Agent 完整指南
type: topic
tags: [prompt-engineering, ai-coding]
sources:
  - raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md
  - raw/ai-chat/vibe-coding-how-to.md
  - raw/ai-chat/claude-code-agent-team-guide.md
created: 2026-05-20
updated: 2026-06-01
summary: 双 CLI 工作流的提示词导师方法论：澄清→调研→拆解→交付包，搭配 skill / subagent / MCP 调度策略和可复用模板，专为「另一个干净上下文的 AI 会话」准备完整指令包。
confidence: medium
---

# 提示词导师 Agent 完整指南

## 概述

你有一个稳定的工作模式：**双 Claude Code CLI 终端**——一个生成提示词（导师），一个执行任务（执行者）。本文把这个模式正式化成一套方法论：导师 Agent 不直接写代码，而是产出**可直接交给另一个干净上下文 AI 会话执行的完整指令包**。

为什么需要导师？因为执行端 AI 拿到的是冷启动会话，看不到你的项目历史、参考代码位置、隐含约束。导师端的工作就是把这些"隐知识"显式化，并提前安排好 [[topics/agent-team-architecture|子代理]]、skill、MCP 的调度方案。

参考真实案例 `raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md`：原始提示词只列了 Goal/Context/Constraint 三段，但缺少澄清阶段、调研策略、子代理分工、验收标准。下文展示如何把这种"裸提示词"升级为完整交付包。

## 分析

### 一、提示词导师的四阶段流程

借鉴 [[topics/vibe-coding]] 的"需求→设计→任务→实现"四步，但导师 Agent 的职责不同——**不实现代码，只产出 prompt**。流程改写为：

| 阶段 | 输入 | 输出 | 该用什么工具 |
|------|------|------|--------------|
| **1. 澄清** | 用户一句话目标 + 模糊上下文 | 澄清后的需求清单 | `brainstorming` skill 启动；让 AI 主动反问 |
| **2. 调研** | 项目根目录 + 参考代码路径 | 现状摘要 + 接口/数据结构 | `Explore` 子代理读参考代码；`context7` 查库文档；`claude-mem mem-search` 找历史经验 |
| **3. 拆解** | 澄清需求 + 调研结果 | 任务分解 + 子代理分工 + 验收标准 | `writing-plans` skill；规划主 Agent + N 个子 Agent |
| **4. 交付** | 完整方案 | 给执行端的 prompt 包（含上下文、步骤、约束、验证） | 输出一份可直接复制粘贴的 markdown |

每一阶段都让导师 Agent 在主动**提问**——这是 Vibe Coding 最核心的技巧：**让 AI 反问你，不要让它猜你的意图**。

### 二、导师端启动语（最关键的一段）

把这段直接粘贴到生成提示词的那个 CLI 里。它会让 Claude 进入"导师模式"：

```
你现在是一个「提示词导师 Agent」，职责是帮我产出一份可直接交给另一个
干净上下文 AI 会话执行的完整指令包。你不要写任何代码，只产出 prompt。

工作流程（严格按顺序执行）：

【阶段 1：澄清】
我会先给你一个粗略目标。你必须使用提问的方式帮我把以下内容确定清楚，
不要猜测我的意图，任何不明确的地方都必须向我提问：
- 业务目标的边界（输入是什么、输出是什么、成功标准是什么）
- 必须遵循的现有代码模式或参考实现的具体路径
- 隐含约束（多环境、多语言、性能、错误处理等）
- 哪些技术决策已锁定、哪些可由执行端自由选择
你一次最多问 3 个问题，问完等我回答再继续。

【阶段 2：调研】
澄清完成后，调用 Explore 子代理（subagent_type: Explore）并行读取
我提到的所有参考文件，输出：
- 关键函数 / 类的签名和职责
- 数据结构（dict 字段、Kafka schema、API 入参出参）
- 我没提到但执行端会用到的隐含依赖
对外部库或框架，使用 context7 MCP 拉最新文档。
对历史经验，使用 claude-mem mem-search 检索过往做法。

【阶段 3：拆解】
基于调研结果，调用 writing-plans skill 产出实施方案，包含：
- 模块拆分（每个模块独立可测）
- 子代理分工（哪些任务该并行、哪些必须串行）
- 验收标准（什么样算完成，怎么自动验证）
- 风险点和兜底（哪里容易出错、出错时怎么处理）

【阶段 4：交付】
把上述全部内容整合成一份 markdown 提示词包，结构如下：

# 任务名称

## 角色
[执行端 Agent 的身份与权限边界]

## 目标
[一句话目标 + 验收标准]

## 上下文
[项目结构、参考代码精确路径、关键 API/数据结构摘要]

## 工作流程
[step-by-step，每步都说明用什么 skill/subagent/工具]

## 约束
[必须遵守、绝对禁止]

## 自检清单
[执行端完成后必须自查的条目]

## 第一步动作
[告诉执行端先干什么，比如先用 systematic-debugging 摸清现状]

---
我的初始目标如下：
[在这里粘贴你的原始 Goal/Context/Constraint]
```

### 三、实战案例与调度速查

Shopee 爬虫案例的完整交付包演示（澄清问题示例 + 最终交付包样例）、Skill / Subagent / MCP 调度速查表、双 CLI 协作实操要点、以及如何将方法论固化为 skill。

详见 [[topics/prompt-mentor-case-study]]。

## 结论

把"双 CLI 提示词工作流"从手工模式升级成方法论的核心是三件事：

1. **导师与执行严格分离**——干净上下文是 AI 不犯错的最大杠杆
2. **澄清前置、调研次之、拆解第三、交付最后**——任何一步省略都会让执行端补救
3. **显式调度 skill / subagent / MCP**——不要让执行端 Agent 自己琢磨该用什么工具

对照原始的 shopee 案例：原版只有 Goal/Context/Constraint 三段裸信息，升级版多出"角色 / 验收标准 / 工作流程 / 自检清单 / 第一步动作"五个关键章节，并显式调用 Explore 子代理 + writing-plans / systematic-debugging 两个 skill。这是裸提示词与工业级提示词包的差距。

## 来源

- raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md
- raw/ai-chat/vibe-coding-how-to.md
- raw/ai-chat/claude-code-agent-team-guide.md
