---
title: 提示词导师 Agent 完整指南
type: topic
tags: [prompt-engineering, ai-coding]
sources:
  - raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md
  - raw/ai-chat/VibeCoding就该这么做.md
  - raw/ai-chat/claude-code-agent-team-guide.md
created: 2026-05-20
updated: 2026-05-21
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

### 三、用爬虫案例演示完整产出

下面是用上述启动语处理 shopee 案例后，导师 Agent 应该产出的交付包样例。

#### 阶段 1：澄清问题示例

导师应该问你（而不是直接动手）：

1. `seller-shopee-spider` 当前是新建空目录，还是已有部分代码？是否需要保留现有 git 历史？
2. "多国家"具体是哪几个站点？域名规则是 `shopee.{country_tld}` 还是子目录形式？登录态是否每国独立保存？
3. Kafka 上传是否需要去重 / 幂等？失败时是否落本地兜底文件还是直接抛错？

#### 阶段 4：最终交付包样例

```markdown
# Shopee 卖家中心采集流水线

## 角色
你是 Python 爬虫工程师，负责在 seller-shopee-spider/ 实现登录-采集-上传
完整流水线。你有读写本仓库的权限，但参考代码所在的 patrick_star 仓库
和 seller-lazada-spider 仓库为只读。

## 目标
在 d:\SpiderCode\dev\livelab-crawler\seller-shopee-spider 实现：
Playwright 自动登录 → 维持登录态 → 调用 simple_downloader 采集业务数据
→ 按 LAZADA 数仓文档定义的总数据结构上传 Kafka。
支持多国家站点（shopee.sg / shopee.my / shopee.id 等）。

验收标准：
1. 单元测试覆盖登录、采集、Kafka 序列化三个模块
2. 至少跑通一个国家站点的端到端 dry-run（Kafka 改本地 jsonl 输出）
3. ruff + pytest 全绿

## 上下文

参考实现（只读）：
- 主流程模板：seller-shopee-spider 现有目录结构（按它的分层组织新模块）
- 请求库：seller-shopee-spider/utils/simple_downloader.py（不要换 requests / httpx）
- Kafka 数据结构：D:\SpiderCode\VAT\patrick_star\livelab\live_dp\crawlers\http
  下的 lazada 实现（重点看 _build_kafka_payload 类方法）
- 登录参考：D:\SpiderCode\VAT\patrick_star\livelab\live_dp\cookie_keeper\
  browser_refresher.py 的 _login_sellercenter（重点是验证码处理与 cookie 注入）
- 数据契约：seller-lazada-spider/docs/LAZADA 数据 Kafka 对接数仓文档.md

## 工作流程

Step 1 — 摸清现状（5 分钟）
  使用 Explore 子代理并行读取上述 4 个参考文件，整理：
    - simple_downloader 的接口签名和重试策略
    - _login_sellercenter 的登录流程（点击顺序 / cookie 字段）
    - lazada 的 Kafka payload 字段
  输出 docs/research-notes.md

Step 2 — 写设计文档（10 分钟）
  调用 writing-plans skill 产出 docs/design.md：
    - 模块边界（auth / fetch / transform / publish）
    - 多国家配置方案（推荐 yaml 配置 + 工厂模式）
    - 异常处理策略（登录失败 / 验证码 / Kafka 不可达）

Step 3 — 实现 + 自检（迭代）
  按 docs/design.md 模块顺序写代码。每写完一个模块：
    - 写 pytest 单测
    - 跑 ruff check
    - 用 systematic-debugging skill 复盘任何 bug

Step 4 — 端到端 dry-run
  Kafka client 注入 mock，跑一次完整链路，确认数据结构与文档对齐。

## 约束

必须：
- 请求统一走 simple_downloader，不引入新请求库
- 多国家通过配置驱动，不硬编码域名
- Kafka payload 与 lazada 现有结构字段对齐（字段名、类型一致）

禁止：
- 修改 patrick_star / seller-lazada-spider 任何文件
- 在代码里硬编码账号密码
- 跳过 ruff / pytest 直接交付

## 自检清单

完成后逐项确认：
- [ ] 至少 3 个国家站点配置存在
- [ ] cookie 持久化文件不进 git（写 .gitignore）
- [ ] Kafka schema 字段与 LAZADA 文档逐字对齐
- [ ] 登录失败有自动重试 + 告警分支
- [ ] 单测覆盖 ≥ 80%

## 第一步动作

立即调用 Explore 子代理读取 4 个参考文件，输出 docs/research-notes.md。
读完后回报现状摘要再开始写代码，不要直接动手。
```

### 四、Skill / Subagent / MCP 调度速查

导师 Agent 在拆解阶段应该按下表分配工具：

| 任务类型 | 推荐 skill / subagent / MCP | 原因 |
|----------|----------------------------|------|
| 模糊需求收敛 | `brainstorming` skill | 强制走需求探索流程 |
| 跨文件代码摸底 | `Explore` 子代理 | 只读窗口大、不污染主上下文 |
| 第三方库 API 查询 | `context7` MCP | 训练数据可能过时 |
| 历史经验复用 | `claude-mem mem-search` skill | 跨会话长期记忆 |
| 多步实施规划 | `writing-plans` skill | 强制写 plan 再动手 |
| 并行独立任务 | `subagent-driven-development` skill | 主 Agent 监工 + 子 Agent 执行 |
| 调试任何 bug | `systematic-debugging` skill | 强制走假设-验证流程 |
| Git 工作流 | `git-feature` / `smart-commit` / `git-pr` | 产品轨标准化 |
| 完工自检 | `requesting-code-review` skill | 防漏 |

详细架构见 [[topics/agent-team-architecture]]，Skill 完整能力见 [[topics/claude-code-tips]]。

### 五、双 CLI 协作的实操要点

**导师端 CLI（生成 prompt）：**
- 进入项目根目录，让导师能 `Read` / `Bash` 摸清现状
- 启动语粘贴后，老老实实回答它的反问
- 拿到交付包后通读一遍，补任何它没问到的隐藏约束
- 复制最终的 markdown 块

**执行端 CLI（执行任务）：**
- **必须**新开一个干净上下文（不复用导师端会话）
- 进入项目根目录，先 `/clear` 再粘贴交付包
- 让它先执行「第一步动作」，看汇报再放手
- 它出问题时，回到导师端 CLI 让导师重新调整 prompt，而不是在执行端硬掰

### 六、把这套方法存成 skill（可选进阶）

如果你想把上述启动语固化成一个 `/prompt-mentor` 命令，使用 [[concepts/agentic-engineering|skill-creator]]：

```
你来用 skill-creator 帮我创建一个名为 prompt-mentor 的 skill。
SKILL.md 的核心内容是 wiki/topics/prompt-mentor-guide.md 中的「导师端启动语」。
触发词：prompt 导师、提示词导师、生成提示词、prompt mentor。
```

固化后，下次只要在导师端 CLI 输 `/prompt-mentor`，再贴上 Goal/Context/Constraint，就自动进入四阶段流程。

## 结论

把"双 CLI 提示词工作流"从手工模式升级成方法论的核心是三件事：

1. **导师与执行严格分离**——干净上下文是 AI 不犯错的最大杠杆
2. **澄清前置、调研次之、拆解第三、交付最后**——任何一步省略都会让执行端补救
3. **显式调度 skill / subagent / MCP**——不要让执行端 Agent 自己琢磨该用什么工具

对照原始的 shopee 案例：原版只有 Goal/Context/Constraint 三段裸信息，升级版多出"角色 / 验收标准 / 工作流程 / 自检清单 / 第一步动作"五个关键章节，并显式调用 Explore / writing-plans / systematic-debugging 三个 skill。这是裸提示词与工业级提示词包的差距。

## 来源

- raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md
- raw/ai-chat/VibeCoding就该这么做.md
- raw/ai-chat/claude-code-agent-team-guide.md
