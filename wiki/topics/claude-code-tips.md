---
title: Claude Code 高频实用技巧
type: topic
tags: [ai-coding, ai]
sources:
  - "raw/feishu/15 条高频实用的 Claude Code 技巧 - 飞书云文档.md"
  - "raw/feishu/Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md"
created: 2026-04-26
updated: 2026-05-18
summary: 整合 Claude Code 团队内部技巧与社区高频实用技巧，涵盖并行开发、Plan 模式、子代理、技能等核心用法。
---

# Claude Code 高频实用技巧

## 概述

来自 Claude Code 团队内部分享（10 个技巧）与社区整理（15 条高频技巧）的实践经验汇总。

## 工作流技巧

- **git worktree 并行开发**：用 `--worktree` 标志同时推进多个独立任务，互不干扰
- **Plan 模式处理复杂任务**：复杂需求先进入计划模式（Shift+Tab），对齐思路再执行
- **`/clear` 清空上下文**：开启无关任务前清空，避免上下文污染
- **Esc+Esc 回滚**：直接进入 `/rewind` 模式撤销上一步操作
- **直接贴报错**：不要用自己的话转述 bug，把原始报错直接粘给 Claude

## 配置技巧

- **设置 `cc` 别名**：`alias c-d='claude --dangerously-skip-permissions'` 加入 `~/.zshrc`
- **`/init` 初始化 CLAUDE.md**：自动生成后手动调整，是 Agent 的核心记忆文件
- **迭代 CLAUDE.md/AGENTS.md**：随开发持续完善，而非一次性写好

## 能力扩展

- **安装 LSP 插件**：为项目语言安装代码智能插件，提升语义理解
- **创建/使用自定义技能（skills）**：将重复工作流封装为可复用技能
- **使用子代理**：复杂任务拆分给子代理，保持主上下文清晰
- **选择合适的 MCP**：通过 MCP 扩展 Claude Code 能力边界
- **告知具体文件**：明确指定要读取的文件，减少无效探索

## 其他用法

- **数据分析**：可用 Claude Code 处理数据分析任务
- **学习工具**：用 Claude Code 辅助学习新技术或代码库

## 相关概念与实体

- [[concepts/claude-md]]
- [[concepts/coding-agent]]
- [[entities/claude-plugins-official]]

## 来源

- `raw/feishu/15 条高频实用的 Claude Code 技巧 - 飞书云文档.md`
- `raw/feishu/Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md`
