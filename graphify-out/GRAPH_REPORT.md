# Graph Report - ./wiki  (2026-04-26)

## Corpus Check
- Corpus is ~4,094 words - fits in a single context window. You may not need a graph.

## Summary
- 52 nodes · 70 edges · 9 communities detected
- Extraction: 77% EXTRACTED · 23% INFERRED · 0% AMBIGUOUS · INFERRED: 16 edges (avg confidence: 0.81)
- Token cost: 4,800 input · 1,200 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Context Config Files|Context Config Files]]
- [[_COMMUNITY_Agentic Engineering Techniques|Agentic Engineering Techniques]]
- [[_COMMUNITY_Claude Code Authors|Claude Code Authors]]
- [[_COMMUNITY_Claude Code Best Practices|Claude Code Best Practices]]
- [[_COMMUNITY_AI Agent Orchestration|AI Agent Orchestration]]
- [[_COMMUNITY_CLAUDE.md Research & Codex Migration|CLAUDE.md Research & Codex Migration]]
- [[_COMMUNITY_Knowledge Base & RAG|Knowledge Base & RAG]]
- [[_COMMUNITY_AI Coding Tools|AI Coding Tools]]
- [[_COMMUNITY_Claude Code Hooks|Claude Code Hooks]]

## God Nodes (most connected - your core abstractions)
1. `Wiki Index` - 11 edges
2. `Agentic Engineering` - 8 edges
3. `Claude Code Best Practice` - 8 edges
4. `Claude Plugins Official` - 7 edges
5. `Everything Claude Code` - 7 edges
6. `CLAUDE.md` - 6 edges
7. `Context Files` - 6 edges
8. `Claude Code Best Practice Guide (82 Tips)` - 6 edges
9. `Coding Agent` - 5 edges
10. `Claude Code Authors` - 5 edges

## Surprising Connections (you probably didn't know these)
- `Context Engineering` --conceptually_related_to--> `Context Files`  [INFERRED]
  wiki/concepts/agentic-engineering.md → wiki/concepts/context-files.md
- `OpenClaw` --semantically_similar_to--> `Context Files`  [INFERRED] [semantically similar]
  wiki/entities/openclaw.md → wiki/concepts/context-files.md
- `Boris Cherny` --conceptually_related_to--> `Claude Code Best Practice`  [INFERRED]
  wiki/entities/claude-code-authors.md → wiki/entities/claude-code-best-practice.md
- `Claude Code Commands` --semantically_similar_to--> `Commandify Everything`  [INFERRED] [semantically similar]
  wiki/entities/claude-code-best-practice.md → wiki/concepts/agentic-engineering.md
- `Subagent Orchestration` --semantically_similar_to--> `Claude Code Subagents`  [INFERRED] [semantically similar]
  wiki/concepts/agentic-engineering.md → wiki/entities/claude-code-best-practice.md

## Hyperedges (group relationships)
- **Context Files + Coding Agent + CLAUDE.md form the AI Agent instruction system** — contextfiles_concept, codingagent_concept, claudemd_concept, agents_concept [EXTRACTED 0.95]
- **Subagents + Commands + Skills + Hooks form Claude Code extension architecture** — claudecodebestpractice_subagents, claudecodebestpractice_commands, claudecodebestpractice_skills, claudecodebestpractice_hooks [EXTRACTED 1.00]
- **PRD First + Commandify + Subagent Orchestration form the Agentic Engineering workflow** — agenticengineering_prd_first, agenticengineering_commandify, agenticengineering_subagent_orchestration [EXTRACTED 0.90]
- **Claude Code Ecosystem: Hooks + Subagents + Skills** — everything_claude_code_hooks, claude_code_best_practice_hooks, everything_claude_code_subagent, claude_code_best_practice_subagents [INFERRED 0.85]
- **Context File Pattern: CLAUDE.md / AGENTS.md / context-files** — claude_md_guide_topic, codex_migration_agents_md, claude_code_best_practice_settings [INFERRED 0.82]
- **Knowledge Base Workflow: raw → wiki → graphify → query** — karpathy_sop_topic, karpathy_sop_graphify_phase, karpathy_sop_rationale_graphify [EXTRACTED 0.95]

## Communities

### Community 0 - "Context Config Files"
Cohesion: 0.3
Nodes (12): AGENTS.md, CLAUDE.md, claude-md-management Plugin, Claude Plugins Official, feature-dev Plugin, skill-creator Plugin, Coding Agent, Context Files (+4 more)

### Community 1 - "Agentic Engineering Techniques"
Cohesion: 0.22
Nodes (11): Commandify Everything, Agentic Engineering, Context Engineering, Modular Rules Architecture, PRD First Development, Subagent Orchestration, Claude Code Commands, Claude Code Best Practice (+3 more)

### Community 2 - "Claude Code Authors"
Cohesion: 0.4
Nodes (5): Affaan Mustafa, Alex Albert, Boris Cherny, Claude Code Authors, Riley Goodside

### Community 3 - "Claude Code Best Practices"
Cohesion: 0.5
Nodes (5): Claude Code - Context Management (40%/60% rule), Claude Code - Settings Priority Hierarchy, Claude Code Best Practice Guide (82 Tips), Claude Code High-Frequency Tips, Git Branch Operations

### Community 4 - "AI Agent Orchestration"
Cohesion: 0.4
Nodes (5): AI Agent Principles, Claude Code - Subagent Isolation Pattern, Everything Claude Code - Subagent Orchestration, OpenCode + Oh My OpenCode, Oh My OpenCode Agent System

### Community 5 - "CLAUDE.md Research & Codex Migration"
Cohesion: 0.4
Nodes (5): ETH Zurich Finding: CLAUDE.md Reduces Task Success, Claude.md Guide, AGENTS.md (Codex equivalent of CLAUDE.md), Codex - Claude Code Command Mapping, Codex Migration Guide from Claude Code

### Community 6 - "Knowledge Base & RAG"
Cohesion: 0.5
Nodes (4): Karpathy SOP - Graphify Phase (50+ articles), Rationale: Why Graphify after 50 articles, Karpathy Knowledge Base + Graphify SOP, RAG System Optimization

### Community 7 - "AI Coding Tools"
Cohesion: 1.0
Nodes (3): Claude Code Generation Best Practices, Everything Claude Code, Vibe Coding Practice

### Community 8 - "Claude Code Hooks"
Cohesion: 1.0
Nodes (2): Claude Code - Hooks (PostToolUse/Stop/PreToolUse), Everything Claude Code - Session Hooks

## Knowledge Gaps
- **19 isolated node(s):** `Wiki Schema`, `PRD First Development`, `Modular Rules Architecture`, `Affaan Mustafa`, `Alex Albert` (+14 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `Claude Code Hooks`** (2 nodes): `Claude Code - Hooks (PostToolUse/Stop/PreToolUse)`, `Everything Claude Code - Session Hooks`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Wiki Index` connect `Context Config Files` to `Agentic Engineering Techniques`, `Claude Code Authors`?**
  _High betweenness centrality (0.136) - this node is a cross-community bridge._
- **Why does `Everything Claude Code` connect `AI Coding Tools` to `Claude Code Hooks`, `Claude Code Best Practices`, `AI Agent Orchestration`, `CLAUDE.md Research & Codex Migration`?**
  _High betweenness centrality (0.091) - this node is a cross-community bridge._
- **Why does `Claude Code Best Practice` connect `Agentic Engineering Techniques` to `Context Config Files`, `Claude Code Authors`?**
  _High betweenness centrality (0.089) - this node is a cross-community bridge._
- **What connects `Wiki Schema`, `PRD First Development`, `Modular Rules Architecture` to the rest of the system?**
  _19 weakly-connected nodes found - possible documentation gaps or missing edges._