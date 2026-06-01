---
title: Codex 命令映射与实操
type: topic
tags: [ai-coding]
sources: [raw/docs/codex-migration-manual.html]
created: 2026-05-21
updated: 2026-06-01
summary: Claude Code 到 Codex 的完整命令映射、CLI 命令总览、Slash 命令、权限沙盒配置、常用工作流与提示词模板。
confidence: medium
---

# Codex 命令映射与实操

## 概述

本文从 [[topics/codex-migration-guide]] 拆分，聚焦 Codex CLI 的具体操作细节：命令映射表、CLI 命令总览、Slash 命令、权限与沙盒配置、常用工作流模板和提示词模板。迁移心智模型与工程体系对比见父页面。

---

## Claude Code → Codex 命令映射

| Claude Code | Codex 对应 | 说明 |
|-------------|-----------|------|
| `claude` | `codex` | 启动交互式会话 |
| `claude "query"` | `codex "query"` | 启动时带首条提示 |
| `claude -p "query"` | `codex exec "query"` | 非交互式执行 |
| `claude -c` | `codex resume --last` | 继续最近会话 |
| `claude --resume <id>` | `codex resume <id>` | 按 ID 恢复会话 |
| `claude mcp` | `codex mcp` | 管理 MCP 服务器 |
| `CLAUDE.md` | `AGENTS.md` | 项目长期指令文件 |
| `.claude/settings.json` | `~/.codex/config.toml` | 配置文件（Codex 用 TOML） |

---

## CLI 命令总览

| 命令                       | 用途                  |
| ------------------------ | ------------------- |
| `codex`                  | 启动交互式 TUI           |
| `codex exec` / `codex e` | 非交互式执行              |
| `codex review`           | 非交互式代码审查            |
| `codex resume`           | 恢复旧会话               |
| `codex fork`             | 从旧会话分叉新线程           |
| `codex app`              | 启动桌面 App            |
| `codex mcp`              | 管理 MCP 服务器          |
| `codex features`         | 查看/切换 feature flags |

**常用全局参数：**

| 参数 | 作用 |
|------|------|
| `--full-auto` | **已废弃**：顶层不再解析（`codex --full-auto` 会报错），改用 `-s workspace-write`（需审批再配 `-a on-request`） |
| `-s read-only / workspace-write / danger-full-access` | 沙盒级别 |
| `-a untrusted / on-request / never` | 审批策略（另有 `on-failure`，源码已标 DEPRECATED） |
| `--search` | 启用实时 web search |
| `-i <FILE>` | 附加图片输入 |
| `-c key=value` | 临时覆盖配置 |

---

## CLI Slash 命令

| 命令 | 用途 | Claude Code 对应 |
|------|------|-----------------|
| `/plan` | 进入计划模式 | — |
| `/compact` | 压缩上下文 | `/compact` |
| `/diff` | 查看 Git diff | — |
| `/review` | 审查当前工作区 | — |
| `/init` | 生成 AGENTS.md 脚手架 | `/init` 生成 CLAUDE.md |
| `/mention` | 附加文件到对话 | `@file` |
| `/model` | 切换模型 | `/model` |
| `/status` | 查看模型/权限/上下文 | — |
| `/fork` | 分叉当前对话 | — |
| `/clear` | 清屏开新聊天 | — |

---

## 权限与沙盒

```bash
# 只读探索
codex -s read-only "解释这个仓库的构建流程"

# 日常开发
codex -s workspace-write "实现登录页错误提示，并运行相关测试"

# 自动化脚本
codex exec -a never -s workspace-write "格式化并运行单元测试"
```

> `--dangerously-bypass-approvals-and-sandbox` 只在外部已强隔离环境使用。

---

## 常用工作流

**修 bug：**
```
codex exec -s workspace-write -a on-request "复现并修复这个 bug：[报错]。先找根因，再做最小改动，最后运行相关测试"
```

**理解陌生项目：**
```
codex -s read-only "请阅读项目，不要改文件。输出：架构概览、关键入口、运行命令、测试命令、风险点"
```

**长会话续航：**
```
/compact 请保留当前任务目标、已改文件、失败测试、下一步计划
```

---

## 提示词模板

**通用模板（Claude Code 老用户习惯）：**
```
请先阅读相关文件，确认根因后再修改。
要求：
1. 改动尽量小。
2. 不要重构无关代码。
3. 修改前说明你要动哪些文件。
4. 修改后运行最相关的测试。
5. 最后总结改动和验证结果。
```

**明确成功标准：**
```
这个任务的成功标准是：
- 用户可以完成 X。
- Y 情况下不能回归。
- 必须通过 npm test -- login.spec.ts。

约束：
- 不要改数据库 schema。
- 不要引入新依赖。
```

---

## 相关

- [[topics/codex-migration-guide]] — 迁移心智模型与工程体系对比
- [[concepts/agentic-engineering]]
- [[concepts/context-files]]
