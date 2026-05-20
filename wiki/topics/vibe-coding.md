---
title: Vibe Coding 实践
type: topic
tags: [ai-coding, prompt-engineering]
sources:
  - "raw/notion/articles/vibe-coding-practice.md"
  - raw/ai-chat/vibe-coding-how-to.md
created: 2026-03-24
updated: 2026-05-21
summary: AI 辅助编程方法论 Vibe Coding：通过需求→设计→任务划分→多 Agent 协作四步流程引导 AI 输出可维护代码，以 B 站老王 FC 模拟器开发为完整实例。
confidence: medium
---

# Vibe Coding 实践

## 概述

Vibe Coding 是一套结构化的 AI 辅助编程方法论：开发者用自然语言描述意图、用结构化提示词约束流程，让 AI 按部就班输出代码，开发者专注方向与验证。它解决的核心痛点是**项目复杂后 AI 犯错概率飙升、代码堆积成垃圾场、上下文窗口爆炸**——直接让 AI "写一个 X"会失控，但拆解成四步 + 多 Agent 协作后能让 AI 持续产出可维护代码。

B 站程序员老王在 [VibeCoding就该这么做！](https://www.bilibili.com/video/BV1YP5W6ZEP9) 中用"AI 从零开发 FC 模拟器跑超级玛丽"为实例，完整演示了这套方法论。

## 分析

### 一、四步方法论

| 步骤 | 输入 | 输出 | 关键技巧 |
|------|------|------|----------|
| **1. 需求确定** | 一句话目标 + 项目环境 | `doc/README.md` 需求文档 | 让 AI 提问，不要猜测意图 |
| **2. 系统设计** | 需求文档 | `doc/design.md` 详细设计 | 强调模块独立性 |
| **3. 任务划分** | 需求 + 设计 | 各模块 `task` 清单 + 总体 `progress.md` | checkbox 标记完成状态 |
| **4. 程序实现** | 需求 + 设计 + 任务 | `doc/prompt.md` 多 Agent 提示词系统 | 主 Agent 监工 + N 个子 Agent 并行 |

每一步都有明确的输入输出，让 AI 按部就班工作，避免盲目编程导致代码混乱。

### 二、提示词构建的"四要素"公式

一个完整的 Vibe Coding 提示词必须包含：

```
目标（做什么）+ 输入（提供什么）+ 输出（生成什么）+ 步骤（怎么做）
```

**核心技巧：让 AI 主动提问，不猜测意图。** 在"步骤"中明确告知"我不了解该领域的任何知识，请使用提问的方式帮助我确定需求，不要猜测我的意图，任何不明确的地方都必须向我提问。"

需求确定阶段的实例：

```
我希望用 Python 开发一个 FC 模拟器，最终能跑超级马力。

目标：用 Python 开发 FC 模拟器，能运行超级玛丽游戏
输入：当前文件夹是 UV Python 工程，ROM 文件夹有超级镜像文件
输出：在 doc 文件夹下生成需求文档 README.md
步骤：我不了解任何 FC 模拟器的有关知识，请使用提问的方式帮助我确定需求，不要猜测我的意图，任何不明确的地方都必须向我提问。
```

### 三、多 Agent 协作架构

> 解决单 Agent 上下文过长导致的信息压缩、幻觉、Token 浪费。

```
┌─────────────────┐
│   主 Agent (监工)  │
│  - 读 progress.md │
│  - 调度子 Agent   │
│  - 汇总结果       │
└────────┬────────┘
         │
    ┌────┼────┬─────────┐
    ▼    ▼    ▼         ▼
[子A1] [子A2] [子A3]  [子AN]
 模块1  模块2  模块3   模块N
 任务+设计+测试独立上下文
```

**职责切分：**
- **主 Agent**：监控整体进度（`progress.md`），统筹子 Agent，不写具体代码
- **子 Agent**：只持有对应模块的设计文档 + 任务清单，独立编码 + 测试，结束后将结果汇报主 Agent

**质量门禁：**
- 每行代码必须有对应单元测试
- 全部通过 `pytest` 和 `ruff` 检测
- 推荐启用 `mypy` 类型检查器
- 建议使用 Docker 容器化或权限管理避免提示词注入风险

### 四、与 [[entities/everything-claude-code]] 的关系

Everything Claude Code 提供了 Vibe Coding 方法论的工程化落地——`/plan`、`/tdd`、`/orchestrate` 等命令对应到四步方法论的"设计 / 实现 / 多 Agent 编排"环节，`memory-bank` 文件夹相当于 Vibe Coding 流程中的 `doc/` 工件存储。两者关注点一致，前者偏方法论、后者偏工具实现。

## 结论

Vibe Coding 的核心洞察是：**AI 编程的失败几乎都是上下文管理失败**。

- 单 Agent 一次性"写完整个项目"→ 上下文爆炸 → 信息压缩 → 幻觉 → 代码混乱
- 四步流程把项目拆成可独立交付的"需求→设计→任务→实现"链路 → 每一步上下文短而清晰
- 多 Agent 协作把"实现"再拆成 N 个并行子任务 → 每个子 Agent 上下文仅含本模块信息

迁移到弱类型语言（Python/JavaScript）时必须前置类型检查、风格检查、单元测试三件套作为质量门禁，否则 AI 生成代码的"幻觉成本"会反噬开发效率。

## 相关概念与实体

- [[concepts/agentic-engineering]]
- [[concepts/coding-agent]]
- [[entities/everything-claude-code]]
- [[topics/agent-team-architecture]]
- [[topics/tdd-ddd-data-collection]]

## 来源

- `raw/notion/articles/vibe-coding-practice.md`（GitHub twwch/vibe-coding 项目）
- `raw/ai-chat/vibe-coding-how-to.md`（B 站程序员老王 FC 模拟器视频解读）
