---
title: AI 驱动的 Git 工作流
type: topic
tags: [ai-coding, full-stack]
sources:
  - "raw/notion/tools/git-assistance.md"
created: 2026-05-18
updated: 2026-05-21
summary: "AI 协助 Git 的六层协作模式：让 AI 写 commit、创建 PR、做 review、跑自定义 Commands/Skills、连接 MCP——你是调度者，不是 Git 工程师。"
confidence: low
---

# AI 驱动的 Git 工作流

## 概述

传统 Git 学习路径要求开发者精通 50+ 命令、分支模型、rebase 冲突解决等。但在 AI 编程时代，开发者真正需要会的只是**7 个最小命令** + **如何指挥 AI 完成剩下的工作**。本主题梳理 AI 协助 Git 的六层协作模式，从手动调用 AI 写 commit 到 MCP 全自动化工作流。

核心论断：

> **你负责下指令、定规则；AI 负责写 commit、创建 PR、代码 review、执行 Git 命令。**
> **你是调度者，不是 Git 工程师。**

## 分析

### 一、最小必会清单（人类亲手敲的 7 条命令）

```bash
git status                # 当前状态
git branch                # 查看分支
git switch -c feature/xxx # 创建并切换分支
git add .                 # 暂存改动
git commit -m "..."       # 提交
git pull                  # 拉取远程更新
git push                  # 推送
```

剩下的（rebase、cherry-pick、reset、reflog、submodule 等）全部交给 AI。

更细的分支操作命令速查见 [[entities/git-branch]]，GitHub 平台术语见 [[entities/github-reference]]。

### 二、AI 协助 Git 的六个层级

**层级 1 —— 让 AI 自动生成 commit（基础）**

```
帮我提交代码并生成规范 commit message
```

AI 会自动：
- 分析 `git diff`
- 生成符合 Conventional Commits 规范的说明
- 执行 `git add` + `git commit`

**层级 2 —— 让 AI 创建 PR（含说明）**

```
帮我创建一个 PR 并写说明
```

AI 会自动：
- 总结改动（基于 `git log` 与 `git diff base...HEAD`）
- 生成 PR 描述（Summary + Test plan）
- 调用 `gh pr create`

**层级 3 —— 自动代码 Review**

```
review 当前改动
```

或在 GitHub PR 评论里 @ 自动化机器人：

```
@Claude review
```

**层级 4 —— 自定义 Slash Commands（强烈推荐）**

把高频 Git 流程沉淀成可复用命令：

```
.claude/commands/
├── smart-commit.md   # 智能 commit
├── git-quick.md      # 快速提交并推送
├── git-feature.md    # 创建 feature 分支
└── git-pr.md         # 推送并创建 PR
```

调用方式：`/smart-commit` `/git-quick` `/git-pr`。详见 [[concepts/agents]] 中关于 Commands 的说明。

**层级 5 —— Skills 定义 Git 规则（高级玩法）**

```
.claude/skills/git-rules/SKILL.md
```

在 SKILL.md 里定义：
- commit 规范（feat/fix/chore/refactor 前缀）
- 分支命名规范（feature/xxx、fix/xxx、release/x.y.z）
- PR 描述模板（Summary / Test plan / Risk）

Claude 此后会自动遵守你的 Git 规则，无需每次重复说明。Skills 与 Commands 的区别见 [[concepts/agentic-engineering]]。

**层级 6 —— MCP 连接 GitHub（进阶全自动化）**

通过 MCP（Model Context Protocol）让 AI 直连 GitHub API：
- 自动创建 / 关闭 issue
- 查询 PR 状态
- 自动 review
- 自动发版（tag + release notes）

### 三、推荐个人开发工作流

```
main
  ↓ 创建 feature 分支（不直接改 main）
feature/xxx
  ↓ 开发（让 AI 写代码、写测试）
  ↓ commit（让 AI 生成消息）
push
  ↓
创建 PR（让 AI 写描述）
  ↓
review（让 AI 自审）
  ↓
merge
```

即使是单人项目也建议走 PR 流程：
- **清晰历史**——每个 PR 是一次可审计的变更单元
- **安全**——主分支永远保持可部署
- **可回滚**——出问题时 revert 整个 PR 即可

### 四、四种 AI Git 自动化模板（按场景选择）

| 模板 | 适用场景 | 核心组件 |
|------|----------|----------|
| **极简个人用** | 一人项目、实验代码 | `/smart-commit` 一条命令搞定 |
| **规范型开发流程** | 团队协作 | Skills 定义规范 + Commands 自动化 |
| **自动发版系统** | 库 / SDK 类项目 | Commands + GitHub Actions + MCP 连发 release |
| **AI 全自动工作流** | 探索性项目 | Hooks 自动提交 + 多 Agent 编排 PR |

## 结论

AI 时代的 Git 能力评估方式发生了根本变化：

- **过去**：能否记住 50+ 命令、能否手动解决 rebase 冲突
- **现在**：能否设计可复用的 Commands/Skills、能否用清晰指令引导 AI 完成 Git 操作

Git 知识依然需要——但需要的是**心智模型**（仓库、分支、合并、远程的概念），而不是**命令记忆**。具体命令的执行下沉到 AI，开发者上移到"调度者"角色，把精力放在代码本身和工作流设计上。

## 相关实体与概念

- [[entities/git-branch]]
- [[entities/github-reference]]
- [[concepts/agentic-engineering]]
- [[concepts/agents]]

## 来源

- `raw/notion/tools/git-assistance.md`
