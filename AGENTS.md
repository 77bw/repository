# AGENTS.md

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
| Layer 3 规则 | `wiki/SCHEMA.md` · `AGENTS.md` | 规范定义 | 严格执行 |
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
2. AI Coding 工具深度使用（Codex / Codex 的 plugin / skill / agent）
3. Spider 项目（AI 驱动的爬虫、数据采集与解析）
4. 全栈技术（前后端 / API / 部署）


<claude-mem-context>
# Memory Context

# [repository] recent context, 2026-06-01 1:34am GMT+8

Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note
Format: ID TIME TYPE TITLE
Fetch details: get_observations([IDs]) | Search: mem-search skill

Stats: 50 obs (20,667t read) | 630,581t work | 97% savings

### May 21, 2026
586 1:39a 🔄 Consolidated Codex migration guide to strategic overview with linked references
S198 知识库规则重构与全库反链补齐：从 Hermes/Karpathy 标准出发，重新组织 wiki 结构，分离战略指南与操作参考，并建立页面间的语义关联 (May 21 at 1:39 AM)
588 " ✅ Reorganized cross-references in CLAUDE.md mechanisms topic
589 1:40a ✅ Consolidated cross-references across multiple wiki topics
590 " 🔄 Compressed Agent Team architecture topic to meet size constraint
591 " ✅ Removed trailing separator to achieve 200-line target
592 " 🔵 Knowledge base restructuring completed with all topics meeting 200-line constraint
593 1:41a ✅ Updated wiki INDEX.md to reflect new page count after restructuring
594 " ✅ Updated wiki INDEX.md with new topics and refined descriptions
595 " 🔵 Final verification of knowledge base restructuring results
596 1:42a 🔵 Launched background task to complete wiki backlinks and eliminate isolated pages
S200 Complete P2 phase verification and finalization for Karpathy LLM Wiki v2: validate file name normalization (32 raw files), SCHEMA repairs + sidecar metadata (3 files), page splitting (4→8 pages), and backlink completion (28 pages all linked). Execute final commit and push. (May 21 at 1:42 AM)
597 " 🔵 Background task analyzing wiki page interconnections and wikilink distribution
S202 Plan user-level graphify cleanup after project-level removal, with execution approval workflow (May 21 at 1:42 AM)
609 1:56a 🔵 User-level graphify artifacts inventory
610 " 🔵 Graphify cleanup scope detailed
611 " 🔵 Project-level graphify references mapped
S203 Clean up user-level graphify artifacts after project-level removal, with plan review and execution (May 21 at 1:57 AM)
613 1:58a ✅ Graphify skill directory deleted
614 " ✅ Graphify section removed from user CLAUDE.md
S208 Remove deprecated /graphify skill references from knowledge base documentation (README and wiki) to prevent user confusion; preserve Karpathy knowledge management methodology (May 21 at 1:58 AM)
S260 Delete misleading wechat-video-content-summary.md source article and clean up all derived wiki content that promoted the "compress CLAUDE.md to 60 lines" methodology (May 21 at 10:33 PM)
### May 28, 2026
903 12:53p 🔵 Contradictory CLAUDE.md guidance in wiki sourced from single WeChat video
904 12:54p 🔵 Wiki dependency mapping: wechat-video source influences two CLAUDE.md-related topics
905 12:55p 🔵 Explicit contradiction in claude-md-mechanisms.md conclusion vs wechat-video source claim
906 " 🔵 INDEX.md stale timestamp and incomplete contradiction documentation
907 " 🔵 Wiki log.md shows schema v2 migration but contradictions not yet recorded
908 12:56p ⚖️ User intent: Delete wechat-video source and remediate downstream wiki pages
909 " 🔵 Wechat-video source is single-point dependency with no cross-references in raw/ layer
910 " 🔵 Wechat-video source file is tracked in git and ready for deletion
911 12:57p ⚖️ Detailed remediation plan generated for wechat-video source deletion and wiki cleanup
913 1:02p ⚖️ Revised plan to rewrite rather than delete "60-line simplification" section
914 " ⚖️ Refined design decision: preserve methodology while removing prescriptive 60-line target
915 1:03p ⚖️ Plan approved and ready for execution: wiki cleanup with source removal and methodology preservation
916 " ✅ Deleted problematic source article about 60-line CLAUDE.md compression
917 1:04p ✅ Updated claude-md-best-practices.md frontmatter to remove wechat source and adjust confidence
918 " ✅ Updated main heading and INDEX.md entry to remove "60行精简法" references
919 " ✅ Updated overview section to replace "60行精简法" with "组织方法"
920 1:05p ✅ Rewrote section to remove 60-line compression target and Ali 厂长 attribution
921 " ✅ Cleaned up claude-md-mechanisms.md to remove wechat source and 60-line references
922 " ✅ Appended cleanup operation record to wiki log
923 " 🔵 Verification complete: all misleading references removed from wiki content
S261 Delete misleading wechat-video-content-summary.md source article about compressing CLAUDE.md to 60 lines, and clean up all derived wiki content promoting this flawed methodology (May 28 at 1:06 PM)
924 1:07p 🔵 Git status shows cleanup changes staged and ready for commit
925 " ✅ Cleanup changes committed to master branch
926 " ✅ Cleanup changes pushed to remote repository
S305 Optimize wiki documentation by auditing all pages for incorrect, outdated, or conflicting content; produce a prioritized modification recommendation document without making direct changes. (May 28 at 1:07 PM)
### Jun 1, 2026
1461 12:52a 🔵 Wiki topics directory contains 17 markdown documents
S306 Generate optimized prompt package for wiki audit task; design two-phase workflow (Phase 1: audit recommendations only, Phase 2: execute changes after approval). (Jun 1 at 12:59 AM)
1462 1:03a ⚖️ Wiki Audit Workflow Architecture: Four-Phase Pipeline with Model Tiering
S307 Wiki 全库内容审计 — 扫描 28 页 wiki，找出过时/错误/与官方冲突的内容，生成分级建议文档 (Jun 1 at 1:03 AM)
1463 " 🔵 Wiki Inventory Complete: 31 Pages Across 5 Concept + 6 Entity + 19 Topic Categories
1464 1:04a 🔵 Wiki Content Audit: 31 Pages Sampled, 3016 Total Lines, Confidence/Update Patterns Identified
1465 " 🔵 Wiki Inventory Extraction Complete: 28 INDEX Pages vs 28 Actual Files, All Tags Registered
1466 " 🔵 Phase 2 Mechanical Check: 4 Broken Wikilinks Detected, 0 Orphaned Pages, Perfect INDEX Alignment
1467 1:05a 🔵 Phase 2 Mechanical Check Complete: INDEX Contains Wikilinks to SCHEMA and Wikilink Concept (Not Files)
1468 " 🔵 Phase 2 Mechanical Check Final: 1 Broken Link (Placeholder Text), 0 Orphans, 0 Frontmatter Issues, 0 Long Pages
1469 " 🔵 Broken Link Root Cause: Placeholder Text in karpathy-knowledge-base-sop.md Line 52
1470 1:06a 🔵 Phase 2 Mechanical Check FINAL: 0 Actionable Broken Links (All Excluded: Anchors, Raw References, Placeholders)
1471 " 🔵 Phase 3 Content Audit Ready: Official Documentation Verification Complete

Access 631k tokens of past work via get_observations([IDs]) or mem-search skill.
</claude-mem-context>