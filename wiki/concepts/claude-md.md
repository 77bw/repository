---
title: CLAUDE.md
type: concept
tags: [ai-coding, ai]
sources:
  - 'raw/feishu/claude-md-beginner-guide.md'
  - raw/ai-chat/claude_md_complete_guide.md
created: 2026-04-26
updated: 2026-06-01
summary: AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。
confidence: high
---

# CLAUDE.md

## 定义

AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。放置于项目根目录或四个层级（托管策略层 / 用户层 / 项目层 / 本地层），Claude Code 启动时按层级拼接加载。

## 核心要点

- 四个层级：托管策略层（企业全员，系统路径如 macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`）、用户层（`~/.claude/CLAUDE.md`）、项目层（`./CLAUDE.md` 或 `./.claude/CLAUDE.md` 两种位置）、本地层（`./CLAUDE.local.md`）
- 多文件**拼接**而非覆盖；越靠近启动目录权重越高
- 工作目录以下的子目录 CLAUDE.md **懒加载**——Claude 读取该目录文件时才触发（与 [[topics/claude-md-mechanisms]]、[[topics/claude-md-best-practices]] 口径一致，有官方依据）；官方 `@import` 语法可在父级文件中显式引入其他文件，但"用 `@import` 强制加载子目录 CLAUDE.md"这一说法源自 AI 对话素材、未经官方文档证实
- 配套机制 `.claude/rules/` 通过 `paths` glob 实现路径作用域规则（确定性触发）——此为社区实践总结，非官方文档明确确认的详细行为
- 自然语言渐进式披露指令（文档地图型 / 任务触发型 / 目录规范型 / 条件读取型）

## 相关概念

- [[concepts/agents]]
- [[concepts/context-files]]
- [[concepts/coding-agent]]

## 来源

- `raw/feishu/claude-md-beginner-guide.md`
- `raw/ai-chat/claude_md_complete_guide.md`
- [[topics/claude-md-mechanisms]]
