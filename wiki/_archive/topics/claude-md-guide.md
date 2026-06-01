---
title: CLAUDE.md 上下文文件：ETH Zurich 研究解读
type: topic
tags: [ai-coding, ai]
sources:
  - 'raw/feishu/claude-md-beginner-guide.md'
created: 2026-04-26
updated: 2026-06-01
summary: ETH Zurich 研究（arXiv 2602.11988）显示，LLM 自动生成的 CLAUDE.md/AGENTS.md 类上下文文件多数会拖累 AI 编程 Agent 成功率，人工编写的文件可小幅提升，两类均使推理成本上升约 20%。
confidence: medium
---

# CLAUDE.md 上下文文件：ETH Zurich 研究解读

## 概述

ETH Zurich 的一项研究（arXiv 2602.11988）评估了上下文文件对 AI 编程 Agent 的影响。结论需区分文件来源：由 LLM 自动生成的上下文文件普遍拖累任务成功率，而由开发者人工编写的文件在部分基准上能小幅提升表现。无论哪一类，启用上下文文件都会让推理成本上升约 20%。

## 分析

- **LLM 生成的上下文文件**：多数场景下"无文件"表现更好（飞书源概述为"8 项测试中 5 项更好"；论文 abstract 按数据集给出变化，如 AGENTbench 约 -3%、SWE-bench Lite 约 -0.5%）。
- **人工编写的上下文文件**：在 AGENTbench 上约 +4%，方向与 LLM 生成文件相反。
- **成本**：两类文件均使推理成本上升约 19–23%（即 20%+）。
- **拖累原因**：论文将 LLM 生成文件的负面影响归因于与现有文档冗余、未提供有效概览——去除其它文档后，LLM 生成文件转为约 +2.7%。（注："更强模型也生不出更好文件、问题在架构而非内容质量"系飞书源的二次解读，并非论文原文；论文给出的原因是"与现有文档冗余"。）
- **命名**：研究以通用 `agents.md` 为对象，按工具选用对应文件名，Claude Code 对应的是 CLAUDE.md。CLAUDE.md 与 AGENTS.md 在该研究中作为同类上下文文件处理。

## 结论

上下文文件并非越多越好：LLM 自动生成、与现有文档冗余的文件往往有害，而人工编写、能提供有效概览的文件才可能带来小幅收益，且始终伴随约 20% 的成本上升。是否添加应取决于文件质量与能否避免冗余，而非盲目堆叠。

## 来源

- `raw/feishu/claude-md-beginner-guide.md`

## 相关概念

- [[concepts/claude-md]]
- [[concepts/agents]]
- [[concepts/coding-agent]]
- [[concepts/context-files]]
