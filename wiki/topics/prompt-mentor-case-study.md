---
title: 提示词导师实战案例
type: topic
tags: [prompt-engineering, ai-coding]
sources:
  - raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md
  - raw/ai-chat/vibe-coding-how-to.md
  - raw/ai-chat/claude-code-agent-team-guide.md
created: 2026-05-21
updated: 2026-06-01
summary: 提示词导师方法论的实战演示：Shopee 爬虫案例完整交付包、Skill/Subagent/MCP 调度速查表、双 CLI 协作实操要点。
confidence: medium
---

# 提示词导师实战案例

## 概述

本文从 [[topics/prompt-mentor-guide]] 拆分，聚焦提示词导师方法论的实战演示：用 Shopee 爬虫案例展示从澄清到交付的完整产出，附 Skill/Subagent/MCP 调度速查表和双 CLI 协作实操要点。方法论四阶段流程与启动语模板见父页面。

## 用爬虫案例演示完整产出

下面是用启动语处理 shopee 案例后，导师 Agent 应该产出的交付包样例。

### 阶段 1：澄清问题示例

导师应该问你（而不是直接动手）：

1. `seller-shopee-spider` 当前是新建空目录，还是已有部分代码？是否需要保留现有 git 历史？
2. "多国家"具体是哪几个站点？域名规则是 `shopee.{country_tld}` 还是子目录形式？登录态是否每国独立保存？
3. Kafka 上传是否需要去重 / 幂等？失败时是否落本地兜底文件还是直接抛错？

### 阶段 4：最终交付包样例

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
- 主流程模板：seller-lazada-spider 现有目录结构（按它的分层组织新模块）
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

## Skill / Subagent / MCP 调度速查

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

## 双 CLI 协作的实操要点

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

## 把这套方法存成 skill（可选进阶）

如果你想把启动语固化成一个 `/prompt-mentor` 命令，使用 [[concepts/agentic-engineering|skill-creator]]：

```
你来用 skill-creator 帮我创建一个名为 prompt-mentor 的 skill。
SKILL.md 的核心内容是 wiki/topics/prompt-mentor-guide.md 中的「导师端启动语」。
触发词：prompt 导师、提示词导师、生成提示词、prompt mentor。
```

固化后，下次只要在导师端 CLI 输 `/prompt-mentor`，再贴上 Goal/Context/Constraint，就自动进入四阶段流程。

## 相关

- [[topics/prompt-mentor-guide]] — 方法论四阶段流程与启动语模板
- [[topics/agent-team-architecture]] — 子代理架构
- [[topics/vibe-coding]] — Vibe Coding 四步方法论
- [[concepts/agentic-engineering]]
