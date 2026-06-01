---
title: Agentic 工程技术
type: concept
tags: [ai-coding, prompt-engineering]
sources: ["raw/notion/tech-stack/claude-code-top-5-techniques.md"]
created: 2026-04-26
updated: 2026-06-01
summary: 将 AI 编程助手从"聊天工具"升级为"可重复工程系统"的五大核心技巧。
confidence: medium
---

# Agentic 工程技术

将 AI 编程助手从"聊天工具"升级为"可重复工程系统"的五大核心技巧。

> 术语说明：「Agentic 工程 / agentic engineering」系社区与教学用语，非 Claude Code 官方术语。官方表述为 "agentic coding environment" 与 "agentic loop"。

## 核心理念

不要把 AI 当聊天机器人，而是建立一套**系统**：从随意的 Prompting，转变为结构化工作流与工程化的输入输出。

## 五大技巧

### 1. PRD First Development

开发前先写 Markdown 文档定义项目完整工作范围，用 PRD 将大项目拆解为细颗粒度功能。

### 2. Modular Rules Architecture

- 全局规则保持极简（< 200 行），只放通用规则
- 具体任务规则拆到独立 skill（`.claude/skills/api-conventions/SKILL.md`），其正文按需加载
- 只有做特定任务时才加载对应规则

### 3. Skillify Everything

同一个 Prompt 输入超过两次，就把它沉淀为一个 Skill：创建 `.claude/skills/<name>/SKILL.md` 定义可复用工作流，之后用 `/<name>` 直接调用。

> 自定义 commands 已并入 skills：`.claude/commands/deploy.md` 与 `.claude/skills/deploy/SKILL.md` 都生成 `/deploy` 且行为一致。旧的 `.claude/commands/` 文件仅作向后兼容保留；新建工作流推荐用 skill（支持附属文件目录、frontmatter 控制调用方、按需自动加载）。

### 4. 上下文工程

在有限上下文窗口中，选择、组织并注入与任务高度相关的信息，让 LLM 做出最佳推理。

### 5. 子智能体编排

- **循环验证**：主智能体评估子智能体结果，不合格则重新派发（轮次为建议实践，官方无硬性上限；如需约束可在子代理 frontmatter 设 `maxTurns`）
- **顺序编排**：每个代理明确输入输出，中间结果存文件

## 相关主题

- [[entities/everything-claude-code]]
- [[concepts/context-files]]
- [[concepts/coding-agent]]
