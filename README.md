# AI 技术知识库

Karpathy 风格的个人 AI 技术知识库。原始素材存入 `raw/`，由 Claude Code 按规范自动整理成结构化 `wiki/`，积累后用于深度问答和知识图谱探索。

聚焦方向：Prompt 工程 · AI Coding 工具 · Spider 项目 · 全栈技术

---

## 目录结构

| 路径 | 用途 | 维护者 |
|------|------|--------|
| `raw/` | 原始素材，只进不改 | 人工放入 |
| `wiki/` | 结构化知识库 | Claude Code |
| `wiki/INDEX.md` | 全库索引入口 | Claude Code |
| `wiki/SCHEMA.md` | wiki 规范说明书（命名/frontmatter/tag） | 人工维护 |
| `wiki/concepts/` | 概念定义页 | Claude Code |
| `wiki/entities/` | 人物/工具/产品页 | Claude Code |
| `wiki/topics/` | 主题分析页 | Claude Code |
| `outputs/` | 问答报告，按 `YYYY-MM-DD-主题.md` 命名 | Claude Code |
| `graphify-out/` | 知识图谱可视化 | graphify 工具 |

---

## 快速开始

**步骤 1：放入素材**

把文章、笔记、PDF 放入 `raw/`（推荐用 Obsidian Web Clipper 一键剪藏网页为 `.md`）。

**步骤 2：触发整理**

在 Claude Code 中说：
> "读取 raw/ 下的新文件，按 wiki/SCHEMA.md 规范整理到 wiki/，更新 INDEX.md"

**步骤 3：开始使用**

直接在 Claude Code 中提问，或打开 `graphify-out/graph.html` 浏览知识图谱。

---

## 工作流程 SOP

| 阶段 | 触发条件 | 操作 |
|------|----------|------|
| **存入** | 有新素材 | 放入 `raw/`，无需其他操作 |
| **构建** | raw/ 有新文件 | 说："处理 raw/ 下的新文件，按 SCHEMA.md 整理，更新 INDEX.md" |
| **使用** | 随时 | 直接提问，或用 graphify 查询 |
| **维护** | 每新增 10 篇后 | 说："对 wiki/ 做健康检查：找孤立文件、死链、可合并条目，建议新方向" |

---

## 常用提示词速查

| 场景 | 提示词 |
|------|--------|
| 处理新素材 | `读取 raw/ 下的新文件，按 wiki/SCHEMA.md 整理，更新 INDEX.md` |
| 解释概念 | `基于 wiki/ 解释 [概念]，列出相关 wikilink` |
| 对比分析 | `对比 wiki/ 中 [A] 和 [B] 的异同` |
| 保存回答 | `将这个回答保存到 outputs/YYYY-MM-DD-[主题].md` |
| 健康检查 | `检查 wiki/ 中的孤立文件、死链、可合并条目，建议新方向` |
| 更新图谱 | `graphify ./wiki --update` 然后打开 `graphify-out/graph.html` |

---

## FAQ

**1. raw/ 支持什么格式？**
`.md`、`.txt`、`.pdf`、图片均可。Claude Code 直接读取文本类文件；PDF 建议先用 `brew install poppler` 安装依赖后再处理。

**2. wiki/ 文件我能手动编辑吗？**
可以，但请保持 frontmatter 完整，避免破坏 SCHEMA.md 规范。AI 下次更新时会保留你的修改。

**3. SCHEMA.md 和 CLAUDE.md 有什么区别？**
`CLAUDE.md` 是给 AI 的项目级指令（做什么、不做什么）；`SCHEMA.md` 是 wiki 的格式规范（怎么写文件）。两者配合使用。

**4. 怎么知道哪些 raw/ 文件还没被处理？**
问 Claude Code："检查 raw/ 下哪些文件在 wiki 文章的 sources 字段中没有被引用？"

**5. outputs/ 和 wiki/ 有什么区别？**
`wiki/` 是长期积累的常青知识条目；`outputs/` 是某次具体问答的结果（有时效性）。好的 outputs 可以手动提炼后移入 wiki。

**6. 知识图谱怎么更新？**
wiki 有较大更新后运行 `graphify ./wiki --update`，重新打开 `graphify-out/graph.html`。

**7. 需要向量数据库吗？**
不需要。wiki 总量在 400K 字符以内时，Claude Code 直接读取全部文件即可，无需额外工具。

**8. 现有的 articles/tools/resources 目录怎么处理？**
暂时保留，下次处理新素材时顺带迁移到 `concepts/`、`entities/`、`topics/`。不需要一次性迁移。

**9. 如何搜索特定主题？**
三种方式：① 直接问 Claude Code；② 在 Obsidian 中用全文搜索（Vault 路径设为 `wiki/`）；③ 用 `graphify query "关键词"`。

**10. 健康检查多久做一次？**
建议每新增 10 篇素材后做一次，或每月一次。触发语："对 wiki/ 做健康检查"。

---

## Obsidian 配置

将 Vault 路径设为 `wiki/`，推荐插件：
- **Dataview**：用 SQL 风格查询 frontmatter 字段
- **Graph View**：可视化 wikilink 关系
