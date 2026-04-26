# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 项目定义

AI 前沿资讯与技术知识库，聚焦 AI 文章、AI Coding、AI 使用技巧、Spider-AI 探索实践。

# 组织结构

- `raw/` 包含未处理的原始素材。切勿修改此类文件。
- `wiki/` 包含整理后的维基百科。完全由 AI 进行维护。
- `outputs/` 包含生成的报告、回答和分析结果。

# wiki 的规则

- 整理 wiki 时必须严格遵循 `wiki/SCHEMA.md` 中的规范。
- 每个主题在 `wiki/` 目录下拥有独立的 `.md` 文件，文件名使用英文小写 + 连字符。
- 每篇 wiki 文章开头必须包含完整的 YAML frontmatter（见 SCHEMA.md 模板）。
- 使用 `[[路径/文件名]]` 格式链接相关主题，同一文章中同一概念只链接一次。
- 在 `wiki/INDEX.md` 维护全库索引，每次新建或修改文章后必须更新。
- 当有新的原始素材加入时，须同步更新相关的 wiki 文章。

# 禁止行为

- 禁止修改或删除 `raw/` 下的任何文件。
- 禁止删除 `wiki/` 下的已有文件（可更新内容）。
- 禁止随意新建 tag，只使用 SCHEMA.md 中预定义的 tag。

# 任务完成标准

处理一批 raw/ 文件后，以下条件全部满足才算完成：
1. 每个 raw 文件在 wiki 文章的 frontmatter sources 字段中被引用。
2. 所有新建/修改的文章包含完整 frontmatter。
3. `wiki/INDEX.md` 已更新，包含所有新条目。

# outputs 规范

- 文件命名格式：`YYYY-MM-DD-主题.md`
- 内容格式：问题 + 回答 + 参考的 wiki 条目列表

# 用户背景

- 后端开发者，熟悉 Python
- 不熟悉前端（Vue、npm、Node.js 等），前端相关操作需给出详细步骤和解释

# 我关注的方向

1. Prompt 工程（提示词设计、优化、结构化输出）
2. AI Coding 工具深度使用（Claude Code、Codex 的插件、Skill、Agent 搭配技巧）
3. Spider 项目（AI 驱动的爬虫、数据采集与解析）
4. 全栈技术（前后端、API 设计、部署）
