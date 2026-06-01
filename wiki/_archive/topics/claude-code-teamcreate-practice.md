---
title: Claude Code teamcreate 七大场景实战
type: topic
tags: [ai, ai-coding, prompt-engineering]
sources:
  - raw/ai-chat/claude-code-agent-team-guide.md
  - raw/ai-chat/mavis-multi-agent-architecture.md
created: 2026-05-21
updated: 2026-05-21
summary: Claude Code teamcreate 七大经典场景提示词模板，涵盖内容分发、路演PPT、招投标、竞品分析、智囊团、营销全案、CLI助手构建。
confidence: medium
---

# Claude Code teamcreate 七大场景实战

## 概述

本文收录 Claude Code `teamcreate` 前缀指令的七大经典场景提示词模板，展示如何通过 Role / Context / Constraints 三要素编排多 Agent 协作。架构背景与核心机制见 [[topics/agent-team-architecture]]。

---

## 场景一：Content Repurposing Engine（内容多平台分发引擎）

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

## 场景二：Research & Pitch Deck Builder（研究与路演 PPT）

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

## 场景三：RFP / Proposal Response（招投标方案响应）

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

## 场景四：Competitive Intelligence Report（竞品情报分析）

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

## 场景五：AI Advisory Board（AI 决策智囊团）

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

## 场景六：Marketing Campaign Launch（营销活动全案）

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

## 场景七：Personal AI Assistant（定制 CLI 助手代码库）

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

## 相关

- [[topics/agent-team-architecture]] — 架构背景与核心机制
- [[concepts/agentic-engineering]]
- [[topics/claude-code-tips]]
