---
title: Agentic 工程技术
type: concept
tags: [ai-coding, prompt-engineering]
sources: []
created: 2026-04-26
updated: 2026-04-26
summary: 将 AI 编程助手从"聊天工具"升级为"可重复工程系统"的五大核心技巧。
---

# Agentic 工程技术

将 AI 编程助手从"聊天工具"升级为"可重复工程系统"的五大核心技巧。

## 核心理念

不要把 AI 当聊天机器人，而是建立一套**系统**：从随意的 Prompting，转变为结构化工作流与工程化的输入输出。

## 五大技巧

### 1. PRD First Development

开发前先写 Markdown 文档定义项目完整工作范围，用 PRD 将大项目拆解为细颗粒度功能。

### 2. Modular Rules Architecture

- 全局规则保持极简（< 200 行），只放通用规则
- 具体任务规则拆到独立文件（`rules/api.md`）
- 只有做特定任务时才加载对应规则

### 3. Commandify Everything

同一个 Prompt 输入超过两次，就把它变成一个 Command（Markdown 文件定义工作流）。

### 4. 上下文工程

在有限上下文窗口中，选择、组织并注入与任务高度相关的信息，让 LLM 做出最佳推理。

### 5. 子智能体编排

- **循环验证**：主智能体评估子智能体结果，不合格则重新派发（最多 3 轮）
- **顺序编排**：每个代理明确输入输出，中间结果存文件

## 相关主题

- [[Everything Claude Code]]
- [[上下文文件]]
- [[编程Agent]]
