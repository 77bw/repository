---
title: CLAUDE.md
type: concept
tags: [ai-coding, ai]
sources: [raw/Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md]
created: 2026-04-26
updated: 2026-04-26
summary: AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。
---

# CLAUDE.md

## 定义

AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。放置于项目根目录，Claude Code 启动时自动读取。

## 核心要点

- 定义项目背景、技术栈、编码规范等上下文信息
- ETH Zurich 研究表明：有 CLAUDE.md 时 AI 任务成功率反而下降，成本增加 20%+
- 问题出在架构层面，而非内容质量——更强的模型也无法生成更好的上下文文件

## 相关概念

- [[concepts/agents]]
- [[concepts/context-files]]
- [[concepts/coding-agent]]

## 来源

- [[topics/claude-md-guide]]
