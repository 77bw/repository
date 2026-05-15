---
title: Agent Team 架构与实战指南
type: topic
tags: [ai, ai-coding, prompt-engineering]
sources:
  - raw/ai-chat/claude-code-agent-team-guide.md
  - raw/ai-chat/mavis-multi-agent-architecture.md
created: 2026-05-16
updated: 2026-05-16
summary: Claude Code teamcreate 七大实战场景提示词 + Mavis Worker-Verifier-Engine 架构哲学，多 Agent 协作从"操作技巧"到"系统设计"的完整视角。
---

# Agent Team 架构与实战指南

## 概述

多 Agent 协作正在成为 AI 实验室的隐秘共识——OpenAI、Anthropic、Google、MiniMax 都在朝同一个方向走：让一个 AI 带一堆 AI 干活，回答**长程任务如何靠谱交付**的问题。

本文整合两个互补视角：

1. **操作层**：Claude Code 的 `teamcreate` 命令，七大经典场景的提示词模板。
2. **架构层**：MiniMax Mavis 提出的 Worker-Verifier-Engine 三角，论证"多 Agent 系统是 runtime，不是 prompt 编排"。

两者拼起来给出一个完整答案：怎么写 prompt 召唤 Agent Team（操作），以及为什么 prompt 编排不够、需要程序化调度（架构）。

---

## 一、为什么需要 Agent Team：单 Agent 干长活的四个症状

理解多 Agent 架构的起点，是先看单 AI 干长任务时是怎么坏掉的。

### 症状 1：上下文焦虑

任何稍微长一点的任务（如写一篇文章的 brief→调研→大纲→初稿→审校→改稿→配图→排版十几步），交给单 Agent 接着上一步往下走，大概率出现两种坏死：

- 把十几步压成两三步草草交付；
- 每做一步就停下问"要不要继续"，用户一晚上都在打"继续"、"继续"。

根本原因：AI 对"任务什么时候算完成"的判断是模糊的，所以宁可啰嗦也不冒险。姚顺宇的话点得很准：**"用短的 context 去训练，但让它能做长的 context 的事"**——关键不是上下文窗口多大，而是 AI 会不会自己管理上下文。

### 症状 2：长任务漂移

让一个 agent 写一整章技术解读，开头是技术分析语气，第三节不知不觉变成营销文案口吻；让它列参考资料，会把自己之前搜过的二手缓存当一手来源贴上去。

这时候追问它，它会诚恳地回头自检——但**它检查的对象，就是它自己刚刚漂移生成的现场**。一个被自己污染过的记忆里，做不出真正的纠偏。

这一条直接推出了"写的不审、审的不写"的硬规则：审校必须是另一个 Agent。

### 症状 3：长任务期间无法快速响应

IM 场景下用户期待秒回，但任务天然需要几分钟甚至更久。AI 要么给浅答案应付，要么让用户盯着对话框等十几分钟。

Mavis 的解法：把"秒回用户"和"执行任务"拆开——主 AI 收到消息先快速应一声"收到，5 件事我去拆，完成后回来找你"，再把任务派到后台并行跑，关键节点主动汇报。

### 症状 4：角色分工没有真正发生

几十个 skill 看似分工挺细（选题、调研、初稿、审校、配图），但**全部跑在同一个 Claude 里、用同一套记忆、看同一组文件**——本质上还是一个 AI 在轮班，每"换"一次角色，前面那个角色的影子都还在。

> "角色扮演不等于角色分工。"

真正的分工得让每个 AI 从一开始就只做一件事，连工具集都不一样——会计用 Excel，设计师用 Figma，能力边界清晰，长期跑下来才有复利。

---

## 二、Mavis 的架构答案：Worker-Verifier-Engine 三角

MiniMax 在 Mavis 技术报告里给出的核心判断：**"多 Agent 系统是 runtime，不是 prompt 编排"**。让多个 AI 一起干活，关键不是写更好的指令，而是搭一个能长期运行、能管它们的底座。

### 整体架构：Owner / Worker / Verifier

- **Owner**：拆任务，把大目标切成可执行的批次；
- **Worker**：干活，目标是把任务赶紧干完；
- **Verifier**：挑刺，目标是把活儿挑回去重做；
- **Team Engine**（关键的非 AI 组件）：确定性代码，在 Worker 和 Verifier 之间中转、调度、限制重试上限。

### 较真的两件事

**第一件：让两个 AI 心思相反**

Worker 和 Verifier 都以"结束"为目标，但**一方结束会触发另一方启动**。Worker 觉得自己干完了，Verifier 立刻开始挑刺；Verifier 挑出问题，Worker 被自动叫回来修；修完 Verifier 再检查，过了才算真的完成。

> "很多框架里的验证环节是可选的附加步骤，在我们这里它是架构的核心。"

**第二件：调度靠程序，不靠 AI 拍板**

两个 AI 之间不直接说话，全程靠 Team Engine 中转。这意味着系统的可靠性**不依赖某个 AI 那一刻清醒不清醒，而是写死在程序里**。

被低估的几个细节：
- 任务被切成一批一批跑，同一批内并行，下一批要不要启动看上一批是否全部通过验证；
- Worker 重做时从上次失败的状态继续，不用从头来；
- 有重试上限，陷入死循环时自动升级到人类决策。

### 与友商方案的对比

| 厂商 / 方案 | 协作模式 | 质量门禁 |
|------|------|------|
| **MiniMax Mavis** | Worker ↔ Verifier 对掐，Engine 中转 | Verifier 嵌入生产状态机，每步立挑 |
| **Anthropic Multi-Agent Research System** | Lead Agent 分发 + 基于 outcome 评分 | 质量主要靠 Lead 的判断 |
| **Anthropic Claude Code [[#三、Claude Code teamcreate 实战\|teamcreate]]** | 多 Agent 并行/串行 + Synthesis Lead | Synthesis Agent 做最后校验 |
| **OpenAI Agents SDK** | Handoff 接力（A→B→C） | 每一棒都不回头 |
| **Google ADK + A2A 协议** | 跨厂商 Agent 互通 | 协议层，质量门禁由实现方决定 |

---

## 三、Claude Code teamcreate 实战

Claude Code 提供了 `teamcreate` 前缀指令，强制 CLI 进入多 Agent 协作调度模式而非单线程子任务。这是 Anthropic 把 Agent Team 落地到 IDE 工具的具体形态。

### 核心触发机制

- **`teamcreate` 前缀**：必须显式声明，触发多 Agent 模式。
- **Objective**：明确团队最终交付物。

### 协作与调度三种模式

| 模式 | 适用 | 例子 |
|------|------|------|
| **并行（Parallel）** | 互不干扰的任务 | 不同社媒写手、不同维度的市场分析师 |
| **串行（Sequential）** | 存在严格前置依赖 | 研究员找数据 → 幻灯片写手填内容 → 设计师排版 |
| **汇聚（Synthesis）** | 需要最终校验合并 | 指定 Team Lead / Synthesis Agent 做信息去重、矛盾标记 |

### Prompt 工程 3 要素

- **Role（角色划分）**：精确定义每个 Agent 的身份、职责、产出边界。
- **Context（全局上下文）**：通过 `@` 挂载本地文件、提供背景或业务 URL，让所有 Agent 共享一套知识库。
- **Constraints（强制约束）**：设定汇报机制（如"开始前相互分享洞察"）、格式限制、人工干预节点（如"需等待用户审批"）。

> ⚠️ **架构视角的提醒**：teamcreate 的本质是 prompt 编排，质量保证主要靠 Constraints 里写的"互相 review"。从 Mavis 的视角看，这种验证是**可选的附加步骤**，没有程序化兜底——长任务下仍有漂移风险。把 teamcreate 用于一次性产出 OK，用于长期生产建议补一层独立审校 Agent + 人工 checkpoint。

---

## 四、七大经典场景提示词模板

### 场景一：Content Repurposing Engine（内容多平台分发引擎）

**适用**：长文本/长视频转化为各平台短内容，主旨一致但角度互斥。

```text
teamcreate

Objective: Repurpose a video transcript into content for four platforms.

Spawn an agent team with the following roles:
- A blog writer
- A LinkedIn writer
- A newsletter writer
- A Twitter writer

Workflow & Constraints:
1. Before writing, each teammate MUST read the full transcript and identify their three most compelling insights.
2. Have them share their chosen insights with each other to ensure that no two platforms lead with the same angle. Each piece should feel fresh and not repetitive.
3. Finally, synthesize a summary comparing the angles each teammate chose and flag any messaging inconsistencies.
```

**意图**：通过"分享洞察"约束实现角度去重，最后用 Synthesis 做信息一致性兜底。

---

### 场景二：Research & Pitch Deck Builder（研究与路演 PPT）

**适用**：严谨的上下游数据传递流（调查 → 撰写 → 格式化输出）。

```text
teamcreate

Objective: Build a 12-slide pitch deck about how AI automation is transforming small business operations in 2026.

Spawn 3 teammates with strict task dependencies:
- The Researcher: Find 8 to 10 data points, stats, and supporting evidence.
- The Slide Writer: Create content with a max of 8 words, 3 to 4 bullets, and some speaker notes at the bottom of each slide based on the research.
- The Designer: Using the Slide Writer's content, build the actual file using the Python HTML to PPTX library.

Workflow & Constraints:
Require plan approval from me (the user) for the designer before they start building the final file.
```

**意图**：用 `strict task dependencies` 实现 Sequential 模式；用户审批节点做人工 checkpoint，对应 Mavis 的"重试上限触发人类升级"机制。

---

### 场景三：RFP / Proposal Response（招投标方案响应）

**适用**：超长且复杂的甲方需求，通过并行阅读和分块撰写提高效率。

```text
teamcreate

Objective: Respond to a request for proposal at [Insert RFP URL].

Context:
We are a 15-person AI consulting firm specializing in building custom automation workflows for mid-market companies. We use tools like Claude Code. Our average project size is between [X] and [Y] and we've completed 40+ projects.

Spawn four agents:
- RFP Analyst
- Company Capability Researcher
- Section Writer A
- Section Writer B

Workflow & Constraints:
After both Section Writer A and Section Writer B have finished, review all sections for consistent tone and terminology. Ensure no contradictions exist between sections, verify every RFP requirement is addressed, and flag any requirements that we missed.
```

**意图**：Context 提供全局共享背景，最后做语气一致性 + 需求覆盖度的双重校验。

---

### 场景四：Competitive Intelligence Report（竞品情报分析）

**适用**：从多个独立数据源获取信息，最终横向对比。

```text
teamcreate

Objective: Build a competitive intelligence report.

Context:
Target product: Claude Code, the AI-powered coding assistant CLI.
Competitors to analyze: anti-gravity, cursor, codeex, and co-pilot.

Workflow & Constraints:
1. Document the latest and greatest info as of 2026 for each platform.
2. Have each analyst share their top three findings with the group BEFORE the final synthesis begins.
```

**意图**：并行调研 + 强制信息共享，避免 Synthesis 阶段才发现重大遗漏。

---

### 场景五：AI Advisory Board（AI 决策智囊团）

**适用**：重大决策前的沙盘推演，引入"魔鬼代言人"对抗单一思维。

```text
teamcreate

Objective: Analyze a complex business decision: Should we launch a $7,500 live six-week AI leadership boot camp for execs and CEOs who want to manage AI teams and integrate it into their operations?

Spawn five agents to investigate different angles:
- The Market Researcher (Analyzes the executive AI education market)
- The Audience Gap Analyst
- The Financial Modeler
- The Competitive Strategist
- The Devil's Advocate (Critiques and challenges the assumption)

Workflow & Constraints:
Once consensus or informed disagreement emerges, synthesize into a single executive brief containing:
1. A go/no-go or conditional recommendation.
2. Top three reasons for the recommendation.
3. Top three risks regardless of the decision.
4. Suggested next steps.
```

**意图**：**Devil's Advocate 角色就是 Mavis Verifier 的雏形**——天然心思相反、专挑刺。这是 teamcreate 里最接近 Worker-Verifier 对抗的场景设计。

---

### 场景六：Marketing Campaign Launch（营销活动全案）

**适用**：邮件、广告、社媒、落地页的立体营销方案。

```text
teamcreate

Objective: Build a complete marketing campaign for the product launch of [Focus Pods Pro].

Spawn agents with the following roles:
- Email Marketer
- Social Media Manager
- Ad Copywriter: Create 4 variations (Variation A: problem agitation; Variation B: social proof angle; Variation C: us versus them comparison; Variation D: take on the persona of Edward Bernays).
- Landing Page Creator
- Synthesis Lead (Team Lead)

Workflow & Constraints:
The Synthesis Lead must ensure brand consistency and cohesive messaging across all deliverables before generating the final campaign markdown files.
```

**意图**：单角色内部再做 4 个变体（A/B/C/D），是 prompt 层面的"多 Worker 并行"。

---

### 场景七：Personal AI Assistant（定制 CLI 助手代码库）

**适用**：解析现有开源架构并重构定制化项目。

```text
teamcreate

Objective: Build a personal AI assistant, a customized version of OpenClaw built specifically to run my company Prompt Advisors.
End goal: A working CLI command line interface connected to Telegram, capable of understanding our business context, picking the right tools, and helping run the company day-to-day.

Pre-requisite Task:
First, before spawning the main team, use a sub-agent to clone and analyze the repo [Insert Repo URL]. Summarize the overall architecture, design patterns, dependencies, the 3 best ideas we should steal, and the 3 things we should do differently.

Spawn five teammates to build the custom version based on the sub-agent's analysis:
- Architect
- Telegram Interface Engineer
- Skill Router and Tools Engineer
- Memory and Context Integration Engineer
- CLI Developer
```

**意图**：**Pre-requisite Task 是关键模式**——用 sub-agent 先做侦察分析，主团队再基于侦察结果分工建造。两阶段调度 + 信息漏斗，是 teamcreate 里最复杂的工作流编排。涉及 [[entities/openclaw]] 这类参考实现。

---

## 五、把架构和实战拼起来：四条可操作的纪律

读完 Mavis 架构论 + Claude Code teamcreate 实战，提炼出长程 AI 任务的四条朴素纪律：

### 1. 写的不审，审的不写

写内容的 AI 不审自己的内容。**审校必须启动一个独立的 Agent**，看不到原始 Agent 的过程上下文，只看产出文件。

为什么：AI 让自己自检，检查的对象就是它刚刚漂移生成的现场。

落到 Claude Code：在 teamcreate 的 Constraints 里写明"reviewer agent must not have access to writer's working notes"，或者在 [[concepts/claude-md|CLAUDE.md]] 里做硬约束。

### 2. 真正的分工是工具集分工，不是角色扮演分工

会计用 Excel，设计师用 Figma。给不同 Agent 配不同的 [[concepts/coding-agent|工具]]、不同的 skill、不同的工作目录，能力边界才真正切开。

### 3. 验证嵌入生产，不是事后跑

Verifier 不能是事后跑的独立工具，必须**嵌在生产状态机里**。每一步产出立刻挑，挑不过就自动叫回来重做。

工程实现：teamcreate 当前还做不到 runtime 级别的自动 verify-retry 循环，但可以通过 Constraints 里的"each agent must review previous agent's output before proceeding"做软实现。

### 4. 重试有上限，超限升级到人

死循环是必然会发生的。预设重试上限（如 3 轮分数不涨就停），自动把决策升级到人类，比无限消耗 token 强。

---

## 六、Agent 协作的下半场：从工具到同事

读完这两份资料的核心判断：**AI 协作的下一个阶段，是把 AI 从"工具"变成"同事"**。

| | 工具 | 同事 |
|--|------|------|
| **身份** | 单次的、无身份 | 有持续身份 |
| **记忆** | 用完即弃 | 记得上次干到哪里、犯过什么错 |
| **配套** | 只有指令 | 验收标准 + 可调度流水线 + 可复盘日志 |
| **底座** | Prompt 编排 | Runtime（Worker-Verifier-Engine） |

teamcreate 目前还在"工具"和"同事"之间——它能让 Agent 一次性协作产出，但没有持续身份、没有跨任务记忆。从 Mavis 的视角看，**Anthropic 在 [[entities/claude-plugins-official|Claude Code]] 中接入 Agent Teams 是第一步，下一步要补的是 Engine 层**：确定性程序调度、持久化身份、自动 verify-retry。

### 一条策略警示

> "Team 不是默认选项，是策略选项。任务越短、越低风险、越确定，单 Agent 甚至脚本就够了。"
> —— MiniMax Mavis 技术报告

不要为了"看起来高级"而对所有任务都套 teamcreate。Agent Team 的开销（token、协调成本、调试难度）只在长程、高风险、高不确定性的任务上才划得来。

---

## 相关概念与主题

- [[concepts/agentic-engineering]]
- [[concepts/coding-agent]]
- [[concepts/claude-md]]
- [[topics/ai-agent-principles]]
- [[topics/claude-code-tips]]
- [[entities/openclaw]]

## 来源

- [[../raw/ai-chat/claude-code-agent-team-guide.md]]
- [[../raw/ai-chat/mavis-multi-agent-architecture.md]]
