---
title: Claude Code Best Practice
type: entity
tags: [ai-coding, ai]
sources: [raw/claude-code-best-practice/README.md]
created: 2026-04-26
updated: 2026-04-26
summary: shanraisshan 维护的 Claude Code 最佳实践仓库，涵盖 Subagents/Commands/Skills/Hooks 完整体系与 82 条实战技巧
---

# Claude Code Best Practice

## 简介

由 Anthropic 社区大使 [shanraisshan](https://github.com/shanraisshan) 维护的开源仓库，系统整理 Claude Code 从 Vibe Coding 到 Agentic Engineering 的最佳实践。曾登上 GitHub Trending 日榜第一，是目前最权威的 Claude Code 社区参考资料。

仓库地址：https://github.com/shanraisshan/claude-code-best-practice

## 核心内容结构

### 四大扩展机制（CONCEPTS）

| 机制 | 位置 | 本质 |
|------|------|------|
| **Subagents** | `.claude/agents/<name>.md` | 独立上下文中运行的自主 Actor，可自定义工具/权限/模型/记忆 |
| **Commands** | `.claude/commands/<name>.md` | 注入当前上下文的 Slash 命令，用于工作流编排 |
| **Skills** | `.claude/skills/<name>/SKILL.md` | 可配置、可预加载、支持上下文分叉的知识注入单元 |
| **Hooks** | `.claude/hooks/` | 在 Agent 循环外响应特定事件的用户自定义处理器 |

### 编排模式

核心架构：**Command → Agent → Skill**

```
/weather-orchestrator (Command)
  └─ weather-agent (Agent，预加载 weather-fetcher Skill)
       └─ weather-svg-creator (Skill，生成 SVG 输出)
```

### 开发工作流（DEVELOPMENT WORKFLOWS）

所有主流工作流收敛于同一模式：**Research → Plan → Execute → Review → Ship**

主要工作流框架：
- [Superpowers](https://github.com/obra/superpowers) — TDD 优先 + Iron Laws
- [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) — instinct scoring + AgentShield
- [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) — 完整 SDLC + 22+ 平台
- [RPI](../topics/claude-code-rpi-workflow.md) — Research → Plan → Implement

## 相关概念与实体

- [[topics/claude-code-tips]]
- [[topics/claude-code-best-practice-guide]]
- [[entities/claude-plugins-official]]
- [[concepts/agentic-engineering]]

## 来源

- `raw/claude-code-best-practice/README.md`
- `raw/claude-code-best-practice/best-practice/claude-subagents.md`
- `raw/claude-code-best-practice/best-practice/claude-skills.md`
- `raw/claude-code-best-practice/best-practice/claude-commands.md`
- `raw/claude-code-best-practice/best-practice/claude-settings.md`
- `raw/claude-code-best-practice/best-practice/claude-memory.md`
