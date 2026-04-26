---
title: Claude Code 最佳实践指南
type: topic
tags: [ai-coding, ai]
sources: [raw/claude-code-best-practice/README.md, raw/claude-code-best-practice/best-practice/claude-subagents.md, raw/claude-code-best-practice/best-practice/claude-skills.md, raw/claude-code-best-practice/best-practice/claude-commands.md, raw/claude-code-best-practice/best-practice/claude-settings.md, raw/claude-code-best-practice/best-practice/claude-memory.md]
created: 2026-04-26
updated: 2026-04-26
summary: 来自 Claude Code 创始人 Boris Cherny 与 Anthropic 团队的 82 条实战技巧，覆盖 Prompting/Context/Session/Agents/Skills/Hooks 全链路
---

# Claude Code 最佳实践指南

## 概述

本文整理自 [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 仓库，汇集 Claude Code 创始人 Boris Cherny、Anthropic 工程师 Thariq、Cat Wu 等人的一手经验，共 82 条实战技巧。

---

## Subagents（子代理）

### Frontmatter 核心字段（16 个）

| 字段 | 说明 |
|------|------|
| `name` | 唯一标识符，小写+连字符 |
| `description` | 触发时机，写 `"PROACTIVELY"` 可让 Claude 自动调用 |
| `tools` | 工具白名单，支持 `Agent(agent_type)` 语法 |
| `model` | `sonnet`/`opus`/`haiku`/`inherit`（默认 inherit） |
| `permissionMode` | `default`/`acceptEdits`/`auto`/`bypassPermissions`/`plan` |
| `maxTurns` | 最大 agentic 轮次 |
| `skills` | 启动时预加载的 Skill 列表（完整内容注入） |
| `hooks` | 作用域限于本 Agent 的生命周期钩子 |
| `isolation` | 设为 `"worktree"` 在临时 git worktree 中运行 |
| `effort` | 推理强度：`low`/`medium`/`high`/`max`（仅 Opus 4.6） |
| `background` | `true` 则始终作为后台任务运行 |
| `color` | 任务列表中的显示颜色 |

### 5 个官方内置 Agent

| Agent | 模型 | 工具 | 用途 |
|-------|------|------|------|
| `general-purpose` | inherit | 全部 | 复杂多步任务 |
| `Explore` | haiku | 只读 | 快速代码库探索 |
| `Plan` | inherit | 只读 | Plan 模式前期规划 |
| `statusline-setup` | sonnet | Read/Edit | 配置状态栏 |
| `claude-code-guide` | haiku | 搜索+Web | 回答 Claude Code 问题 |

---

## Skills（技能）

### Frontmatter 核心字段（15 个）

| 字段 | 说明 |
|------|------|
| `name` | 显示名称和 `/slash-command`，默认取目录名 |
| `description` | 触发条件（写给模型看，不是给人看） |
| `context` | 设为 `fork` 在隔离子代理中运行 |
| `agent` | `context: fork` 时使用的子代理类型 |
| `allowed-tools` | Skill 激活时免权限提示的工具 |
| `paths` | Glob 模式，限制 Skill 自动激活的文件范围 |
| `user-invocable` | `false` 则从 `/` 菜单隐藏，仅作后台知识 |
| `hooks` | 作用域限于本 Skill 的生命周期钩子 |

### 5 个官方内置 Skill

| Skill | 功能 |
|-------|------|
| `simplify` | 审查代码质量，消除重复 |
| `batch` | 批量跨文件执行命令 |
| `debug` | 调试失败命令或代码 |
| `loop` | 定时循环执行（最长 3 天） |
| `claude-api` | 构建 Claude API / Anthropic SDK 应用 |

---

## Commands（命令）

Slash 命令存放于 `.claude/commands/<name>.md`，Frontmatter 字段与 Skills 基本一致（15 个）。官方内置 75 个命令，按功能分组：

- **Auth**：`/login` `/logout` `/setup-bedrock` `/setup-vertex`
- **Config**：`/config` `/model` `/effort` `/permissions` `/sandbox`
- **Session**：`/clear` `/compact` `/rewind` `/resume` `/rename`
- **Context**：`/context` `/usage` `/compact`
- **Project**：`/init` `/review` `/ultrareview` `/security-review`
- **Remote**：`/schedule` `/remote-control` `/autofix-pr`
- **Debug**：`/doctor` `/tasks` `/powerup`

---

## Settings（配置）

配置文件 `settings.json` 支持 **60+ 设置项** 和 **175+ 环境变量**。

### 优先级层次（高→低）

1. Managed settings（组织强制，不可覆盖）
2. 命令行参数（单次会话）
3. `.claude/settings.local.json`（个人项目，git-ignored）
4. `.claude/settings.json`（团队共享）
5. `~/.claude/settings.json`（全局个人默认）

### 常用配置项

```json
{
  "model": "opus",
  "alwaysThinkingEnabled": true,
  "tui": "fullscreen",
  "permissions": {
    "allow": ["Edit(*)", "Write(*)", "Bash(npm run *)", "Bash(git *)"],
    "deny": ["Read(.env)"],
    "defaultMode": "acceptEdits"
  },
  "attribution": { "commit": "" }
}
```

### 权限语法

| 工具 | 示例 |
|------|------|
| `Bash` | `Bash(npm run *)`, `Bash(git *)` |
| `Edit` | `Edit(src/**)`, `Edit(*.ts)` |
| `Read` | `Read(.env)` |
| `Agent` | `Agent(*)` |
| `MCP` | `mcp__*` |

---

## Memory（记忆）

### CLAUDE.md 加载机制

- **向上加载（Ancestor）**：启动时自动加载当前目录到根目录的所有 CLAUDE.md
- **向下懒加载（Descendant）**：子目录的 CLAUDE.md 仅在访问该目录文件时才加载
- **兄弟目录不加载**：`frontend/` 不会加载 `backend/CLAUDE.md`

### 最佳实践

- 每个文件保持 **200 行以内**
- 用 `.claude/rules/*.md` 拆分大型指令（支持 `paths:` 懒加载）
- 用 `<important if="...">` 标签防止 Claude 忽略关键规则
- 用 `settings.json` 处理确定性行为（如 attribution），不要写进 CLAUDE.md

---

## 82 条实战技巧精选

### Prompting（提示词）
- 挑战 Claude："grill me on these changes and don't make a PR until I pass your test"
- 修复后追问："knowing everything you know now, scrap this and implement the elegant solution"
- 直接说 "fix"，不要微管理实现方式

### Context（上下文）管理
- 上下文超 **40%** 开始退化，超 **60%** 应考虑结束会话
- 1M 上下文模型约 **300-400k tokens** 后出现 context rot
- 优先用 `/compact focus on X` 而非等待自动压缩
- 用子代理隔离上下文：20 次文件读取 + 12 次 grep 留在子代理，只返回最终结论

### Session（会话）管理
- 每次 Claude 结束轮次后，在 Continue / `/rewind` / `/clear` / `/compact` / Subagent 中选择
- 新任务 = 新会话；相关任务可复用上下文
- `/rewind` 优于纠错：回到失败前重新提示，避免污染上下文

### Agents（子代理）
- 说 "use subagents" 让 Claude 投入更多算力
- 用 agent teams + tmux + git worktrees 并行开发
- 分离上下文窗口提升结果质量：一个 Agent 制造 bug，另一个（同模型）发现 bug

### Skills（技能）
- `description` 字段是触发器，不是摘要——写给模型看
- 每个 Skill 都要有 **Gotchas** 章节，记录 Claude 的失败点
- 用 `context: fork` 隔离运行，主上下文只看最终结果
- 在 SKILL.md 中用 `` !`command` `` 注入动态 shell 输出

### Hooks（钩子）
- `PostToolUse` hook 自动格式化代码
- `Stop` hook 在每轮结束时验证工作
- `PreToolUse` hook 将权限请求路由给 Opus 扫描安全性

### Git / PR
- PR 保持小而专注（Boris 的 p50 是 118 行）
- 始终 squash merge，保持线性历史
- 至少每小时提交一次

### 调试
- 遇到问题截图发给 Claude
- 用 MCP（Chrome/Playwright）让 Claude 自己看控制台日志
- 长时间运行的终端命令作为后台任务运行

---

## 结论

Claude Code 的核心使用哲学：

1. **不要保姆式监管**（🚫👶）——让 Claude 自主完成，只在关键节点介入
2. **上下文是稀缺资源**——主动管理，不要让它自然腐烂
3. **工具化重复工作**——每天做超过一次的事情，就做成 Skill 或 Command
4. **测试时间计算**——分离上下文窗口让同一模型既能制造又能发现问题

## 来源

- [[entities/claude-code-best-practice]]
