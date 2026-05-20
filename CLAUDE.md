# CLAUDE.md

本文件是 AI 在本仓库工作时的行为指引。设计模式：[Karpathy LLM Wiki](https://gist.github.com/karpathy) + [Hermes research-llm-wiki](https://hermes-agent.nousresearch.com/docs/user-guide/skills/bundled/research/research-llm-wiki)。

---

## 项目定位

**纯文档型 AI 知识库**。聚焦 prompt 工程、AI Coding 工具链、Spider 项目、全栈技术、知识管理方法论。
**不**是代码仓库，禁止任何可独立 build/run 的工程进入此库。

---

## 三层架构

| 层 | 路径 | 性质 | AI 权限 |
|----|------|------|---------|
| Layer 1 源 | `raw/` | 不可变原始素材 | 只读 |
| Layer 2 知识 | `wiki/{concepts,entities,topics,comparisons,queries}/` | 结构化页面 | 创建 / 更新 / 归档 |
| Layer 3 规则 | `wiki/SCHEMA.md` · `CLAUDE.md` | 规范定义 | 严格执行 |
| 报告 | `outputs/` | 一次性答复 | 创建 |

完整规则定义见 [wiki/SCHEMA.md](wiki/SCHEMA.md)。

---

## 会话定向（CRITICAL）

**任何 wiki 操作前必须先做这三步**，跳过会导致重复建页 / 漏交叉引用 / 违反规范：

1. 读 `wiki/SCHEMA.md` —— 弄清规则
2. 读 `wiki/INDEX.md` —— 知道有哪些页
3. 读 `wiki/log.md` 最近 30 行 —— 了解近期动作

仅做单页查询/小改动可省略 ②③；任何**摄入新源**或**lint** 必须完整三步。

---

## 三大核心操作

### Ingest（摄入新源）

用户提供新素材（URL / 文件 / 粘贴）时：

1. **落盘 raw/**：按性质放对应子目录（`articles/` `papers/` `transcripts/` `assets/`）；命名描述性（`raw/articles/karpathy-llm-wiki-2026.md`）；正文前加 frontmatter（`source_url` / `ingested` / `sha256`）
2. **讨论要点**：与用户对齐这份素材的关键点（自动场景跳过）
3. **查重**：先搜 INDEX，再 grep wiki，找已有相关页
4. **写/改 wiki**：守第 7 节阈值（2+ 源或核心才建页）；至少 2 个出向 wikilink；tag 在第 6 节注册；多源合成的页加 `^[raw/path/source.md]` provenance 标记
5. **设质量信号**：单源主观 → `confidence: low`；多源验证 → `high`；与现有页冲突 → `contested: true` + `contradictions:`
6. **更新 INDEX.md**：对应分段、字母序，bump `Last updated` 与 `Total pages`
7. **append log.md**：`## [YYYY-MM-DD HH:MM] ingest | <主题>` + 列出所有变动文件
8. **报告变更**：列出所有创建/更新的文件给用户

> 一份高质量源往往触发 5-15 个 wiki 页变化，这是**期望的复利效应**，不是过度行动。

### Query（查询）

用户在本仓库领域内提问时：

1. 读 INDEX 定位相关页
2. 100+ 页时再做一次 grep 全文搜索补充
3. 综合答复，引用 `[[页面]]` 来源
4. **高价值答复归档**：值得未来复用的对比 / 深度综合 → 写入 `queries/<YYYY-MM-DD>-<slug>.md`；琐碎查询不归档
5. append log.md：`## [...] query | <问题摘要>`（注明是否归档）

### Lint（健康检查）

用户要求审计/健康检查时，扫描以下问题（按严重度排序输出）：

1. **断链** —— `[[link]]` 指向不存在的页
2. **孤立页** —— 无任何入向 wikilink
3. **INDEX 与文件系统差异** —— 文件存在但未在索引；索引存在但文件已删
4. **frontmatter 缺失** —— 必填字段不全；tag 未在 SCHEMA 第 6 节注册
5. **源漂移** —— raw/ 文件 sha256 与 frontmatter 中记录不一致
6. **矛盾** —— `contested: true` / `contradictions:` 暴露给用户审阅
7. **质量低信号** —— `confidence: low` 或单源无 confidence 字段
8. **陈旧** —— `updated` 比相关源最新日期老 90+ 天
9. **超长页** —— > 200 行候选拆分
10. **log 轮换** —— 超 500 条 → `log-YYYY.md`

输出格式：分级清单 + 每条带具体路径 + 建议动作。完成后 append log.md。

---

## 禁止

- 修改或删除 `raw/` 下任何文件
- 删除 `wiki/` 下页面（可改、可移到 `_archive/`）
- 自创 tag（必须先到 SCHEMA 第 6 节注册）
- 跳过会话定向就动手摄入或 lint
- 静默覆盖矛盾内容（必须双方保留 + 标记）
- 在仓库内放可独立 build/run 的代码工程

---

## outputs/ 规范

- 命名：`YYYY-MM-DD-<topic>.md`
- 内容：问题 + 答复 + 引用的 wiki 条目列表

---

## 用户背景

- 后端工程师，熟悉 Python（uv 管理）
- 不熟悉前端（Vue / npm / Node.js）—— 涉及前端时给详细步骤+解释

## 关注方向

1. Prompt 工程（设计、优化、结构化输出）
2. AI Coding 工具深度使用（Claude Code / Codex 的 plugin / skill / agent）
3. Spider 项目（AI 驱动的爬虫、数据采集与解析）
4. 全栈技术（前后端 / API / 部署）
