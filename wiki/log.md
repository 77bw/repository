# Wiki Log

> 所有 wiki 操作的时间线记录。append-only，绝不删除历史条目。
> 格式：`## [YYYY-MM-DD HH:MM] action | subject`
> action 取值：ingest / update / query / lint / create / archive / migrate / delete
> 当本文件超过 500 条时轮换：重命名为 `log-YYYY.md`，本文件重置为空。

## [2026-05-20 23:35] migrate | wiki schema v1 → v2

参考 [Hermes research-llm-wiki SKILL.md](https://hermes-agent.nousresearch.com/docs/user-guide/skills/bundled/research/research-llm-wiki) 与 GitHub 50+ Karpathy LLM Wiki 实现，完成知识库结构升级。

**清理：**
- 删除 `raw/claude-code-best-practice/`（克隆工程，与纯文档定位不符）
- 删除 `graphify-out/`（短期不再主动跑 graphify，加入 .gitignore）
- 删除 4 处 .DS_Store

**新增：**
- `wiki/log.md`（本文件）
- `wiki/comparisons/`（side-by-side 分析）
- `wiki/queries/`（高价值问答归档）
- `wiki/_archive/`（已过期/被取代的内容）

**重写：**
- `wiki/SCHEMA.md`：新增 confidence/contested/contradictions、raw/ frontmatter 含 sha256、页面阈值、归档策略、tag taxonomy v2
- `CLAUDE.md`：突出三大核心操作（Ingest / Query / Lint）+ 会话定向流程

**保留决策：**
- `topics/` 类型保留（中文知识库主题分析高频用法）
- 不启用 obsidian-headless（用户用 git 同步）
- 不在 CLAUDE.md 中触发 graphify（保留 skill 但不主动跑）

## [2026-05-21 01:00] migrate | P1 阶段：raw frontmatter + wiki confidence + lint baseline

并行派遣 Agent-B（Lint）/ Agent-C（raw frontmatter）/ Agent-D（wiki confidence），独立 Review 代理终审 PASS。

**产出：**
- 30 个 raw/.md 添加 frontmatter（source_url / ingested / sha256）；7 个保留已有 fm 跳过；3 个非 .md（pdf + 2 html）跳过待 sidecar 方案
- 24 篇 wiki 页补 `confidence` 字段（high 6 / medium 14 / low 4）并 bump `updated: 2026-05-21`
- 全库 lint 基线建立：3 断链 / 24 孤立页 / 31 文件名违规 / 4 超长页 / 4 出向链不足

**规则修订：**
- CLAUDE.md + wiki/SCHEMA.md 放宽 raw/ 禁令为「正文不可改，文件不可删，允许添加 frontmatter 元数据块」（A 方案：内联例外）

**已知偏差（非阻塞）：**
- Lint 报告 4.1 节「15 页 sources 为空」实为「15 页单源」误判（grep 漏识别 block-style YAML），已在报告头加订正。sources 字段实际全 24 页非空。

**报告：**
- `outputs/2026-05-21-lint-baseline.md`
- `outputs/2026-05-21-raw-frontmatter-baseline.md`
- `outputs/2026-05-21-wiki-confidence-baseline.md`
- `outputs/2026-05-21-p1-review.md`
