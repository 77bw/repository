---
title: 上下文文件
type: concept
tags: [ai-coding, ai]
sources: [raw/Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md]
created: 2026-04-26
updated: 2026-04-26
summary: 向 AI Agent 提供项目级别指令的配置文件（如 CLAUDE.md、AGENTS.md），研究表明其效果存在争议。
---

# 上下文文件

## 定义

向 AI Agent 提供项目级别指令的配置文件，如 [[concepts/claude-md|CLAUDE.md]]、[[concepts/agents|AGENTS.md]]。放置于项目根目录，Agent 启动时自动读取。

## 核心要点

- 用于定义项目背景、规范、禁止行为等上下文
- ETH Zurich 研究：有上下文文件时 AI 任务成功率反而下降，成本增加 20%+
- 不同工具使用不同文件名：Claude Code 用 CLAUDE.md，OpenCode 用 AGENTS.md

## 相关概念

- [[concepts/claude-md]]
- [[concepts/agents]]
- [[concepts/coding-agent]]

## 来源

- [[topics/claude-md-guide]]
