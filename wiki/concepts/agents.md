---
title: AGENTS.md
type: concept
tags: [ai-coding, ai]
sources:
  - 'raw/feishu/claude-md-beginner-guide.md'
created: 2026-04-26
updated: 2026-06-01
summary: 跨工具通用的 AI Agent 上下文配置文件名（OpenAI 主导的开放格式，Codex、Cursor 等采用），用于提供项目级指令；它不是 Claude Code 官方支持的文件名，Claude Code 使用 CLAUDE.md。
confidence: low
---

# AGENTS.md

## 定义

AGENTS.md 是多个 AI 编程工具采用的通用上下文配置文件名（OpenAI 主导的开放格式），作用与 [[concepts/claude-md|CLAUDE.md]] 类似——向 AI Agent 提供项目级指令。但它**不是 Claude Code 官方支持的文件名**：官方 best-practices 文档从未提及 AGENTS.md，实际使用 Claude Code 时应创建 CLAUDE.md。

## 核心要点

- 作为跨工具约定，AGENTS.md 用于向 AI Agent 提供项目级指令；在 Claude Code 语境下，其等价物是 CLAUDE.md（在 Claude Code 项目里创建 AGENTS.md 不会被加载）
- 不同工具按各自原生约定选用文件名：Codex / Cursor 等用 AGENTS.md，Claude Code 用 CLAUDE.md

## 相关概念

- [[concepts/claude-md]]
- [[concepts/context-files]]

## 来源

- `raw/feishu/claude-md-beginner-guide.md`
