---
title: OpenCode + Oh My OpenCode
type: entity
tags: [ai-coding]
sources:
  - "raw/notion/tech-stack/opencode-oh-my-opencode-guide.md"
  - "raw/notion/tech-stack/usage-tips.md"
created: 2026-04-26
updated: 2026-06-01
summary: 开源 AI 编程助手 OpenCode 及其增强插件 Oh My OpenCode，提供多模型编排和专业 Agent 矩阵。
confidence: high
---

# OpenCode + Oh My OpenCode

OpenCode 是开源 AI 编程助手（约 168K Stars）。Oh My OpenCode（OMO）是其全能增强插件，提供多模型编排、专业 Agent 矩阵、工作流自动化和深度代码理解能力。

> 命名说明：GitHub 仓库已从 `oh-my-opencode` 更名为 [`oh-my-openagent`](https://github.com/code-yeongyu/oh-my-openagent)（仓库描述标注 "previously oh-my-opencode"），但 npm 包名、安装命令、配置文件名与品牌名仍沿用 **oh-my-opencode**，旧名保留供检索。

## 安装

```bash
curl -fsSL https://opencode.ai/install | bash
bunx oh-my-opencode install   # npx oh-my-opencode install 亦可；更名后命令不变
```

> 官方更推荐把安装指南交给 LLM agent 执行：让 agent 拉取并按 `oh-my-opencode/docs/guide/installation.md` 操作。

配置文件：用户级 `~/.config/opencode/oh-my-opencode.json`、项目级 `.opencode/oh-my-opencode.json`，OpenCode 主配置为 `~/.config/opencode/opencode.json`（仓库虽更名，配置文件名仍为 `oh-my-opencode.json`）。

## OMO Agent 系统

下表为部分核心 agent（模型默认值，均可在配置中覆盖）：

| Agent | 职责 |
|---|---|
| Sisyphus | 主编排器（默认 Opus 4.5），任务分析拆解委派、并行调度 |
| Oracle | 架构决策、代码审查、高难度调试（只读，GPT-5.2） |
| Librarian | 查找外部文档、多仓库分析、OSS 实现示例 |
| Explore | 极速代码库探索 / 上下文 grep |
| Multimodal Looker | 解析 PDF / 图片 / 图表等多模态内容，节省主上下文 token |
| Prometheus | 编码前制定详尽技术方案（Planner，按 Tab 进入） |
| Metis | 规划前分析，识别隐藏意图和歧义 |
| Momus | 评估工作计划的清晰度、可验证性与完整性 |

> 另有 `frontend-ui-ux`（前端开发，Gemini 3 Pro）等能力以内置 Skill 形式提供。

## 常用命令

| 命令 | 用途 |
|---|---|
| `/init-deep` | 初始化分层 AGENTS.md 知识库 |
| `/ralph-loop` | 启动自循环开发模式直到任务完成 |
| `/ulw-loop` | ultrawork 模式的循环执行（同 ralph-loop 但启用 ultrawork） |
| `/refactor` | LSP/AST-grep + 架构分析 + TDD 校验的智能重构 |
| `/start-work` | 从 Prometheus 计划启动 Sisyphus 工作会话 |

> `ultrawork`/`ulw` 是触发关键词（写进 prompt 即生效），并非 slash 命令；`git-master`、`playwright` 等以内置 Skill 形式提供（如 `/git-master commit these changes`）。

## 使用技巧

1. 简单任务：直接描述，Sisyphus 自动处理
2. 复杂任务：使用 `/ralph-loop` 持续工作
3. 架构咨询：明确说"咨询 Oracle"

## 相关实体与概念

- [[entities/everything-claude-code]]
- [[concepts/agentic-engineering]]
