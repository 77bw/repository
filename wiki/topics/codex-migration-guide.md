---
title: 从 Claude Code 迁移到 Codex CLI / Codex App 使用手册
type: topic
tags: [ai-coding]
sources: [raw/docs/codex-migration-manual.html]
created: 2026-04-26
updated: 2026-06-01
summary: 面向 Claude Code 用户的 Codex CLI/App 完整迁移手册，含命令映射、权限配置、工作流与提示词模板
confidence: medium
---

# 从 Claude Code 迁移到 Codex CLI / Codex App

## 概述

本文面向已有 Claude Code 使用习惯的开发者，聚焦迁移路径：命令映射、权限模型、配置文件、常用工作流。

两者都属于 [[concepts/agentic-engineering]] 体系下的 AI 编程工具，核心理念相同——区别在于生态和操作习惯。

---

## 1. 心智模型

| 工具 | 定位 |
|------|------|
| Codex CLI | 终端本地编码代理，读写文件、执行命令、code review，支持非交互式 |
| Codex App | 桌面任务中心，多项目/多线程/并行代理/worktree/Git/浏览器 |

**迁移建议：** 习惯 Claude Code REPL → 先用 `codex`；需要并行任务、diff 审阅、本地网页 → 用 Codex App。

---

## 2. 快速开始

```bash
# 安装
npm i -g @openai/codex@latest

# 启动交互式 TUI
codex

# 第一次推荐：让 Codex 先读项目，不改文件
codex "请先阅读这个项目，说明目录结构、启动方式、测试方式，不要改文件"

# 低摩擦自动模式
# --full-auto 等价于 -a on-request -s workspace-write（按需审批 + 工作区可写）
# 注意：新版 CLI 顶层已弃用 --full-auto，建议直接写 -s workspace-write（需审批再加 -a on-request）
codex -s workspace-write "修复当前测试失败，保持改动最小"

# 非交互式（适合脚本/CI）
codex exec --cd /path/to/repo "运行测试并总结失败原因"
```

---

## 3. 命令映射、CLI 总览、Slash 命令、权限沙盒、工作流与提示词模板

Claude Code 到 Codex 的完整命令映射表、CLI 命令总览、Slash 命令对照、权限与沙盒三级配置、修 bug / 理解项目 / 长会话续航等常用工作流模板，以及通用提示词模板。

详见 [[topics/codex-command-mapping]]。

---

## 7. 配置与规则文件

**`~/.codex/config.toml`** — 默认配置，`-c key=value` 临时覆盖。

**`AGENTS.md`** — 对应 Claude Code 的 `CLAUDE.md`，建议写入：
- 项目启动/测试/lint 命令
- 禁止修改的目录
- 提交前必跑的验证
- 代码风格与安全红线

```bash
# 在 Codex CLI 里生成脚手架
/init
```

---

## 8. MCP / Skills / Plugins

- **MCP**：`codex mcp list/add/remove` — 接入外部工具或数据源
- **Skills**：可复用工作流，App 中输入 `$` 调用
- **Plugins**：打包 skills + MCP + 工具，适合团队内部能力共享

---

## 9. 常用工作流与提示词模板

修 bug、理解陌生项目、长会话续航等常用工作流，以及通用提示词模板和明确成功标准的写法。

详见 [[topics/codex-command-mapping]]。

---

## 11. 与 Claude Code 工程体系对比

| 维度 | Claude Code | Codex |
|------|-------------|-------|
| 规则文件 | `CLAUDE.md`（可经 `@import` 拆分） | `AGENTS.md` |
| 配置文件 | `.claude/settings.json` | `~/.codex/config.toml` |
| Skills | `/skill-name` | `$skill-name`（App）|
| 子智能体 | Agent tool + subagent_type | `/agent` + `codex fork` |
| 上下文压缩 | `/compact` | `/compact` |
| 记忆持久化 | Hooks（session-start/end） | Codex 已推出持久记忆预览（2026-04 起分阶段放量），AGENTS.md 仍可作显式补充 |

**迁移核心原则**（来自 [[concepts/agentic-engineering]]）：
- `CLAUDE.md` 的内容直接迁移到 `AGENTS.md`，结构不变
- Commandify 习惯保留：把高频 prompt 写成 `AGENTS.md` 里的工作流说明
- [[entities/everything-claude-code]] 的 Hooks 机制在 Codex 暂无对应，需手动用 `AGENTS.md` 补偿

---

## 相关概念

- [[concepts/agentic-engineering]] — Codex 和 Claude Code 共享的工程化 AI 编程理念
- [[concepts/context-files]] — CLAUDE.md / AGENTS.md 的本质与写法
- [[entities/everything-claude-code]] — Claude Code 侧的完整配置参考

---

## 来源

- [[raw/docs/codex-migration-manual.html]]
