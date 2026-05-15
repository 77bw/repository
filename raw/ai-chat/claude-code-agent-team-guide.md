# 🧠 Claude Code Agent Team 实战指南与核心提示词库

## 第一部分：Agent Team 核心工作流解析 (思维导图大纲)

*   **核心触发机制**
    *   **显式指令**：必须使用 `teamcreate` 作为前缀，强制 CLI 进入多 Agent 协作调度模式，而非单线程子任务。
    *   **目标声明 (Objective)**：明确团队需要完成的最终交付物。
*   **协作与调度模式**
    *   **并行处理 (Parallel)**：多个 Agent 负责互不干扰的任务（例如：不同的社媒写手、不同维度的市场分析师）。
    *   **串行流转 (Sequential / Task Dependencies)**：存在严格的前置依赖（例如：研究员找数据 -> 幻灯片写手填内容 -> 设计师排版）。
    *   **汇聚综合 (Synthesis)**：通常需要指定一个“Team Lead”或“Synthesis Agent”负责最后的信息校验、去重和总结合并。
*   **Prompt 工程 3 要素**
    *   **Role (角色划分)**：精确定义每个 Agent 的身份、职责和产出边界。
    *   **Context (全局上下文)**：通过 `@` 挂载本地文件、提供背景信息或业务 URL，让所有 Agent 共享一套知识库。
    *   **Constraints (强制约束)**：设定汇报机制（如“开始前相互分享洞察”）、格式限制或人工干预节点（如“需等待用户审批”）。

---

## 第二部分：七大经典场景实战提示词 (中英对照)

### 场景一：Content Repurposing Engine（内容多平台分发引擎）

**适用场景**：将长文本/长视频转化为各平台的短内容，保持主旨一致但切入角度互斥。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：将一段视频脚本重组为适用于四个平台的内容。
> **生成以下角色的团队**：博客写手、LinkedIn 写手、Newsletter 写手、Twitter 写手。
> **工作流与约束**：
> 1. 在开始写作前，每位成员必须阅读完整的文本，并提取出 3 个最引人注目的洞察。
> 2. 强制他们彼此分享挑选的洞察，以确保没有哪两个平台使用相同的主打角度。每篇内容必须新鲜且不重复。
> 3. 最后，综合一份总结，对比每位成员的切入角度，并标记任何信息传递上的矛盾。
> 
> 

---

### 场景二：Research & Pitch Deck Builder（研究与路演 PPT 生成）

**适用场景**：需要严谨的上下游数据传递流（调查 -> 撰写 -> 格式化输出）。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：制作一份 12 页的商业路演 PPT，主题为“2026年 AI 自动化如何重塑小企业运营”。
> **生成 3 个带有严格任务依赖的成员**：
> * 研究员：寻找 8-10 个数据点、统计数据及支撑证据。
> * 幻灯片文案：基于研究数据，为每页幻灯片创建内容（限制：最多 8 个词的标题、3-4 个核心要点，底部附带演讲者备注）。
> * 设计师：使用文案提供的内容，通过 Python 的 HTML to PPTX 库生成实际文件。
> **工作流与约束**：在设计师开始构建最终文件前，必须中断流程并请求我（用户）的审批。
> 
> 

---

### 场景三：RFP / Proposal Response（招投标方案响应）

**适用场景**：处理超长且复杂的甲方需求，通过并行阅读和分块撰写提高效率。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：响应位于 [插入 RFP 网址] 的需求建议书（RFP）。
> **背景**：我们是一家 15 人的 AI 咨询公司，专为中型企业构建自动化工作流。使用 Claude Code 等工具，项目均价为 [X] 至 [Y]，已完成 40+ 项目。
> **生成四个 Agent**：RFP 分析师、公司能力研究员、章节撰写员 A、章节撰写员 B。
> **工作流与约束**：在两位撰写员都完稿后，全局审查所有章节的语气和术语一致性。确保章节间无矛盾，核实每一项 RFP 需求均被回应，并明确标记出我们遗漏的任何需求。

---

### 场景四：Competitive Intelligence Report（竞品情报分析）

**适用场景**：需要从多个独立数据源获取信息，并最终横向对比得出结论。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：生成一份竞品情报分析报告。
> **背景**：目标产品是 Claude Code (AI 编码助手 CLI)。需要分析的竞品包括 anti-gravity, cursor, codeex 和 co-pilot。
> **工作流与约束**：
> 1. 为每个平台收集并记录截至 2026 年最新、最全面的信息。
> 2. 在综合报告开始撰写之前，强制每位分析师先向团队分享他们的三大核心发现。
> 
> 

---

### 场景五：AI Advisory Board（AI 决策智囊团）

**适用场景**：重大决策前进行沙盘推演，引入“魔鬼代言人”对抗单一思维。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：分析一项复杂商业决策：是否该推出售价 7500 美元、为期 6 周的 AI 领导力直播训练营（面向希望管理 AI 团队并落地的 CEO/高管）？
> **生成 5 个负责不同视角的 Agent**：市场研究员、受众缺口分析师、财务建模师、竞争策略师、魔鬼代言人（负责挑刺和挑战假设）。
> **工作流与约束**：当团队达成共识或形成有理有据的分歧时，将其整合成一份高管简报，包含：1. 执行/终止的决定或带条件的建议；2. 该建议的三大理由；3. 无论何种决定都会面临的三大风险；4. 下一步行动建议。

---

### 场景六：Marketing Campaign Launch（营销活动全案策划）

**适用场景**：为一个产品打造一套涵盖邮件、广告、社媒和落地页的立体营销方案。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：为 [Focus Pods Pro] 的产品发布策划完整的营销活动。
> **生成以下角色的 Agent**：邮件营销专家、社交媒体经理、广告文案（需创建 4 个变体：A 痛点放大；B 社会认同；C 竞品对比；D 扮演公关之父爱德华·伯内斯进行策划）、落地页创建者、综合负责人（Team Lead）。
> **工作流与约束**：在输出最终的营销 Markdown 文件前，综合负责人必须确保所有交付物在品牌基调和信息传达上保持一致。

---

### 场景七：Personal AI Assistant（定制个人 CLI 助手代码库）

**适用场景**：利用 Agent Team 解析现有开源架构并重构属于自己的定制化项目。

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

**📝 中文翻译与意图解析**

> **触发团队创建**
> **目标**：构建一个定制版的 OpenClaw 个人 AI 助手，专为运营我的公司 Prompt Advisors 服务。最终目标是交付一个可运行的 CLI 界面，对接 Telegram，理解业务背景并自动调用工具辅助日常运营。
> **前置任务**：在生成主力团队前，先使用一个 Sub-agent 克隆并分析目标代码库 [插入 URL]。总结其整体架构、设计模式、依赖项、我们应该借鉴的 3 个最佳创意，以及我们要改进的 3 个设计。
> **生成团队**：基于前期分析，生成 5 个核心成员进行构建：架构师、Telegram 接口工程师、技能路由工程师、记忆与上下文集成工程师、CLI 开发工程师。

```

```