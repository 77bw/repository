# Wiki Schema

本文件是 wiki/ 的规范说明书，AI 整理知识库时必须严格遵循。

---

## 目录结构

```
wiki/
├── INDEX.md          # 全库索引（必须保持最新）
├── SCHEMA.md         # 本文件，规范说明
├── concepts/         # 概念定义页
├── entities/         # 人物/工具/产品页
└── topics/           # 主题分析页
```

---

## 文件命名规范

| 类型 | 路径格式 | 示例 |
|------|----------|------|
| 概念 | `concepts/concept-name.md` | `concepts/prompt-engineering.md` |
| 实体 | `entities/entity-name.md` | `entities/claude-code.md` |
| 主题 | `topics/topic-description.md` | `topics/rag-optimization.md` |
| 索引 | `INDEX.md` | — |

规则：
- 统一英文小写 + 连字符，禁止中文文件名
- 禁止空格、大写字母（INDEX.md 和 SCHEMA.md 除外）

---

## Frontmatter 模板

每篇 wiki 文章开头必须包含：

```yaml
---
title: 文章标题
type: concept | entity | topic
tags: [tag1, tag2]
sources: [raw/来源文件路径]
created: YYYY-MM-DD
updated: YYYY-MM-DD
summary: 一句话摘要（用于 INDEX.md）
---
```

---

## Tag 分类

禁止随意新建顶级 tag，只使用以下预定义 tag：

**领域 tag**：
- `ai` — AI 通用
- `prompt-engineering` — 提示词工程
- `ai-coding` — AI 编程工具
- `spider` — 爬虫与数据采集
- `full-stack` — 全栈技术
- `knowledge-management` — 知识管理

**状态 tag**：
- `stub` — 待完善（内容不足）
- `mature` — 内容完整

---

## 文章结构模板

### concept 页
```
# 概念名称

[frontmatter]

## 定义
[一段话定义]

## 核心要点
- 要点1
- 要点2

## 相关概念
- [[相关概念1]]
- [[相关概念2]]

## 来源
- [[sources/来源文件]]
```

### entity 页（人物/工具/产品）
```
# 实体名称

[frontmatter]

## 简介
[一段话介绍]

## 核心功能/贡献
- 功能1
- 功能2

## 相关概念与实体
- [[相关概念]]
- [[相关实体]]

## 来源
- [[sources/来源文件]]
```

### topic 页（主题分析）
```
# 主题标题

[frontmatter]

## 概述
[背景与问题]

## 分析
[多角度展开]

## 结论
[核心观点]

## 来源
- [[sources/来源文件]]
```

---

## Wikilink 规则

- 首次提及已有 wiki 页的概念时使用 `[[路径/文件名]]`
- 同一文章中同一概念只链接一次
- 提及的概念若无 wiki 页，创建对应的 stub 文件

---

## INDEX.md 规则

每次新建或修改 wiki 文章后必须更新 INDEX.md。格式：

```markdown
| [[路径/文件名\|显示名称]] | 类型 | 一句话简介 |
```

---

## 任务完成标准

AI 处理一批 raw/ 文件后，以下条件全部满足才算完成：
1. 每个 raw 文件在 `sources/` 中有对应摘要（或在文章 frontmatter 的 sources 字段中引用）
2. 所有新建/修改的文章包含完整 frontmatter
3. INDEX.md 已更新，包含所有新条目
4. 文件名符合命名规范（英文小写 + 连字符）
