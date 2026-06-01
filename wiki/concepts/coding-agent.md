---
title: 编程 Agent
type: concept
tags: [ai-coding, ai]
sources:
  - 'raw/feishu/claude-md-beginner-guide.md'
created: 2026-04-26
updated: 2026-06-01
summary: 能够自主执行编程任务的 AI 系统，如 Claude Code、Cursor 等，通过上下文文件获取项目指令。
confidence: medium
---

# 编程 Agent

## 定义

能够自主执行编程任务的 AI 系统，通过读取 [[concepts/context-files|上下文文件]] 获取项目指令，自主完成代码编写、调试、重构等任务。

## 核心要点

- 代表工具：Claude Code、Cursor、OpenAI Codex、Windsurf 等
- 通过上下文文件获取项目级别指令（如 Claude Code 的 CLAUDE.md、Cursor 的 .cursorrules 等）
- 部分工具（如 Claude Code）支持子智能体编排，将复杂任务拆解给多个专职 Agent

## 相关概念

- [[concepts/context-files]]
- [[concepts/agentic-engineering]]
- [[entities/everything-claude-code]]

## 来源

- `raw/feishu/claude-md-beginner-guide.md`
