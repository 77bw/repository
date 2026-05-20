---
title: raw/ frontmatter 基线建立报告
created: 2026-05-21
type: report
---

# raw/ frontmatter 基线建立报告

## 摘要

- 扫描总数：37 .md + 2 html + 1 pdf
- 成功添加 frontmatter：30
- 已有 frontmatter 跳过：7
- 失败：0（初次扫描有 1 个权限问题，已通过临时 `chmod u+w`/恢复 444 解决）
- 跳过 html/pdf：3 个

抽样验证 4 个文件（`README.md`、`ai-chat/claude-code-agent-team-guide.md`、`notion/tools/01-Git协助...md`、`ai-chat/claude_md_complete_guide.md`）：
`tail -n +7 file | shasum -a 256` 输出与 frontmatter 中的 `sha256` 字段全部一致，正文 byte-for-byte 不变。

> **说明**：所有新建 frontmatter 文件的 `source_url` 字段当前都为空字符串。本次素材集合中 `raw/notion/`、`raw/ai-chat/`、`raw/README.md` 均无外链原文（私有空间导出 / 对话记录 / 仓库说明）；脚本对 `notion/`、`ai-chat/` 路径已硬规则置空。`feishu/`、`web/`、`wechat/` 均已在前置批次填好 frontmatter，本次只需跳过。

## 成功列表（按目录分组）

### raw/
- `raw/README.md` — source_url: ""，ingested: 2026-04-29

### raw/ai-chat/
- `raw/ai-chat/2026-05-02-tdd-ddd-testing-notes.md` — source_url: ""，ingested: 2026-05-16
- `raw/ai-chat/2026-05-20-prompt-mentor-shopee-case.md` — source_url: ""，ingested: 2026-05-20
- `raw/ai-chat/claude-code-agent-team-guide.md` — source_url: ""，ingested: 2026-05-16
- `raw/ai-chat/claude_md_complete_guide.md` — source_url: ""，ingested: 2026-05-16（处理后已恢复 0444 只读位）
- `raw/ai-chat/mavis-multi-agent-architecture.md` — source_url: ""，ingested: 2026-05-16

### raw/notion/
- `raw/notion/📚 02-知识库 2ef81caa0df880bbb313dfa12958d76f.md` — source_url: ""，ingested: 2026-04-29

### raw/notion/articles/
- `raw/notion/articles/Claude 3 5 Sonnet 代码生成能力评测与最佳实践 32c81caa0df88000ba27d9a62e5bd901.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/articles/RAG 系统性能优化：从检索到生成的完整链路 32c81caa0df8802b9563dd83247ecdd1.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/articles/vibe-coding：我的 AI 编程实践 60ddeb93255d4c3488122b5843a55694.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/articles/无标题 32c81caa0df8806f9366f04201078b63.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/articles/深入理解 AI Agent 的工作原理与实践应用 32c81caa0df8808ab96fd0c49b56f5ae.md` — source_url: ""，ingested: 2026-04-29

### raw/notion/best-practices/
- `raw/notion/best-practices/最佳实践 32781caa0df8819698e9e502b13ad1cb.md` — source_url: ""，ingested: 2026-04-29

### raw/notion/tech-stack/
- `raw/notion/tech-stack/5个ClaudeCode顶级工程技术 2ef81caa0df8805298cfe8e2dc7b7845.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/AI 技术栈 32781caa0df881549c36e91a76226555.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充 2fd81caa0df88056a6a3d6c978c8136a.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/ClaudeCode 2fc81caa0df880edb4dfd06d8499e993.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/Everything Claude Code 使用指南 2fc81caa0df881839b61ddb79c82f131.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/OpenCode + Oh My OpenCode 完整使用指南 2f781caa0df881689124e89483dfa762.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/everything-claude-code-guide md 2ff81caa0df8813f998ffaafec20aeb2.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/oh-my-opencode json 配置文件 2f781caa0df88158a0f5d752253c758a.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/opencode json 配置文件 2f781caa0df881888b2de0d8840ac1ac.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/使用小tips 2f581caa0df88050ac20db637bfb912c.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/使用指南 2fd81caa0df8804ba98bc5292a535179.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tech-stack/配置相关 2fc81caa0df880a48a70e271bf4b0ff1.md` — source_url: ""，ingested: 2026-04-29

### raw/notion/tools/
- `raw/notion/tools/01-Git协助 30581caa0df8806c9d91c3915aa767d9.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tools/GitHub 功能术语速查 31981caa0df8805fb2daccb0659dd904.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tools/分支操作命令 30581caa0df880a495e8dfd481586bea.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tools/场景：在 feature-A 分支工作到一半，需要紧急切到 master 修 bug 34c81caa0df880648e1efdd656f75fdb.md` — source_url: ""，ingested: 2026-04-29
- `raw/notion/tools/工具使用 32781caa0df8817d8f79f7327f4ffd34.md` — source_url: ""，ingested: 2026-04-29

## 跳过列表（已有 frontmatter）

- `raw/ai-chat/VibeCoding就该这么做.md` — 已有 frontmatter
- `raw/feishu/15 条高频实用的 Claude Code 技巧 - 飞书云文档.md` — 已有 frontmatter
- `raw/feishu/Claude Code 团队分享的 10 个内部AI 编程技巧 - 飞书云文档.md` — 已有 frontmatter
- `raw/feishu/Claude Code 官方最强插件 claude-plugins-official ，AI编程全流程覆盖 - 飞书云文档.md` — 已有 frontmatter
- `raw/feishu/Claude.md,AI 编程的 "宪法"，如何编写 Claude.md 新手指南 - 飞书云文档.md` — 已有 frontmatter
- `raw/web/把一堆乱七八糟的笔记，变成一个会自己生长的知识库-Karpathy.md` — 已有 frontmatter
- `raw/wechat/微信视频号内容总结文档.md` — 已有 frontmatter

## 失败列表

无（最终全部成功）。

> 过程记录：`raw/ai-chat/claude_md_complete_guide.md` 初次因文件位 `0444` 报 `Permission denied`；以 `chmod u+w` 解锁、写入 frontmatter、再 `chmod 444` 还原。文件大小从 12473 → 12591（差值 = 写入的 118 字节 frontmatter）。

## 已跳过的非 .md 文件

- `raw/docs/Obsidian-AI-The-Complete-Guide-v1.0.0.pdf`
- `raw/docs/claude-code-cheatsheet.html`
- `raw/docs/codex-migration-manual.html`

> 未来若需对二进制 / HTML 做漂移检测，建议沿用 SCHEMA 思路放外置 sidecar：在同目录放 `<filename>.meta.yaml`（含 `source_url` / `ingested` / `sha256`），保持原文件 byte-for-byte 不变即可。

## 验证摘要

随机抽 4 文件，比对 frontmatter `sha256` 与 `tail -n +7 <file> | shasum -a 256` 输出，全部命中：

| 文件 | 一致 |
|------|------|
| `raw/README.md` | ✓ |
| `raw/ai-chat/claude-code-agent-team-guide.md` | ✓ |
| `raw/notion/tools/01-Git协助 30581caa0df8806c9d91c3915aa767d9.md` | ✓ |
| `raw/ai-chat/claude_md_complete_guide.md` | ✓ |
