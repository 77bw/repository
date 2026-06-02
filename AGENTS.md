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

# [repository] recent context, 2026-06-02 2:09pm GMT+8

Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note
Format: ID TIME TYPE TITLE
Fetch details: get_observations([IDs]) | Search: mem-search skill

Stats: 50 obs (10,636t read) | 0t work

### Jun 1, 2026
S306 Generate optimized prompt package for wiki audit task; design two-phase workflow (Phase 1: audit recommendations only, Phase 2: execute changes after approval). (Jun 1 at 12:59 AM)
S307 Wiki 全库内容审计 — 扫描 28 页 wiki，找出过时/错误/与官方冲突的内容，生成分级建议文档 (Jun 1 at 1:00 AM)
S309 Optimize personal knowledge base according to wiki audit suggestions from outputs/2026-06-01-wiki-audit-suggestions.md (Jun 1 at 1:03 AM)
S310 Wiki 全库内容审计 → 修改建议文档。扫描 wiki/ 全部页面，找出内容不正确/过时/与官方文档冲突/内部不一致的点，产出建议文档。 (Jun 1 at 1:33 AM)
### Jun 2, 2026
S311 Execute git-quick workflow to commit and push comprehensive wiki audit corrections (Jun 2 at 12:01 AM)
S315 Research iv8 JavaScript execution environment and create comprehensive wiki documentation focusing on environment补充 (environment patching) techniques and anti-scraping countermeasures (Jun 2 at 12:16 AM)
S316 Deep analysis of iv8 (Python V8 runtime) and reverse-skill (AI-driven JS reverse-engineering) repositories to synthesize reusable knowledge clusters: iv8 supplement-env handbook + AI reverse-engineering 0→1 workflow guide with Node→iv8 technique mapping. (Jun 2 at 12:56 AM)
S317 Deep reverse engineering knowledge synthesis: extract & orchestrate iv8 implementation documentation, reverse-skill JS逆向 methodology (17 references + AST handbook), ai-reverse-toolkit & js-reverse-mcp frameworks into unified wiki knowledge base via multi-phase workflow orchestration. (Jun 2 at 1:13 AM)
S319 补写并整理 iv8 与 AI JS 逆向 wiki 知识簇，并同步索引与日志 (Jun 2 at 9:17 AM)
S320 开启 fast 模式：确认现有上下文后，仅补充一小段 fast 模式规则，避免覆盖已有改动 (Jun 2 at 1:35 PM)
1818 1:57p 🟣 h5st example packages the canonical iv8 replay recipe
1819 " 🟣 abogus example confirms the Douyin netLog harvest pattern
1820 " 🟣 tdc example combines vision, POW, and trusted pointer replay
1817 " 🔵 iv8 逆向知识页的事实漂移与待校准点已定位
1823 " ✅ iv8 实体页已补齐为可复用的补环境基座
1824 " ✅ reverse-skill 实体页已收敛为“侦察 + 补环境”双层工作流
1825 " ✅ iv8-first 逆向工作流与补环境手册已形成统一主线
1829 1:58p 🟣 abogus example codifies the Douyin replay pattern
1830 " 🟣 h5st example packages the canonical iv8 signing recipe
1831 " 🟣 tdc example combines vision, POW, and trusted input replay
1832 " 🟣 税务 and 药监局 examples mark the minimal iv8 footprint
1833 " 🔵 Current wiki pages already encode the upstream boundaries and caveats
1826 " 🔵 Context7 未收录 iv8，只能回退到 V8 与 Node.js 背景库
1827 " ✅ js-reverse-mcp 实体页已固定为真实 Chrome 侦察层
1828 " 🔵 iv8 / reverse-skill / js-reverse-mcp / ai-reverse-toolkit 的一手校验素材已定位
1835 " ⚖️ jsr-reverse 的默认路由已收敛为七阶段工作脊柱
1836 " ✅ reverse-skill README 已对齐统一入口与交接工件
1837 " 🔵 iv8 仓库的一手 API 和 demo 证据已被确认
1838 " 🔵 raw 源目录已收敛到一份 GitHub 与一份 WeChat 原始文件
1834 1:59p 🔵 Issue #9 captures a Ruishu edge case where AJAX cookie validation fails
1839 " 🔵 raw/GitHub 与 raw/wechat 源文件缺少摄入 frontmatter
1840 " 🔵 jsr-reverse 的 stage 路由已固化为两步决策
1841 " ✅ iv8 相关一手证据已从文案落到 demos/examples
1842 " ⚖️ Context7 的 iv8 缺口已确认，后续只能借 V8/Node 背景补位
1843 2:00p ✅ JS 逆向补环境概念页已抽象为通用方法论
1844 " 🔵 当前 iv8 知识页规模受控，未触发单页拆分阈值
1845 " 🔵 reverse-skill 仓库已形成完整的技能、参考与测试分层
1846 2:01p ✅ iv8 相关断言已改为“可证能力 + 需实测边界”
1847 " ✅ reverse-skill 实体页已对齐真实 stage spine 与入口来源
1848 " ✅ 逆向工具与案例页的热度与职责边界已刷新
1849 2:02p ✅ iv8 概念与实体页已收敛为“可证能力 + 实测边界”
1850 " ✅ reverse-skill 实体页已对齐仓库本体与阶段主干
1851 " ✅ js-reverse-mcp 与 ai-reverse-toolkit 的热度与定位已刷新
1852 2:03p ✅ iv8 API 速查页已弱化过度承诺并强化实测边界
1853 " ✅ iv8 补环境手册已去重并重排证据等级
1854 " ✅ iv8 反爬案例库已标清官方 examples 与迁移参考边界
1855 " ✅ iv8 API 速查页已收敛为“可证能力 + 实测边界”
1858 2:04p ⚖️ Converged on IV8 reverse-engineering wiki goal
1859 2:05p ✅ Wiki index and raw catalog updated for IV8 reverse-engineering corpus
1860 " 🔵 Wiki corpus expanded with new iv8 and JS reverse-engineering pages
1861 " 🔵 iv8 case pages now separate official examples from Akamai migration reference
1862 2:06p 🔵 Wiki frontmatter validation script failed on missing Date constant
1863 " 🔵 Wiki link audit exposed one false-positive wikilink and two dangling raw references
1864 " 🔵 Raw corpus integrity check revealed widespread sha256 drift and missing frontmatter
1865 2:07p 🔵 Wiki frontmatter validation exposed a tag taxonomy mismatch across the corpus
1866 " 🔵 Wiki index parity check passed after filtering to navigation entries only
1867 " 🔵 Wiki link validation now passes with raw provenance links excluded
1868 " 🔵 Updated raw README checksum now matches the file body
1869 " 🔵 Wiki schema now clearly defines the allowed tag taxonomy v2
1870 " 🔵 Raw ingestion spec hashes only the post-frontmatter body
</claude-mem-context>