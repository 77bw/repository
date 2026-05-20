---
title: wiki confidence 信号补齐报告
created: 2026-05-21
type: report
---

# wiki confidence 信号补齐报告

## 摘要

- 总篇数：24
- high: 6
- medium: 14
- low: 4
- 跳过（已有 confidence）：0
- parse error：0

操作内容：在每篇 wiki 页 frontmatter 中追加 `confidence:` 字段，并将 `updated:` bump 到 `2026-05-21`。其他字段（title / type / tags / sources / created / summary）保持不动。正文 100% 未动。

## 详细列表

### high（高置信，6 篇）

| 文件 | sources 数 | 判断理由 |
|------|---------|--------|
| `wiki/concepts/claude-md.md` | 2 | 飞书 + ai-chat 两份完整指南互证，描述客观事实（层级/加载机制） |
| `wiki/entities/everything-claude-code.md` | 4 | 四份 Notion tech-stack 文档交叉互证 + 官方 GitHub 仓库可查 |
| `wiki/entities/git-branch.md` | 2 | 客观事实命令速查，多源验证 + 内容是 Git 命令汇总 |
| `wiki/entities/opencode.md` | 2 | 多份 Notion 文档互证 + 内容是官方安装/Agent/命令汇总 |
| `wiki/topics/claude-code-tips.md` | 2 | 团队内部分享 + 社区高频技巧两份独立来源汇总，事实型条目 |
| `wiki/topics/claude-md-mechanisms.md` | 2 | 完整指南 + 微信视频号互证，规范+机制类客观内容 |

### medium（中置信，14 篇）

| 文件 | sources 数 | 判断理由 |
|------|---------|--------|
| `wiki/concepts/agentic-engineering.md` | 1 | 单源观点综合（"五大技巧"是该作者的主张），但内容已被广泛验证 |
| `wiki/concepts/agents.md` | 1 | 单源 + 内容是事实陈述（AGENTS.md 与 CLAUDE.md 等价），保守取 medium |
| `wiki/concepts/coding-agent.md` | 1 | 单源 + 简短客观定义，无强观点 |
| `wiki/concepts/context-files.md` | 1 | 单源 + 内容含 ETH Zurich 学术研究引用，权威一手来源 |
| `wiki/entities/claude-plugins-official.md` | 1 | 单源（飞书）但内容是官方插件清单的客观汇总 |
| `wiki/entities/github-reference.md` | 1 | 单源 + 内容是 GitHub 平台事实速查，无主观判断 |
| `wiki/entities/openclaw.md` | 1 | 单源（GitHub README）+ 客观项目说明 |
| `wiki/topics/agent-team-architecture.md` | 2 | 双源（Claude Code 实战 + Mavis 架构）互补，但混入观点综合 |
| `wiki/topics/ai-agent-principles.md` | 1 | 单源 + 内容简短客观，保守取 medium |
| `wiki/topics/claude-md-guide.md` | 1 | 单源但引用 ETH Zurich 学术研究（权威一手） |
| `wiki/topics/codex-migration-guide.md` | 1 | 单源（HTML 文档）+ 内容是命令映射等客观事实 |
| `wiki/topics/prompt-mentor-guide.md` | 3 | 三源融合 + 内容是方法论综合（含主观主张），保守 medium |
| `wiki/topics/rag-optimization.md` | 1 | 单源 + 内容简短客观，保守取 medium |
| `wiki/topics/vibe-coding.md` | 2 | 双源（Notion + ai-chat）互证 + 含案例与观点综合 |

### low（建议后续补源或验证，4 篇）

| 文件 | sources 数 | 判断理由 |
|------|---------|--------|
| `wiki/topics/ai-driven-git-workflow.md` | 1 | 单源 + 强观点（"你是调度者，不是 Git 工程师"，"7 个最小命令"等主观主张） |
| `wiki/topics/claude-code-gen.md` | 1 | 单源 + 内容是评测+最佳实践（隐含主观判断），且页面极短 |
| `wiki/topics/karpathy-knowledge-base-sop.md` | 1 | 单源 + 大量 SOP 主张（"原则：不整理、不分类、直接扔"等强观点） |
| `wiki/topics/tdd-ddd-data-collection.md` | 1 | 单源 + 强观点（"TDD 三个隐藏前提一个都不满足"、"取什么扔什么"基于个人经验） |

## 后续建议

- low 类四篇：找第二个独立来源验证主张，或在页面 body 中明确标注观点边界（"基于作者个人经验"）。
- medium 类偏单源者（agents.md / coding-agent.md / ai-agent-principles.md / rag-optimization.md）是早期种子页，体量也短，下一轮 ingest 可考虑补源升至 high 或合并入更详尽的 topic 页。
- 没有页面被识别为已存在 confidence 字段——本次是真正的 P1 基线补齐。
