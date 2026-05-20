# Wiki Schema v2

本文件是 wiki/ 的规范说明书。AI 维护知识库时**必须严格遵循**。
设计参考：[Karpathy LLM Wiki](https://gist.github.com/karpathy) + [Hermes research-llm-wiki](https://hermes-agent.nousresearch.com/docs/user-guide/skills/bundled/research/research-llm-wiki) + 50+ 社区实现。

---

## 1. 领域

AI 前沿资讯与技术知识库，覆盖：

- prompt 工程
- AI Coding 工具链（Claude Code / Codex / Cursor / 各类 skill·agent·plugin）
- Spider 项目（AI 驱动爬虫与数据采集）
- 全栈技术
- 知识管理方法论（含本仓库自身使用的 Karpathy LLM Wiki 模式）

---

## 2. 三层架构

```
repository/
├── CLAUDE.md          # 项目级 AI 操作规范（Ingest / Query / Lint SOP）
├── raw/               # Layer 1：不可变源
├── wiki/              # Layer 2：agent 维护的结构化知识
│   ├── SCHEMA.md      # Layer 3：本文件，规则
│   ├── INDEX.md       # 全库目录
│   ├── log.md         # append-only 操作日志
│   ├── concepts/      # 概念页
│   ├── entities/      # 实体页（人 / 工具 / 产品 / 模型）
│   ├── topics/        # 主题深度分析
│   ├── comparisons/   # side-by-side 对比
│   ├── queries/       # 高价值问答归档
│   └── _archive/      # 过期内容（不进 INDEX）
└── outputs/           # 报告 / 答复
```

**职责边界**：

- `raw/`：只读不可改。AI 仅读取，永不修改。
- `wiki/`：AI 完全负责创建、更新、交叉引用、归档。
- `SCHEMA.md`：人定规则，AI 严格执行。

---

## 3. 文件命名

| 类型 | 路径格式 | 示例 |
|------|----------|------|
| 概念 | `concepts/<name>.md` | `concepts/prompt-engineering.md` |
| 实体 | `entities/<name>.md` | `entities/claude-code.md` |
| 主题 | `topics/<name>.md` | `topics/rag-optimization.md` |
| 对比 | `comparisons/<a>-vs-<b>.md` | `comparisons/claude-code-vs-codex.md` |
| 查询 | `queries/<YYYY-MM-DD>-<slug>.md` | `queries/2026-05-20-spider-stack.md` |
| 归档 | `_archive/<原路径>` | `_archive/entities/old-tool.md` |

规则：英文小写 + 连字符；禁止中文文件名、空格、大写字母（INDEX/SCHEMA/MEMORY/CLAUDE 例外）。

---

## 4. Frontmatter

### 4.1 wiki 页面（必需）

```yaml
---
title: 页面标题
type: concept | entity | topic | comparison | query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [仅限第 6 节定义的 tag]
sources: [raw/path/to/source.md, ...]
summary: 一句话摘要（用于 INDEX，≤ 80 字）
---
```

### 4.2 wiki 页面（质量信号，可选但强推荐）

```yaml
confidence: high | medium | low      # 多源验证强→high；单源主观→low
contested: true                      # 存在未解决的矛盾
contradictions: [page-slug-a, ...]   # 与哪些页面冲突
```

`confidence` 与 `contested` 是防止「弱证据沉淀为事实」的关键。Lint 会主动暴露 `confidence: low` 与 `contested: true` 的页面。

### 4.3 raw/ 文件（必需）

每个 raw 文件正文前加：

```yaml
---
source_url: https://...              # 原始 URL（如有）
ingested: YYYY-MM-DD
sha256: <body 的 SHA256，body 指 frontmatter 之后的内容>
---
```

`sha256` 用途：未来再次摄入同一 URL 时对比指纹，未变则跳过，已变则触发 drift 警报并更新。

---

## 5. Wikilink

- 使用 `[[path/filename]]`（含目录前缀）或 `[[filename]]`（Obsidian 自动解析）
- 每个新建/更新的 wiki 页**至少 2 个出向 wikilink**（避免孤立页）
- 同一页同一概念**只链一次**
- 提及但无对应 wiki 页的概念：若达到第 7 节阈值就建 stub；否则纯文本提及

---

## 6. Tag 分类法 v2

**新增 tag 必须先在此节注册再使用**。Lint 会拒绝未注册 tag。

### 领域 domain
- `ai` · `prompt-engineering` · `ai-coding` · `spider` · `full-stack` · `knowledge-management`

### 生态 ecosystem
- `claude-code` · `codex` · `cursor` · `obsidian` · `langchain` · `mcp` · `github`

### 形态 form
- `methodology` · `sop` · `case-study` · `benchmark` · `controversy` · `timeline` · `tooling`

### 状态 status
- `stub`（待完善）· `mature`（内容完整）· `contested`（存在争议）· `deprecated`（已过时）

---

## 7. 页面阈值（防止 stub 化与冗余化）

| 情况 | 处理 |
|------|------|
| 实体/概念在 **2+ 源**中出现 OR 是某源的核心 | 建独立页 |
| 单次路过提及 | 不建页，仅在相关页内纯文本提及 |
| 页面 > 200 行 | 拆分为子主题 + 双向交叉链接 |
| 内容被新页面完全覆盖 | 移入 `_archive/`，反链改为「纯文本（archived）」 |

---

## 8. 更新策略（处理矛盾）

新信息与已有内容冲突时：

1. **比较日期**：新源默认取代旧源，**但旧源若是权威一手来源**则保留
2. **真正矛盾**：双方都保留，各自标注日期 + 源
3. 在 frontmatter 加 `contradictions: [page-slug]` 双向标记
4. 在 Lint 报告中暴露，让用户判定

**禁止**：静默覆盖矛盾内容。

---

## 9. 归档

内容被完全取代或脱离领域范围时：

1. 移到 `_archive/<原路径>`（保留原结构）
2. 从 `INDEX.md` 移除对应条目
3. 更新所有反向链接：`[[old-page]]` → `old-page (archived)`（纯文本）
4. log.md 写入 `archive` 操作

---

## 10. 文章结构模板

### concept 页

```markdown
# 概念名称

## 定义
[一段话定义]

## 核心要点
- 要点 1
- 要点 2

## 相关概念
- [[相关概念]]

## 来源
- [[raw/path/to/source]]
```

### entity 页

```markdown
# 实体名称

## 简介
[一段话]

## 核心功能 / 贡献

## 关键事实与时间线

## 相关实体与概念

## 来源
```

### topic 页

```markdown
# 主题标题

## 概述（背景与问题）

## 分析（多角度展开）

## 结论

## 来源
```

### comparison 页

```markdown
# A vs B

## 比较对象与目的

## 维度对比（表格优先）
| 维度 | A | B |
|------|---|---|

## 综合判定

## 来源
```

### query 页

```markdown
# 问题：[原问题]

## 上下文（提问背景）

## 答复（综合多页 wiki）

## 引用页面
- [[page-a]]
- [[page-b]]
```

---

## 11. INDEX.md 规则

- 按类型分段：Concepts / Entities / Topics / Comparisons / Queries
- 每条一行：`- [[path/file]] — 一句话摘要`
- 段内字母序
- 单段超 50 条 → 按首字母拆子段
- 总条目超 200 → 创建 `_meta/topic-map.md` 按主题聚类

---

## 12. log.md 规则

- append-only，**绝不删除**历史
- 格式：`## [YYYY-MM-DD HH:MM] <action> | <subject>`
- action：`ingest` `update` `query` `lint` `create` `archive` `migrate` `delete`
- 超 500 条 → 轮换为 `log-YYYY.md`，本文件重置为空

---

## 13. 任务完成标准

AI 处理完一批 raw 后，**全部满足才算完成**：

1. 每个 raw 文件已被某 wiki 页 `sources:` 引用
2. 所有改动页 frontmatter 完整 + `updated` 已 bump
3. INDEX.md 对应分段已更新
4. log.md 已 append 本次操作
5. 每个新建/更新页至少 2 个出向 wikilink
6. 所有使用的 tag 在第 6 节注册
7. 文件名符合命名规范
