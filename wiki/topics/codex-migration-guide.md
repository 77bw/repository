---
title: 从 Claude Code 迁移到 Codex CLI / Codex App 使用手册
type: topic
tags: [ai-coding]
sources: [raw/docs/codex-migration-manual.html]
created: 2026-04-26
updated: 2026-05-18
summary: 面向 Claude Code 用户的 Codex CLI/App 完整迁移手册，含命令映射、权限配置、工作流与提示词模板
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
codex --full-auto "修复当前测试失败，保持改动最小"

# 非交互式（适合脚本/CI）
codex exec --cd /path/to/repo "运行测试并总结失败原因"
```

---

## 3. Claude Code → Codex 命令映射

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

## 4. CLI 命令总览

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
| `--full-auto` | 低摩擦预设：workspace-write + on-request |
| `-s read-only / workspace-write / danger-full-access` | 沙盒级别 |
| `-a untrusted / on-request / never` | 审批策略 |
| `--search` | 启用实时 web search |
| `-i <FILE>` | 附加图片输入 |
| `-c key=value` | 临时覆盖配置 |

---

## 5. CLI Slash 命令

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

## 6. 权限与沙盒

```bash
# 只读探索
codex -s read-only "解释这个仓库的构建流程"

# 日常开发
codex --full-auto "实现登录页错误提示，并运行相关测试"

# 自动化脚本
codex exec -a never -s workspace-write "格式化并运行单元测试"
```

> `--dangerously-bypass-approvals-and-sandbox` 只在外部已强隔离环境使用。

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

## 9. 常用工作流

**修 bug：**
```
codex --full-auto "复现并修复这个 bug：[报错]。先找根因，再做最小改动，最后运行相关测试"
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

## 10. 提示词模板

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

## 11. 与 Claude Code 工程体系对比

| 维度 | Claude Code | Codex |
|------|-------------|-------|
| 规则文件 | `CLAUDE.md` + `rules/` | `AGENTS.md` |
| 配置文件 | `.claude/settings.json` | `~/.codex/config.toml` |
| Skills | `/skill-name` | `$skill-name`（App）|
| 子智能体 | Agent tool + subagent_type | `/agent` + `codex fork` |
| 上下文压缩 | `/compact` | `/compact` |
| 记忆持久化 | Hooks（session-start/end） | 暂无内置，需 AGENTS.md 补充 |

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

- [[raw/docs/codex-migration-manual]]
