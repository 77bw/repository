---
title: Everything Claude Code
type: entity
tags: [ai-coding]
sources:
  - "raw/notion/tech-stack/everything-claude-code-guide.md"
  - "raw/notion/tech-stack/anthropic-hackathon-claude-code-config.md"
  - "raw/notion/tech-stack/everything-claude-code-guide-raw.md"
  - "raw/notion/tech-stack/usage-guide.md"
created: 2026-04-26
updated: 2026-05-21
summary: Anthropic 黑客马拉松冠军 affaan-m 整理的 Claude Code 配置集合，覆盖跨会话记忆、持续学习、项目可维护性、子智能体编排，提供 PLAN→CHECKPOINT→TDD→VERIFY→REVIEW→LEARN→RECORD 七步标准开发循环。
confidence: high
---

# Everything Claude Code

[everything-claude-code](https://github.com/affaan-m/everything-claude-code) 是 Anthropic 黑客马拉松冠军 affaan-m 整理的完整 Claude Code 配置集合，包含 Agents、Skills、Commands、Hooks、Rules 五类组件，围绕 Memory Bank 模式构建复杂软件项目的可复用工作流。

## 安装

```bash
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code

# 手动复制规则（必需）
git clone https://github.com/affaan-m/everything-claude-code.git
cp -r everything-claude-code/rules/* ~/.claude/rules/
```

## 组件概览

| 组件 | 自动安装 | 数量 |
|---|---|---|
| Agents | ✅ | 13 个 |
| Skills | ✅ | 28 个 |
| Commands | ✅ | 24 个 |
| Rules | ❌ 需手动复制 | 8 个 |

## 四大核心功能

### 1. 跨会话共享内存

通过三个 Hook 解决会话压缩导致关键信息丢失的问题：

- `session-start.js`：新会话启动时自动加载上次状态
- `pre-compact.js`：压缩前将重要状态保存到文件
- `session-end.js`：会话结束时持久化学习成果

### 2. 持续学习

- **自动**：Stop / SessionEnd / PostToolUse Hook 自动触发
- **手动**：`/learn` 命令提取当前会话可复用模式

### 3. 项目可维护性

- **检查点评估**（有明确里程碑）：`/checkpoint create` → 实现 → `/checkpoint verify`
- **持续评估**（探索性重构）：每 N 分钟自动运行测试套件
- **清理死代码**：`/refactor-clean`

### 4. 子智能体编排

```bash
/orchestrate custom "architect,tdd-guide,code-reviewer" "重设计缓存层"
```

## 导师提示词（启动新项目时使用）

将以下提示词发送给 Claude，让它扮演 everything-claude-code 作者角色进入"导师模式"：

```markdown
# Role
你现在是 Anthropic Hackathon 获胜者，也是 GitHub 项目 everything-claude-code 的作者。
你深刻理解如何通过 Claude Code 结合 MCP 工具，以 Memory Bank 模式构建复杂软件。

# Context & Knowledge Base
1. GitHub 项目: https://github.com/affaan-m/everything-claude-code
2. 参考文档: .claude/reference/Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充.md

# User Status
我已经完成以下配置（请不要重复教学安装步骤）：
1. 已通过插件方式安装 everything-claude-code
2. 已执行 cp -r everything-claude-code/rules/* ~/.claude/rules
3. 项目根目录下已初始化 memory-bank 文件夹（productContext.md, activeContext.md 等）

# Task
请以"导师"身份，教我如何开始构建项目。回答时：
1. 给出具体可执行的 Prompt 范例
2. 解释每个步骤的目的
3. 使用规范的命令格式（带 /everything-claude-code: 前缀）
```

## 标准 7 步开发循环

```
PLAN  →  CHECKPOINT  →  TDD  →  VERIFY  →  REVIEW  →  LEARN  →  RECORD
 ↓          ↓           ↓         ↓          ↓          ↓          ↓
规划      保存初态    测试先行   验证变更   代码审查   提取模式   更新 MB
                                                              + commit
```

| 步骤 | 命令 | 说明 |
|------|------|------|
| 1. PLAN | `/everything-claude-code:plan` | 输出实现方案，等待用户确认 |
| 2. CHECKPOINT | `/everything-claude-code:checkpoint create` | 保存初始状态 |
| 3. TDD | `/everything-claude-code:tdd` | 测试先行，逐模块实现 |
| 4. VERIFY | `/everything-claude-code:checkpoint verify` | 验证所有变更 |
| 5. REVIEW | `/everything-claude-code:python-review` / `go-review` | 语言专项审查 |
| 6. LEARN | `/everything-claude-code:learn` | 提取可复用模式 |
| 7. RECORD | 更新 `memory-bank/` + `git commit` | 记录进度并提交 |

**两种执行模式：**

- **小步快跑**（学习/调试）：每个 phase 完成后逐项验证，更新 `todo.md` + `progress.md`，再创建 commit
- **一步到位**（批量开发）：每完成一个模块立即 commit，全部完成后统一更新 memory-bank，目标测试覆盖率 60%+

## Memory Bank 更新规则

| 文件 | 更新时机 | 内容 |
|------|----------|------|
| `todo.md` | 任务完成后 | 标记 ✅，添加新任务 |
| `progress.md` | Phase 完成后 | 记录里程碑、commit SHA、关键特性 |
| `architecture.md` | 新增 / 删除文件后 | 更新项目结构说明 |
| `PRD.md` | 需求变更时 | 更新需求定义 |

## 常用命令速查

**插件命令（需前缀 `/everything-claude-code:`）：**

| 命令 | 用途 | 使用时机 |
|------|------|----------|
| `plan` | 规划实现方案 | 开始新功能前 |
| `checkpoint create` | 保存当前状态 | 写代码前 |
| `checkpoint verify` | 验证变更 | 写代码后 |
| `tdd` | 测试驱动开发 | 实现功能时 |
| `python-review` / `go-review` | 语言专项审查 | 代码完成后 |
| `learn` | 保存学习记录 | 发现好模式时 |
| `build-fix` | 修复构建错误 | 构建失败时 |
| `refactor-clean` | 清理冗余代码 | 代码维护时 |
| `verify quick` | 快速验证 | 快速检查时 |
| `e2e` | E2E 测试 | 端到端测试时 |
| `orchestrate` | 编排子智能体 | 复杂任务时 |

**原生命令（无需前缀）：** `/clear` `/compact` `/help` `/cost` `/resume`

## Hooks 自动化

| Hook | 触发时机 | 功能 |
|------|----------|------|
| `session-start.js` | 会话开始 | 自动加载上次会话状态 |
| `session-end.js` | 会话结束 | 保存会话状态 |
| `pre-compact.js` | 上下文压缩前 | 保存重要信息 |
| `PostToolUse` | 工具使用后 | 自动格式化、类型检查 |

## 知识沉淀体系（两种类型互补）

| 类型 | 级别 | 位置 | 目的 |
|------|------|------|------|
| **学习记录** Learn Skills | 用户级 | `~/.claude/skills/learned/` | 跨项目复用的通用模式（如 Pydantic 单例） |
| **项目状态** Memory Bank | 项目级 | `./memory-bank/` | 项目特定的工作状态（如 Phase 完成情况） |

而 [[concepts/claude-md]] 作为第三类——**项目约束规则**——存放在 `./CLAUDE.md`，与上述两类形成三角互补。

## 最佳实践

1. **写代码前必读 memory-bank** —— 确保了解项目上下文
2. **每次执行前 `checkpoint create`** —— 保存初始状态
3. **每次完成后 `checkpoint verify`** —— 验证变更
4. **Phase 完成后更新 `progress.md`** —— 记录里程碑
5. **发现好模式就 `/learn`** —— 沉淀可复用知识
6. **始终使用 TDD 模式** —— 测试先行，保证质量
7. **代码审查不能少** —— `/python-review` 或 `/go-review`

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 命令报错 "Skill is not a prompt-based skill" | 使用 `Task` 工具或 `EnterPlanMode` 替代直接调用 |
| 忘记命令前缀 | 所有命令必须加 `/everything-claude-code:` 前缀 |
| 上下文过长 | 使用 `/compact` 压缩，或 `/clear` 清除后重新开始 |

## 相关实体与概念

- [[concepts/agentic-engineering]]
- [[concepts/context-files]]
- [[concepts/claude-md]]
- [[topics/vibe-coding]]
- [[topics/agent-team-architecture]]
