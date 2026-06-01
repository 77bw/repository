---
title: 上下文文件
type: concept
tags: [ai-coding, ai]
sources:
  - 'raw/feishu/claude-md-beginner-guide.md'
created: 2026-04-26
updated: 2026-06-01
summary: 向 AI Agent 提供项目级别指令的配置文件（如 CLAUDE.md、AGENTS.md），研究表明其效果存在争议。
confidence: medium
---

# 上下文文件

## 定义

向 AI Agent 提供项目级别指令的配置文件，如 [[concepts/claude-md|CLAUDE.md]]、[[concepts/agents|AGENTS.md]]。放置于项目根目录，Agent 启动时自动读取。

## 核心要点

- 用于定义项目背景、规范、禁止行为等上下文
- 不同工具使用不同文件名：Claude Code 用 CLAUDE.md；Cursor、Copilot 等支持 AGENTS.md（OpenAI 主导的开放格式，github.com/openai/agents.md）；部分项目用符号链接统一两者

## 相关概念

- [[concepts/claude-md]]
- [[concepts/agents]]
- [[concepts/coding-agent]]

## 来源

- `raw/feishu/claude-md-beginner-guide.md`
