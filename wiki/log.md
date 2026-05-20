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
