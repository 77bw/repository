# Graph Report - ./wiki  (2026-05-20)

## Corpus Check
- 26 files · ~7,881 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 62 nodes · 100 edges · 9 communities detected
- Extraction: 76% EXTRACTED · 24% INFERRED · 0% AMBIGUOUS · INFERRED: 24 edges (avg confidence: 0.79)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Plugin & Agent Ecosystem|Plugin & Agent Ecosystem]]
- [[_COMMUNITY_Agentic Engineering Methodology|Agentic Engineering Methodology]]
- [[_COMMUNITY_CLAUDE.md Config & Codex|CLAUDE.md Config & Codex]]
- [[_COMMUNITY_Claude Code Tooling|Claude Code Tooling]]
- [[_COMMUNITY_Key Contributors|Key Contributors]]
- [[_COMMUNITY_Session Management Patterns|Session Management Patterns]]
- [[_COMMUNITY_Subagent & Multi-Agent|Subagent & Multi-Agent]]
- [[_COMMUNITY_Workflow Architecture|Workflow Architecture]]
- [[_COMMUNITY_Knowledge Base & RAG|Knowledge Base & RAG]]

## God Nodes (most connected - your core abstractions)
1. `Wiki Index` - 25 edges
2. `Everything Claude Code` - 11 edges
3. `Agentic Engineering` - 8 edges
4. `Claude Code Best Practice` - 8 edges
5. `Claude Plugins Official` - 7 edges
6. `Agentic Engineering` - 7 edges
7. `CLAUDE.md` - 6 edges
8. `Context Files` - 6 edges
9. `Claude Code Best Practice Guide (82 Tips)` - 6 edges
10. `Claude Code Tips` - 6 edges

## Surprising Connections (you probably didn't know these)
- `Context Engineering` --conceptually_related_to--> `Context Files`  [INFERRED]
  wiki/concepts/agentic-engineering.md → wiki/concepts/context-files.md
- `Context Files` --semantically_similar_to--> `OpenClaw`  [INFERRED] [semantically similar]
  wiki/concepts/context-files.md → wiki/entities/openclaw.md
- `Boris Cherny` --conceptually_related_to--> `Claude Code Best Practice`  [INFERRED]
  wiki/entities/claude-code-authors.md → wiki/entities/claude-code-best-practice.md
- `Vibe Coding` --semantically_similar_to--> `Claude Code Generation Best Practices`  [INFERRED] [semantically similar]
  wiki/topics/vibe-coding.md → wiki/topics/claude-code-gen.md
- `CLAUDE.md Writing Guide` --semantically_similar_to--> `AGENTS.md (Codex equivalent of CLAUDE.md)`  [INFERRED] [semantically similar]
  wiki/topics/claude-md-guide.md → wiki/topics/codex-migration-guide.md

## Hyperedges (group relationships)
- **Context Files + Coding Agent + CLAUDE.md form the AI Agent instruction system** — contextfiles_concept, codingagent_concept, claudemd_concept, agents_concept [EXTRACTED 0.95]
- **CLAUDE.md Knowledge Cluster** — claude_md_concept, claude_md_guide_topic, claude_md_mechanisms_topic [EXTRACTED 0.95]
- **Multi-Agent Methodology** — agentic_engineering_concept, agent_team_architecture_topic, prompt_mentor_guide_topic, vibe_coding_topic [INFERRED 0.80]
- **Claude Code Tooling Ecosystem** — everything_claude_code_entity, claude_plugins_official_entity, claude_code_tips_topic, codex_migration_guide_topic [INFERRED 0.75]

## Communities

### Community 0 - "Plugin & Agent Ecosystem"
Cohesion: 0.25
Nodes (14): AGENTS.md, CLAUDE.md, claude-md-management Plugin, Claude Plugins Official, feature-dev Plugin, skill-creator Plugin, Coding Agent, Context Files (+6 more)

### Community 1 - "Agentic Engineering Methodology"
Cohesion: 0.22
Nodes (11): Commandify Everything, Agentic Engineering, Context Engineering, Modular Rules Architecture, PRD First Development, Subagent Orchestration, Claude Code Commands, Claude Code Best Practice (+3 more)

### Community 2 - "CLAUDE.md Config & Codex"
Cohesion: 0.33
Nodes (7): CLAUDE.md Concept, ETH Zurich Finding: CLAUDE.md Reduces Task Success, CLAUDE.md Writing Guide, CLAUDE.md Mechanisms, AGENTS.md (Codex equivalent of CLAUDE.md), Codex - Claude Code Command Mapping, Codex Migration Guide from Claude Code

### Community 3 - "Claude Code Tooling"
Cohesion: 0.47
Nodes (6): Claude Code Generation Best Practices, Claude Code Tips, Claude Plugins Official, Everything Claude Code, Git Branch Operations, Vibe Coding

### Community 4 - "Key Contributors"
Cohesion: 0.4
Nodes (5): Affaan Mustafa, Alex Albert, Boris Cherny, Claude Code Authors, Riley Goodside

### Community 5 - "Session Management Patterns"
Cohesion: 0.4
Nodes (5): Claude Code - Context Management (40%/60% rule), Claude Code - Hooks (PostToolUse/Stop/PreToolUse), Claude Code - Settings Priority Hierarchy, Claude Code Best Practice Guide (82 Tips), Everything Claude Code - Session Hooks

### Community 6 - "Subagent & Multi-Agent"
Cohesion: 0.4
Nodes (5): AI Agent Principles, Claude Code - Subagent Isolation Pattern, Everything Claude Code - Subagent Orchestration, OpenCode + Oh My OpenCode, Oh My OpenCode Agent System

### Community 7 - "Workflow Architecture"
Cohesion: 0.5
Nodes (5): Agent Team Architecture, Agentic Engineering, AI-Driven Git Workflow, Codex Migration Guide, Prompt Mentor Agent Guide

### Community 8 - "Knowledge Base & RAG"
Cohesion: 0.5
Nodes (4): Karpathy SOP - Graphify Phase (50+ articles), Rationale: Why Graphify after 50 articles, Karpathy Knowledge Base + Graphify SOP, RAG System Optimization

## Knowledge Gaps
- **19 isolated node(s):** `Wiki Schema`, `PRD First Development`, `Modular Rules Architecture`, `Affaan Mustafa`, `Alex Albert` (+14 more)
  These have ≤1 connection - possible missing edges or undocumented components.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Wiki Index` connect `Plugin & Agent Ecosystem` to `Agentic Engineering Methodology`, `CLAUDE.md Config & Codex`, `Claude Code Tooling`, `Key Contributors`, `Workflow Architecture`?**
  _High betweenness centrality (0.641) - this node is a cross-community bridge._
- **Why does `Everything Claude Code` connect `Claude Code Tooling` to `Plugin & Agent Ecosystem`, `CLAUDE.md Config & Codex`, `Session Management Patterns`, `Subagent & Multi-Agent`, `Workflow Architecture`?**
  _High betweenness centrality (0.257) - this node is a cross-community bridge._
- **Why does `Claude Code Best Practice` connect `Agentic Engineering Methodology` to `Plugin & Agent Ecosystem`, `Key Contributors`?**
  _High betweenness centrality (0.136) - this node is a cross-community bridge._
- **Are the 2 inferred relationships involving `Everything Claude Code` (e.g. with `Claude Plugins Official` and `Claude Code Tips`) actually correct?**
  _`Everything Claude Code` has 2 INFERRED edges - model-reasoned connections that need verification._
- **What connects `Wiki Schema`, `PRD First Development`, `Modular Rules Architecture` to the rest of the system?**
  _19 weakly-connected nodes found - possible documentation gaps or missing edges._