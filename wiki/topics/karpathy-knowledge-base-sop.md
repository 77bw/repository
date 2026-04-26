---
title: Karpathy 知识库 + Graphify 搭建与使用 SOP
type: topic
tags: [knowledge-management, ai-coding, ai]
sources: [raw/karpathy_note_create/把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md]
created: 2026-04-26
updated: 2026-04-26
summary: 基于 Karpathy 方法论，结合 Graphify 图谱检索工具，从零搭建可持续生长的个人 AI 知识库的完整 SOP
---

# Karpathy 知识库 + Graphify 搭建与使用 SOP

## 概述

本 SOP 基于 Andrej Karpathy 提出的"AI 图书管理员"方法论，结合 [[entities/graphify]] 图谱检索工具，形成一套可持续运转的个人知识库工作流。

**核心角色分工：**
- 你 = 采集者 + 提问者
- AI = 图书管理员（整理、分类、建立关联）
- Graphify = 检索加速器（知识库大了之后提升 AI 读取效率）

---

## 阶段一：初始化搭建（一次性）

### 1. 创建目录结构

```
my-knowledge-base/
  raw/       ← 原始素材（只进不改）
  wiki/      ← AI 整理后的结构化知识
  outputs/   ← AI 回答问题的报告
  CLAUDE.md  ← AI 的说明书
  wiki/SCHEMA.md ← wiki 编写规范
```

### 2. 编写 CLAUDE.md

告诉 AI 这个知识库的主题、规则和你关注的方向。关键内容：

```markdown
# 项目定义
[你的主题，如：AI 前沿资讯与技术知识库]

# 组织结构
- raw/ 包含未处理的原始素材。切勿修改此类文件。
- wiki/ 包含整理后的维基百科。完全由 AI 进行维护。
- outputs/ 包含生成的报告、回答和分析结果。

# wiki 的规则
- 每个主题在 wiki/ 目录下拥有独立的 .md 文件
- 使用 [[路径/文件名]] 格式链接相关主题
- 在 wiki/INDEX.md 维护全库索引

# 我关注的方向
[列出 3-5 个重点领域]
```

### 3. 编写 wiki/SCHEMA.md

定义 wiki 文章的 frontmatter 模板、目录结构、tag 分类、命名规范。这是 AI 整理时的强制约束。

---

## 阶段二：日常采集（持续进行）

### 采集方式

| 来源 | 操作 |
|------|------|
| 网页文章 | Obsidian Web Clipper 一键转 Markdown |
| 微信/论坛帖子 | 复制内容，新建 `.md` 文件存入 `raw/` |
| 个人笔记 | 直接写成 `.md` 扔进 `raw/` |

**原则：不整理、不分类，直接扔，保持原始状态。**

---

## 阶段三：AI 整理 wiki（每次有新素材后）

### 触发指令

```
先阅读 raw/ 中所有新增的原始内容，然后按照 CLAUDE.md 和 wiki/SCHEMA.md 的规则，
在 wiki/ 目录下更新 wiki。为每个主要主题创建或更新 .md 文件，
链接相关主题，并更新 INDEX.md。
```

### AI 完成标准（检查清单）

- [ ] 每个 raw 文件在某篇 wiki 文章的 `sources` 字段中被引用
- [ ] 所有新建/修改的文章包含完整 frontmatter
- [ ] `wiki/INDEX.md` 已更新，包含所有新条目
- [ ] 文件名符合命名规范（英文小写 + 连字符）

---

## 阶段四：Graphify 图谱生成（知识库 50+ 篇后启用）

### 为什么需要 Graphify

wiki 文章超过 50 篇后，AI 逐一读取文件会消耗大量 token，且难以发现跨文章的深层关联。Graphify 将整个 wiki 转成**关系图谱**，AI 读图谱比读文件更快、更省上下文。

### 使用方式

在 Claude Code 中执行：

```
/graphify
```

Graphify 会读取指定目录（如 `wiki/`），输出：
- `knowledge-graph.html` — 可视化关系图（节点 = 文章，连线 = 关联）
- `knowledge-graph.json` — 供 AI 查询的结构化数据
- 审计报告 — 发现孤立节点、缺失链接等问题

### 更新时机

每次 wiki 有较大更新后（新增 10+ 篇），重新运行一次 `/graphify`。

---

## 阶段五：提问与复利使用（持续进行）

### 基础提问模板

```
用知识库里的内容，给我写一篇关于 [主题] 的 500 字简报
```

### 场景化用法

| 场景 | 指令示例 |
|------|---------|
| 学习总结 | "总结知识库里关于 Claude Code 配置的核心技巧" |
| 写作辅助 | "基于 wiki，写一篇关于 Prompt 工程的文章大纲" |
| 决策支持 | "我要做爬虫项目，知识库里有哪些相关实践？" |
| 发现盲区 | "找出 wiki 里被提到但没有独立页面的话题" |
| 好答案沉淀 | 将有价值的回答存入 `outputs/YYYY-MM-DD-主题.md` |

---

## 阶段六：定期维护（每 1-2 周）

### 健康检查指令

```
检查整个 wiki/ 目录：
1. 找出页面之间有没有矛盾的说法
2. 找出被提到但没有独立页面的话题
3. 找出有哪些说法在 raw/ 里找不到来源
4. 建议 3 个新文章方向来填补空白
```

**重要：** 不要手动编辑 wiki/ 里的内容，让 AI 来修改，保持一致性。

---

## 完整工作流总览

```
[你发现好文章]
      ↓
[存入 raw/]
      ↓
[告诉 Claude Code 更新 wiki]
      ↓
[wiki 自动生长]
      ↓
[50篇后运行 /graphify 生成图谱]
      ↓
[随时提问，AI 用整个知识网络回答]
      ↓
[好答案存入 outputs/]
      ↓
[每 1-2 周健康检查]
```

## 来源

- [[raw/karpathy_note_create/把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy]]
