---
title: CLAUDE.md 最佳实践
type: topic
tags: [ai-coding, ai, knowledge-management]
sources:
  - raw/ai-chat/claude_md_complete_guide.md
created: 2026-05-21
updated: 2026-06-01
summary: CLAUDE.md 写作规范、组织方法、两种机制选择、推荐目录结构、调试方法与 Auto Memory 互补关系。
confidence: medium
---

# CLAUDE.md 最佳实践

## 概述

本文从 [[topics/claude-md-mechanisms]] 中拆分，聚焦 CLAUDE.md 的实操最佳实践：写作规范、组织方法、机制选择决策树、推荐目录结构、调试方法，以及与 Auto Memory 的互补关系。机制原理（文件层级、加载机制、rules 路径作用域、渐进式披露指令）见父页面。

## 写作规范

- **大小**：每个 CLAUDE.md 控制在 200 行以内；超出后 Claude 遵从度下降
- **结构**：用 Markdown headers 和 bullet 分组，结构越清晰遵从越好
- **具体性**：写可验证的指令（✅ "使用 2 空格缩进" / ❌ "格式化好代码"）
- **HTML 注释**：块级 `<!-- ... -->` 注释注入前被去除（可写给人看的说明而不消耗 token）；代码块内的注释会保留；Read 工具直接打开文件时注释仍可见

## CLAUDE.md 组织方法

当 CLAUDE.md 内容膨胀导致维护困难或遵从度下降时，可按以下方法拆分组织：

1. **根文件做薄**——主 CLAUDE.md 只保留真正全局的约束（命名规范、提交规范、文档地图），任何带"如果……则……"的条件性规则都迁出。
2. **规则按关注点拆 `.claude/rules/`**——按文件类型或模块拆分（`typescript.md`、`testing.md`、`sql.md`、`frontend/react.md`），配合 `paths` 或 `globs` frontmatter 触发（注意 `paths:` 存在已知 bug——issue #16038/#17204 报告字符串与数组语法静默失败，建议用 `globs:` 或 YAML 数组形式），根文件不再背负这些规则。
3. **用 `/memory` 命令逐条验收**——每删一条规则，运行 `/memory` 确认当前 session 加载链路仍完整；保留即"删了就不工作"的规则，删除即"保留也无人遵守"的规则。

## 两种机制的选择

```
规则是"针对某种文件类型的编写规范"
  → .claude/rules/ + paths（确定性触发）
  例：TypeScript 规范、测试文件规范、SQL 规范

规则是"针对某个模块/功能域的背景知识"
  → 子目录 CLAUDE.md（懒加载）
  例：API 模块的设计背景、前端组件规范

规则是"任何时候都要遵守的全局约定"
  → 根 CLAUDE.md 或无 paths 的 rules 文件
  例：代码提交规范、命名约定

规则是"某些场景下才需要的业务/架构文档"
  → CLAUDE.md 自然语言指令（渐进式披露）
  例：项目规格、架构设计、API 合约
```

## 推荐完整目录结构

```
project/
├── CLAUDE.md                    ← 全局：架构总览、build 命令、文档地图
├── CLAUDE.local.md              ← 个人：沙箱 URL、测试数据（gitignore）
├── src/
│   ├── api/
│   │   └── CLAUDE.md            ← 模块级：API 设计背景（懒加载）
│   └── frontend/
│       └── CLAUDE.md            ← 模块级：前端组件规范（懒加载）
└── .claude/
    ├── CLAUDE.md                ← 补充项目级说明
    ├── settings.json
    ├── settings.local.json      ← 个人配置（gitignore）
    ├── rules/
    │   ├── typescript.md        ← globs: **/*.ts
    │   ├── testing.md           ← globs: **/*.test.*
    │   ├── sql.md               ← globs: **/*.sql
    │   └── global-style.md      ← 无 globs：全局风格
    ├── commands/
    ├── agents/
    └── skills/
```

> ⚠️ 路径作用域字段建议用 `globs:`：GitHub issue #16038/#17204 报告 `paths:` 的字符串与数组语法存在静默失败，`globs:`（或等价的 YAML 数组形式）触发更可靠。

## 调试方法

| 问题 | 解决方案 |
|------|----------|
| 不知道哪些文件已加载 | 运行 `/memory` 列出当前 session 已加载的 CLAUDE.md、CLAUDE.local.md 和 rules 文件 |
| 子目录规则没生效 | 先让 Claude 读该目录的一个文件，再试 |
| /compact 后指令消失 | 检查是否在根 CLAUDE.md 中，子目录需重新触发 |
| 规则被其他团队的 CLAUDE.md 干扰 | 在 `.claude/settings.local.json` 配置 `claudeMdExcludes` |
| 想精确追踪加载时机 | 使用 `InstructionsLoaded` hook 记录日志（仅观察性，不可阻止加载） |

## Auto Memory 与 CLAUDE.md 的互补关系

| | CLAUDE.md | Auto Memory |
|---|---|---|
| 谁写 | 开发者 | Claude 自己 |
| 内容 | 规则和指令 | 学习到的模式和偏好 |
| 存储位置 | 项目仓库 | `~/.claude/projects/<project>/memory/` |
| 每次 session 加载 | 项目根及以上全量；子目录懒加载 | MEMORY.md 前 200 行或 25KB |

通过 `/memory` 命令可查看和管理 Auto Memory 文件。

## 相关

- [[topics/claude-md-mechanisms]] — 机制原理（文件层级、加载、rules、渐进式披露）
- [[concepts/claude-md]]
- [[concepts/context-files]]
