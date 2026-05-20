---
title: Lint 基线审计报告
created: 2026-05-21
type: report
---

# Lint 基线审计报告

> 阶段：P1 启动前的基线快照
> 扫描范围：wiki/ + raw/
> 扫描时间：2026-05-21 12:48 GMT+8
> 总文件：wiki/ 24 篇 + 3 元数据文件，raw/ 42 个
>
> **订正（2026-05-21 01:00）**：第 4.1 节「15 页 sources 为空」是误判——这 15 页都是**单源**（非空），grep 漏识别 block-style YAML（`sources:\n  - ...`）。sources 字段实际全 24 页非空。下方 P0 行动清单中「补充 sources 字段」一条作废。详见 `outputs/2026-05-21-p1-review.md`。

## 摘要

- **严重问题**：3 个
- **警告**：19 个
- **提示**：37 个

---

## 1. 断链（严重）

### 实际断链（目标文件不存在）

1. **`wiki/topics/karpathy-knowledge-base-sop.md`** 第 N 行
   - 链接：`[[raw/web/把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy]]`
   - 问题：raw 文件存在但路径不匹配（实际文件名含 `.md` 后缀）
   - 目标应为：`raw/web/把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md`

2. **`wiki/topics/codex-migration-guide.md`** 第 N 行
   - 链接：`[[raw/docs/codex-migration-manual]]`
   - 问题：raw 文件存在但路径不匹配（实际文件名含 `.html` 后缀）
   - 目标应为：`raw/docs/codex-migration-manual.html`

3. **`wiki/SCHEMA.md`** 第 N 行（示例文档中）
   - 链接：`[[raw/path/to/source]]`
   - 问题：这是示例占位符，不是真实链接，但仍在文档中
   - 建议：改为注释或移除

---

## 2. 孤立页（警告）

**所有 24 个 wiki 页均为孤立页**（无任何入向 wikilink）。

这违反了 SCHEMA.md 第 5 节规则：「每个新建/更新的 wiki 页**至少 2 个出向 wikilink**（避免孤立页）」。

虽然规则指的是出向链接，但实际上孤立页表明知识图谱尚未形成。所有页面仅在 INDEX.md 中被引用，但 INDEX.md 的引用不计入「入向 wikilink」。

**完整列表**（24 页）：
- concepts/agentic-engineering
- concepts/agents
- concepts/claude-md
- concepts/coding-agent
- concepts/context-files
- entities/claude-plugins-official
- entities/everything-claude-code
- entities/git-branch
- entities/github-reference
- entities/openclaw
- entities/opencode
- topics/agent-team-architecture
- topics/ai-agent-principles
- topics/ai-driven-git-workflow
- topics/claude-code-gen
- topics/claude-code-tips
- topics/claude-md-guide
- topics/claude-md-mechanisms
- topics/codex-migration-guide
- topics/karpathy-knowledge-base-sop
- topics/prompt-mentor-guide
- topics/rag-optimization
- topics/tdd-ddd-data-collection
- topics/vibe-coding

---

## 3. INDEX 与文件系统差异（严重）

### 文件系统中存在但 INDEX 未列出

**无**。所有 24 个 wiki 页均在 INDEX.md 中正确列出。

### INDEX 中列出但文件已删

**无**。INDEX 中的所有条目对应的文件都存在。

### INDEX 元数据过期

- **Last updated**: 2026-05-20（应为 2026-05-21）
- **Total pages**: 24（正确）

---

## 4. Frontmatter 缺失（严重 / 警告）

### 必填字段完整性检查

所有 24 个 wiki 页的必填字段（title / type / created / updated / tags / sources / summary）**均已填写**。

### 但发现的问题

#### 4.1 sources 字段为空或缺失（15 页）

这些页面的 `sources:` 字段为空数组 `[]` 或缺失，违反 SCHEMA.md 第 4.1 节要求：

1. `concepts/agents.md` — sources 为空
2. `concepts/claude-md.md` — sources 为空
3. `concepts/coding-agent.md` — sources 为空
4. `concepts/context-files.md` — sources 为空
5. `entities/claude-plugins-official.md` — sources 为空
6. `entities/everything-claude-code.md` — sources 为空
7. `entities/git-branch.md` — sources 为空
8. `entities/opencode.md` — sources 为空
9. `topics/agent-team-architecture.md` — sources 为空
10. `topics/ai-driven-git-workflow.md` — sources 为空
11. `topics/claude-code-tips.md` — sources 为空
12. `topics/claude-md-guide.md` — sources 为空
13. `topics/claude-md-mechanisms.md` — sources 为空
14. `topics/prompt-mentor-guide.md` — sources 为空
15. `topics/vibe-coding.md` — sources 为空

**根本原因**：这些页面在 frontmatter 中声明了 `sources:` 但值为空列表，应该填入对应的 raw 文件路径。

#### 4.2 confidence 字段缺失（全部 24 页）

SCHEMA.md 第 4.2 节标记 `confidence` 为「可选但强推荐」。当前**所有页面均缺失此字段**。

这是基线数据，后续由专项 agent 补充。

---

## 5. 非分类法 tag（警告）

### 已注册的合法 tag（SCHEMA.md 第 6 节）

**domain**: ai · prompt-engineering · ai-coding · spider · full-stack · knowledge-management
**ecosystem**: claude-code · codex · cursor · obsidian · langchain · mcp · github
**form**: methodology · sop · case-study · benchmark · controversy · timeline · tooling
**status**: stub · mature · contested · deprecated

### 实际使用的 tag

所有使用的 tag 均在上述注册表中，**无非分类法 tag**。

使用统计：
- `ai` — 8 页
- `ai-coding` — 13 页
- `prompt-engineering` — 4 页
- `full-stack` — 3 页
- `knowledge-management` — 4 页
- `spider` — 1 页

---

## 6. 缺 confidence 信号（提示）

**全部 24 页缺失 `confidence` 字段**。

这是基线数据，会由 Agent-D 处理。建议按以下规则补充：

- `confidence: high` — 多源验证（2+ raw 文件）或权威一手来源
- `confidence: medium` — 单一可信来源
- `confidence: low` — 单源主观或推断内容

**当前页面分类**（基于 sources 字段）：

| 类型 | 页数 | 建议 confidence |
|------|------|-----------------|
| 多源（2+ raw） | 3 | high |
| 单源 | 6 | medium |
| 无源（空） | 15 | low |

---

## 7. 超长页（提示）

### 超过 200 行的页面（候选拆分）

1. **`wiki/topics/agent-team-architecture.md`** — 368 行
   - 内容：Claude Code teamcreate 七大场景 + Mavis 架构
   - 建议拆分：可分为「Agent Team 设计模式」+ 「Claude Code teamcreate 实战」

2. **`wiki/topics/claude-md-mechanisms.md`** — 281 行
   - 内容：CLAUDE.md 完整机制与最佳实践
   - 建议拆分：可分为「CLAUDE.md 加载机制」+ 「CLAUDE.md 最佳实践」

3. **`wiki/topics/prompt-mentor-guide.md`** — 258 行
   - 内容：提示词导师 Agent 完整指南
   - 建议拆分：可分为「提示词导师方法论」+ 「提示词导师实战案例」

4. **`wiki/topics/codex-migration-guide.md`** — 227 行
   - 内容：Claude Code 迁移到 Codex 手册
   - 建议拆分：可分为「Codex 心智模型」+ 「Codex 命令映射」

### 接近阈值的页面（150-200 行）

- `wiki/topics/karpathy-knowledge-base-sop.md` — 181 行
- `wiki/entities/everything-claude-code.md` — 187 行
- `wiki/topics/tdd-ddd-data-collection.md` — 147 行

---

## 8. Raw/ Frontmatter 缺失（提示）

### 统计

- **总 raw 文件**：42 个
- **无 frontmatter**：30 个（71%）
- **有 frontmatter**：12 个（29%）

### 无 frontmatter 的文件（30 个）

这些文件需要在文件头添加 frontmatter 块（source_url / ingested / sha256）。

**ai-chat/ 目录**（7 个）：
- 2026-05-02-tdd-ddd-testing-notes.md
- 2026-05-20-prompt-mentor-shopee-case.md
- claude-code-agent-team-guide.md
- claude_md_complete_guide.md
- mavis-multi-agent-architecture.md
- VibeCoding就该这么做.md
- README.md

**feishu/ 目录**（4 个）：
- Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md
- 15 条高频实用的 Claude Code 技巧 - 飞书云文档.md
- Claude Code 官方最强插件 claude-plugins-official ，AI编程全流程覆盖 - 飞书云文档.md
- Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md

**notion/articles/ 目录**（5 个）：
- Claude 3 5 Sonnet 代码生成能力评测与最佳实践 32c81caa0df88000ba27d9a62e5bd901.md
- RAG 系统性能优化：从检索到生成的完整链路 32c81caa0df8802b9563dd83247ecdd1.md
- vibe-coding：我的 AI 编程实践 60ddeb93255d4c3488122b5843a55694.md
- 无标题 32c81caa0df8806f9366f04201078b63.md
- 深入理解 AI Agent 的工作原理与实践应用 32c81caa0df8808ab96fd0c49b56f5ae.md

**notion/best-practices/ 目录**（1 个）：
- 最佳实践 32781caa0df8819698e9e502b13ad1cb.md

**notion/tech-stack/ 目录**（13 个）：
- 5个ClaudeCode顶级工程技术 2ef81caa0df8805298cfe8e2dc7b7845.md
- AI 技术栈 32781caa0df881549c36e91a76226555.md
- Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充 2fd81caa0df88056a6a3d6c978c8136a.md
- ClaudeCode 2fc81caa0df880edb4dfd06d8499e993.md
- Everything Claude Code 使用指南 2fc81caa0df881839b61ddb79c82f131.md
- OpenCode + Oh My OpenCode 完整使用指南 2f781caa0df881689124e89483dfa762.md
- everything-claude-code-guide md 2ff81caa0df8813f998ffaafec20aeb2.md
- oh-my-opencode json 配置文件 2f781caa0df88158a0f5d752253c758a.md
- opencode json 配置文件 2f781caa0df881888b2de0d8840ac1ac.md
- 使用指南 2fd81caa0df8804ba98bc5292a535179.md
- 使用小tips 2f581caa0df88050ac20db637bfb912c.md
- 工具使用 32781caa0df8817d8f79f7327f4ffd34.md
- 配置相关 2fc81caa0df880a48a70e271bf4b0ff1.md

**notion/tools/ 目录**（4 个）：
- 01-Git协助 30581caa0df8806c9d91c3915aa767d9.md
- GitHub 功能术语速查 31981caa0df8805fb2daccb0659dd904.md
- 分支操作命令 30581caa0df880a495e8dfd481586bea.md
- 场景：在 feature-A 分支工作到一半，需要紧急切到 master 修 bug 34c81caa0df880648e1efdd656f75fdb.md

**web/ 目录**（1 个）：
- 把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md

---

## 9. 文件名规范（警告）

### 规范要求（SCHEMA.md 第 3 节）

英文小写 + 连字符；禁止中文文件名、空格、大写字母。

### 违规文件（raw/ 目录）

**严重违规**（含中文、空格、特殊字符）：

1. **feishu/** — 4 个文件
   - `Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md` — 含中文、逗号、引号、空格
   - `15 条高频实用的 Claude Code 技巧 - 飞书云文档.md` — 含中文、空格
   - `Claude Code 官方最强插件 claude-plugins-official ，AI编程全流程覆盖 - 飞书云文档.md` — 含中文、空格
   - `Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md` — 含中文、空格

2. **notion/articles/** — 5 个文件
   - `Claude 3 5 Sonnet 代码生成能力评测与最佳实践 32c81caa0df88000ba27d9a62e5bd901.md` — 含中文、空格
   - `RAG 系统性能优化：从检索到生成的完整链路 32c81caa0df8802b9563dd83247ecdd1.md` — 含中文、冒号、空格
   - `vibe-coding：我的 AI 编程实践 60ddeb93255d4c3488122b5843a55694.md` — 含中文、冒号、空格
   - `无标题 32c81caa0df8806f9366f04201078b63.md` — 含中文、空格
   - `深入理解 AI Agent 的工作原理与实践应用 32c81caa0df8808ab96fd0c49b56f5ae.md` — 含中文、空格

3. **notion/best-practices/** — 1 个文件
   - `最佳实践 32781caa0df8819698e9e502b13ad1cb.md` — 含中文、空格

4. **notion/tech-stack/** — 13 个文件
   - `5个ClaudeCode顶级工程技术 2ef81caa0df8805298cfe8e2dc7b7845.md` — 含中文、空格
   - `AI 技术栈 32781caa0df881549c36e91a76226555.md` — 含中文、空格
   - `Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充 2fd81caa0df88056a6a3d6c978c8136a.md` — 含中文、空格、连字符
   - `ClaudeCode 2fc81caa0df880edb4dfd06d8499e993.md` — 含大写字母、空格
   - `Everything Claude Code 使用指南 2fc81caa0df881839b61ddb79c82f131.md` — 含大写字母、中文、空格
   - `OpenCode + Oh My OpenCode 完整使用指南 2f781caa0df881689124e89483dfa762.md` — 含大写字母、中文、空格、加号
   - `everything-claude-code-guide md 2ff81caa0df8813f998ffaafec20aeb2.md` — 含空格（md 前）
   - `oh-my-opencode json 配置文件 2f781caa0df88158a0f5d752253c758a.md` — 含中文、空格
   - `opencode json 配置文件 2f781caa0df881888b2de0d8840ac1ac.md` — 含中文、空格
   - `使用指南 2fd81caa0df8804ba98bc5292a535179.md` — 含中文、空格
   - `使用小tips 2f581caa0df88050ac20db637bfb912c.md` — 含中文、空格
   - `工具使用 32781caa0df8817d8f79f7327f4ffd34.md` — 含中文、空格
   - `配置相关 2fc81caa0df880a48a70e271bf4b0ff1.md` — 含中文、空格

5. **notion/tools/** — 4 个文件
   - `01-Git协助 30581caa0df8806c9d91c3915aa767d9.md` — 含中文、空格
   - `GitHub 功能术语速查 31981caa0df8805fb2daccb0659dd904.md` — 含中文、空格
   - `分支操作命令 30581caa0df880a495e8dfd481586bea.md` — 含中文、空格
   - `场景：在 feature-A 分支工作到一半，需要紧急切到 master 修 bug 34c81caa0df880648e1efdd656f75fdb.md` — 含中文、冒号、空格

6. **web/** — 1 个文件
   - `把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md` — 含中文、空格

7. **ai-chat/** — 1 个文件
   - `VibeCoding就该这么做.md` — 含中文

8. **wechat/** — 1 个文件
   - `微信视频号内容总结文档.md` — 含中文、空格

9. **notion/** — 1 个文件
   - `📚 02-知识库 2ef81caa0df880bbb313dfa12958d76f.md` — 含 emoji、中文、空格

**总计**：31 个文件违反命名规范（74% 的 raw 文件）

---

## 10. 出向 Wikilink 不足（警告）

SCHEMA.md 第 5 节要求：「每个新建/更新的 wiki 页**至少 2 个出向 wikilink**」。

### 不足 2 个出向 wikilink 的页面（4 页）

1. **`wiki/entities/git-branch.md`** — 1 个 wikilink
2. **`wiki/entities/github-reference.md`** — 1 个 wikilink
3. **`wiki/entities/openclaw.md`** — 1 个 wikilink
4. **`wiki/topics/rag-optimization.md`** — 1 个 wikilink

---

## 11. Raw 文件覆盖率（警告）

### 统计

- **总 raw 文件**：42 个
- **在 wiki 中被引用**：8 个（19%）
- **未被引用**：34 个（81%）

### 未被引用的 raw 文件（34 个）

这些文件已落盘但尚未被任何 wiki 页的 `sources:` 字段引用，表明摄入工作不完整。

**ai-chat/** — 7 个：
- 2026-05-02-tdd-ddd-testing-notes.md
- 2026-05-20-prompt-mentor-shopee-case.md
- VibeCoding就该这么做.md
- claude-code-agent-team-guide.md
- claude_md_complete_guide.md
- mavis-multi-agent-architecture.md
- README.md

**feishu/** — 4 个：
- 15 条高频实用的 Claude Code 技巧 - 飞书云文档.md
- Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md
- Claude Code 官方最强插件 claude-plugins-official ，AI编程全流程覆盖 - 飞书云文档.md
- Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md

**notion/articles/** — 5 个：
- Claude 3 5 Sonnet 代码生成能力评测与最佳实践 32c81caa0df88000ba27d9a62e5bd901.md
- RAG 系统性能优化：从检索到生成的完整链路 32c81caa0df8802b9563dd83247ecdd1.md
- vibe-coding：我的 AI 编程实践 60ddeb93255d4c3488122b5843a55694.md
- 无标题 32c81caa0df8806f9366f04201078b63.md
- 深入理解 AI Agent 的工作原理与实践应用 32c81caa0df8808ab96fd0c49b56f5ae.md

**notion/best-practices/** — 1 个：
- 最佳实践 32781caa0df8819698e9e502b13ad1cb.md

**notion/tech-stack/** — 13 个：
- 5个ClaudeCode顶级工程技术 2ef81caa0df8805298cfe8e2dc7b7845.md
- AI 技术栈 32781caa0df881549c36e91a76226555.md
- Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充 2fd81caa0df88056a6a3d6c978c8136a.md
- ClaudeCode 2fc81caa0df880edb4dfd06d8499e993.md
- Everything Claude Code 使用指南 2fc81caa0df881839b61ddb79c82f131.md
- OpenCode + Oh My OpenCode 完整使用指南 2f781caa0df881689124e89483dfa762.md
- everything-claude-code-guide md 2ff81caa0df8813f998ffaafec20aeb2.md
- oh-my-opencode json 配置文件 2f781caa0df88158a0f5d752253c758a.md
- opencode json 配置文件 2f781caa0df881888b2de0d8840ac1ac.md
- 使用指南 2fd81caa0df8804ba98bc5292a535179.md
- 使用小tips 2f581caa0df88050ac20db637bfb912c.md
- 工具使用 32781caa0df8817d8f79f7327f4ffd34.md
- 配置相关 2fc81caa0df880a48a70e271bf4b0ff1.md

**notion/tools/** — 4 个：
- 01-Git协助 30581caa0df8806c9d91c3915aa767d9.md
- GitHub 功能术语速查 31981caa0df8805fb2daccb0659dd904.md
- 分支操作命令 30581caa0df880a495e8dfd481586bea.md
- 场景：在 feature-A 分支工作到一半，需要紧急切到 master 修 bug 34c81caa0df880648e1efdd656f75fdb.md

**web/** — 1 个：
- 把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md

---

## 建议优先级

### P0（立即修）

1. **修复断链**（3 个）
   - 更新 `karpathy-knowledge-base-sop.md` 中的 raw 链接
   - 更新 `codex-migration-guide.md` 中的 raw 链接
   - 移除或注释 SCHEMA.md 中的示例占位符

2. **补充 sources 字段**（15 页）
   - 所有 sources 为空的页面需要填入对应的 raw 文件路径
   - 这是 SCHEMA.md 第 4.1 节的必填要求

3. **修复文件名规范**（31 个 raw 文件）
   - 将所有含中文、空格、特殊字符的文件名改为英文小写 + 连字符
   - 这会影响 wiki 中的 sources 引用，需要同步更新

### P1（本批修）

1. **补充 confidence 字段**（24 页）
   - 由 Agent-D 处理
   - 按多源/单源/无源分类补充

2. **补充 raw/ frontmatter**（30 个文件）
   - 由 Agent-C 处理
   - 添加 source_url / ingested / sha256 元数据

3. **补充出向 wikilink**（4 页）
   - `git-branch.md` / `github-reference.md` / `openclaw.md` / `rag-optimization.md`
   - 每页至少补充到 2 个出向链接

### P2（择期）

1. **拆分超长页**（4 页）
   - `agent-team-architecture.md`（368 行）
   - `claude-md-mechanisms.md`（281 行）
   - `prompt-mentor-guide.md`（258 行）
   - `codex-migration-guide.md`（227 行）

2. **补充孤立页的入向链接**（24 页）
   - 建立知识图谱，让页面之间相互引用
   - 这是长期工作，不影响基线功能

3. **完成 raw 文件摄入**（34 个）
   - 这些文件已落盘但未被 wiki 页引用
   - 需要评估是否应该创建新 wiki 页或合并到现有页

---

## 数据快照

| 指标 | 数值 |
|------|------|
| Wiki 页总数 | 24 |
| Raw 文件总数 | 42 |
| 断链 | 3 |
| 孤立页 | 24 |
| 出向 wikilink < 2 的页 | 4 |
| Sources 为空的页 | 15 |
| Confidence 缺失的页 | 24 |
| Raw 无 frontmatter | 30 |
| 文件名违规（raw） | 31 |
| Raw 文件被引用率 | 19% |

---

## 后续行动

1. **立即**：修复 P0 问题（断链、sources、文件名）
2. **本周**：补充 P1 数据（confidence、frontmatter、wikilink）
3. **下周**：处理 P2 优化（拆分、图谱、摄入完成）
4. **定期**：每周运行 lint 检查，保持基线健康

