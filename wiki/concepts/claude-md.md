---
title: CLAUDE.md
type: concept
tags: [ai-coding, ai]
sources:
  - raw/feishu/Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md
  - raw/ai-chat/claude_md_complete_guide.md
created: 2026-04-26
updated: 2026-05-15
summary: AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。
---

# CLAUDE.md

## 定义

AI 编程 Agent 的上下文配置文件，用于向 LLM 提供项目级别的指令和约束。放置于项目根目录或四个层级（托管策略层 / 用户层 / 项目层 / 本地层），Claude Code 启动时按层级拼接加载。

## 核心要点

- 四个层级：托管策略层（企业全员）、用户层（`~/.claude/CLAUDE.md`）、项目层（`./CLAUDE.md`）、本地层（`./CLAUDE.local.md`）
- 多文件**拼接**而非覆盖；越靠近启动目录权重越高
- 工作目录以下的子目录 CLAUDE.md **懒加载**——Claude 读取该目录文件时才触发；可通过 `@import` 语法强制加载
- 配套机制 `.claude/rules/` 通过 `paths` glob 实现路径作用域规则（确定性触发）
- 自然语言渐进式披露指令（文档地图型 / 任务触发型 / 目录规范型 / 条件读取型）
- ETH Zurich 研究：盲目添加 CLAUDE.md 反而降低任务成功率、成本增加 20%+——问题出在架构层面而非内容质量

## 相关概念

- [[concepts/agents]]
- [[concepts/context-files]]
- [[concepts/coding-agent]]

## 来源

- [[topics/claude-md-guide]]
- [[topics/claude-md-mechanisms]]
