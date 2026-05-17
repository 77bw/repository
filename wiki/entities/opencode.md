---
title: OpenCode + Oh My OpenCode
type: entity
tags: [ai-coding]
sources:
  - "raw/notion/tech-stack/OpenCode + Oh My OpenCode 完整使用指南 2f781caa0df881689124e89483dfa762.md"
  - "raw/notion/tech-stack/使用小tips 2f581caa0df88050ac20db637bfb912c.md"
created: 2026-04-26
updated: 2026-05-18
summary: 开源 AI 编程助手 OpenCode 及其增强插件 Oh My OpenCode，提供多模型编排和专业 Agent 矩阵。
---

# OpenCode + Oh My OpenCode

OpenCode 是开源 AI 编程助手（60K+ Stars），Oh My OpenCode（OMO）是其全能增强插件，提供多模型编排、专业 Agent 矩阵、工作流自动化和深度代码理解能力。

## 安装

```bash
curl -fsSL https://opencode.ai/install | bash
bunx oh-my-opencode install
```

配置文件：`~/.config/opencode/opencode.json` 和 `~/.config/opencode/oh-my-opencode.json`

## OMO Agent 系统

| Agent | 职责 |
|---|---|
| Sisyphus | 主编排器，任务分析拆解委派 |
| Oracle | 高难度调试、复杂架构咨询（只读） |
| Librarian | 查找外部文档、库用法 |
| Prometheus | 编码前制定详尽技术方案 |
| Metis | 分析请求，识别隐藏意图和歧义 |
| Momus | 评估工作计划完整性和可行性 |

## 常用命令

| 命令 | 用途 |
|---|---|
| `/ralph-loop` | 启动自循环开发模式 |
| `/init-deep` | 初始化项目知识库 |
| `/refactor` | 智能重构 |
| `/git-master` | Git 操作专家模式 |

## 使用技巧

1. 简单任务：直接描述，Sisyphus 自动处理
2. 复杂任务：使用 `/ralph-loop` 持续工作
3. 架构咨询：明确说"咨询 Oracle"

## 相关实体与概念

- [[entities/everything-claude-code]]
- [[concepts/agentic-engineering]]
