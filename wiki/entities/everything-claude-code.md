---
title: Everything Claude Code
type: entity
tags: [ai-coding]
sources: ["raw/Notion note/ExportBlock-e5803e1d-77c8-40c3-8740-733711539745-Part-1/📚 02-知识库/AI 技术栈/ClaudeCode/Everything Claude Code 使用指南 2fc81caa0df881839b61ddb79c82f131.md", "raw/Notion note/ExportBlock-e5803e1d-77c8-40c3-8740-733711539745-Part-1/📚 02-知识库/AI 技术栈/ClaudeCode/Everything Claude Code 使用指南/使用指南/Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充 2fd81caa0df88056a6a3d6c978c8136a.md"]
created: 2026-04-26
updated: 2026-04-26
summary: Anthropic 黑客马拉松冠军整理的完整 Claude Code 配置集合，覆盖跨会话记忆、持续学习、项目可维护性、子智能体编排四大核心功能。
---

# Everything Claude Code

Anthropic 黑客马拉松冠军整理的完整 Claude Code 配置集合，包含 Agents、Skills、Commands、Hooks、Rules 组件，覆盖跨会话记忆、持续学习、项目可维护性、子智能体编排四大核心功能。

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

- **自动**：Stop/SessionEnd/PostToolUse Hook 自动触发
- **手动**：`/learn` 命令提取当前会话可复用模式

### 3. 项目可维护性

- **检查点评估**（有明确里程碑）：`/checkpoint create` → 实现 → `/checkpoint verify`
- **持续评估**（探索性重构）：每 N 分钟自动运行测试套件
- **清理死代码**：`/refactor-clean`

### 4. 子智能体编排

```bash
/orchestrate custom "architect,tdd-guide,code-reviewer" "重设计缓存层"
```

## 常用命令

| 命令 | 用途 |
|---|---|
| `/plan` | 功能实现规划 |
| `/tdd` | 测试驱动开发 |
| `/python-review` | Python 专项审查 |
| `/learn` | 提取可复用模式 |
| `/checkpoint` | 保存/验证检查点 |
| `/refactor-clean` | 清理死代码 |

## 相关实体与概念

- [[concepts/agentic-engineering]]
- [[entities/claude-code-authors]]
- [[concepts/context-files]]
