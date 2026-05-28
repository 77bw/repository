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

## [2026-05-21 01:45] migrate | P2 阶段：文件名规范化 + 超长页拆分 + 反链补齐

并行 4 子代理（阶段 A→B→C）+ Review 终审 PASS。

**文件名规范化（32 个 raw/ 文件）：**
- git mv 重命名：去掉中文/空格/UUID/大写，统一英文小写+连字符
- 更新 wiki 中 37 处 sources 引用指向新路径
- 验证：无违规文件名残留，无旧路径引用残留

**SCHEMA 断链 + sidecar：**
- 修复 SCHEMA.md 模板中 1 处 `[[raw/...]]` 占位符（改为纯文本）
- 创建 3 个 sidecar 元数据文件（pdf + 2 html 的 sha256 + ingested）

**超长页拆分（4 页 → 8 页）：**
- agent-team-architecture (369→200) + claude-code-teamcreate-practice (198)
- claude-md-mechanisms (282→200) + claude-md-best-practices (108)
- prompt-mentor-guide (258→126) + prompt-mentor-case-study (165)
- codex-migration-guide (227→123) + codex-command-mapping (145)

**反链补齐：**
- 28 个 wiki 页面全部有 ≥1 入向 wikilink，知识图谱完全连通
- 高连接度节点：agentic-engineering(18) / claude-md(14) / context-files(11) / coding-agent(11)

**报告：**
- `outputs/2026-05-21-p2-review.md`


## [2026-05-21 22:30] update | 移除文档中残留的 graphify 引用

**背景：** /graphify skill 已在用户层面彻底删除，但项目文档（README + wiki）仍残留推荐用法，按当前状态会误导读者去用一个不存在的工具。

**清理：**
- `README.md`：第 37 行去掉 graphify-out 浏览提示；表格删「更新图谱」一行，「使用」一行去掉 graphify 查询；FAQ 删「知识图谱怎么更新？」，后续编号 7-10 → 6-9；「如何搜索特定主题？」由三种方式改两种
- `wiki/topics/karpathy-knowledge-base-sop.md`：标题/summary/H1 去掉「+ Graphify」字样；概述删 Graphify 角色分工；删除整个「阶段四：Graphify 图谱生成」章节；后续阶段五/六重新编号为四/五；完整工作流 ASCII 删除 `/graphify` 节点
- `wiki/INDEX.md`：第 41 行条目描述同步更新

**保留（按规则）：**
- `raw/web/karpathy-knowledge-base-article.md` 原文不动（raw/ 严格只读）
- `log.md` 既有 2 条历史记录保留（log append-only）
- `.obsidian/workspace.json` 不动（Obsidian 个人配置）

**验证：** `grep -rn graphify` 仅命中 raw/ 与 log.md 历史条目，所有面向读者的推荐已清除。

## [2026-05-28 12:55] delete | raw/wechat/wechat-video-content-summary.md

**原因：** Ali 厂长视频号内容主张"将 CLAUDE.md 压到 60 行以内"，该建议缺乏可靠依据，用户判定为误导性内容。

**操作：**
- `git rm raw/wechat/wechat-video-content-summary.md`
- `wiki/topics/claude-md-best-practices.md`：重写"60 行精简法"小节为"CLAUDE.md 组织方法"（保留三步拆分方法论，去掉60行结论和归因）；更新 title/summary/confidence
- `wiki/topics/claude-md-mechanisms.md`：移除 sources 引用；更新 summary 和正文
- `wiki/INDEX.md`：更新 best-practices 条目描述
