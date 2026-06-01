# Wiki 内容审计报告 (2026-06-01)

## 审计概览

- **机械检查**：全部通过。INDEX 差异、断链、孤立页、frontmatter 缺失、超长页 均为 0 项。
- **内容审计**：共扫描 28 个页面，发现问题 **86 项**，分布于 24 个页面。
- **严重度分布**：高危 30 · 中危 32 · 低危 24。
- **无问题页面（4）**：`entities/github-reference.md`、`topics/claude-code-tips.md`、`topics/claude-code-gen.md`（stub）、`topics/rag-optimization.md`（stub）。
- **核心风险主题**：
  1. **`teamcreate` 命令幻觉**（2 个页面 + 源文件）——官方不存在该命令，agent teams 经自然语言触发。
  2. **commands 已并入 skills**（多页未更新）——v2.1.3 起 `.claude/commands/` 与 skills 等价。
  3. **ETH Zurich 论文（arXiv 2602.11988）被过度简化**（4 个页面）——未区分 LLM 生成 vs 人工编写。
  4. **第三方飞书/社区文档转录错误**（`claude-plugins-official`、`everything-claude-code`、`opencode`、`openclaw`）。
  5. **`--full-auto` / `paths:` / `@import` 等版本敏感断言失准**。

---

## 一、高危（30 项）

### `concepts/agentic-engineering.md`

**[H1] L32-L34**
- 摘录：`### 3. Commandify Everything` / 同一个 Prompt 输入超过两次，就把它变成一个 Command（Markdown 文件定义工作流）。
- 依据：官方已将 commands 合并进 skills。`https://code.claude.com/docs/en/slash-commands`：“Custom commands have been merged into skills… both create /deploy and work the same way.”
- 动作：改写为 “Skillify Everything”，说明应创建 `.claude/skills/<name>/SKILL.md`，并注明 `.claude/commands/` 仅向后兼容。

### `concepts/agents.md`

**[H2] L9-L10**
- 摘录：`summary: 与 CLAUDE.md 本质相同的 AI Agent 上下文配置文件，由 ETH Zurich 研究中使用此名称进行测试。`
- 依据：官方文档 `https://code.claude.com/docs/en/best-practices` 从未提及 AGENTS.md；Claude Code 仅支持 CLAUDE.md 系列。
- 动作：改写——明确 AGENTS.md 是 ETH Zurich 研究为跨工具对比选用的通用名称，非 Claude Code 官方支持文件名。

**[H3] L15-L17**
- 摘录：与 `[[concepts/claude-md|CLAUDE.md]]` 本质相同的 AI Agent 上下文配置文件，由 ETH Zurich 研究中使用此名称进行测试。
- 依据：同上；读者可能误以为可在 Claude Code 项目中创建 AGENTS.md 并期望生效。
- 动作：改写——说明这是学术研究中的命名选择，实际使用 Claude Code 时应用 CLAUDE.md。

### `concepts/claude-md.md`

**[H4] L24**
- 摘录：工作目录以下的子目录 CLAUDE.md 懒加载——Claude 读取该目录文件时才触发。
- 依据：源 `raw/ai-chat/claude_md_complete_guide.md` 系 AI 对话生成；WebSearch/WebFetch 未找到 Anthropic 官方文档确认懒加载机制。
- 动作：标记为未经官方验证特性，或删除该断言直到找到官方来源。（注：该机制在其他页 `claude-md-best-practices.md`/`claude-md-mechanisms.md` 中有官方依据，建议交叉核对后统一。）

**[H5] L24**
- 摘录：可通过 `@import` 语法强制加载。
- 依据：WebSearch 未在官方文档找到 `@import` 语法确认，仅出现在 AI 生成的源文件中。
- 动作：标记为未经官方验证，或删除直到补齐官方来源。

**[H6] L27**
- 摘录：ETH Zurich 研究：盲目添加 CLAUDE.md 反而降低任务成功率、成本增加 20%+——问题出在架构层面而非内容质量。
- 依据：arXiv 2602.11988 实际结论：LLM 生成文件 8 项测试 5 项更差、成本 +20-23%；开发者编写文件在 AGENTbench 上 +4%、成本 +19%。原文未区分两类。
- 动作：改写为准确反映研究结论，区分 LLM 生成 vs 人工编写，并注明研究针对 agents.md 而非专门针对 Claude Code。

### `concepts/context-files.md`

**[H7] L23**
- 摘录：不同工具使用不同文件名：Claude Code 用 CLAUDE.md，OpenCode 用 AGENTS.md。
- 依据：AGENTS.md 是 OpenAI 主导的开放格式（`github.com/openai/agents.md`），未找到名为 “OpenCode” 的主流编码工具采用它。
- 动作：删除 “OpenCode” 引用，改为：Claude Code 用 CLAUDE.md，Cursor/Copilot 等支持 AGENTS.md（OpenAI 主导开放格式），部分项目用符号链接统一。

### `entities/claude-plugins-official.md`

**[H8] L1-L42**
- 摘录：整个页面将 `claude-plugins-official` 作为实体名称和官方插件集合。
- 依据：`github.com/anthropics/claude-code/tree/main/plugins` 无此名称，系飞书第三方文档的误导性标题。
- 动作：删除整页，或重命名为 `claude-code-bundled-plugins` 并在开头说明这不是官方命名集合。

**[H9] L23**
- 摘录：`skill-creator | 创建自定义技能 | 技能`
- 依据：官方 13 个插件中无 skill-creator；官方有 `plugin-dev`（提供 `/plugin-dev:create-plugin` 与 skill-reviewer agent）。
- 动作：删除此行或改为 `plugin-dev`。

**[H10] L24**
- 摘录：`claude-md-management | 管理 CLAUDE.md 文件 | 技能`
- 依据：官方插件无此项；`code-review` 插件含 CLAUDE.md 合规检查 agent，但非独立管理插件。
- 动作：删除此行。

### `entities/everything-claude-code.md`

**[H11] L35-L38**
- 摘录：`| Agents | ✅ | 13 个 | / | Skills | ✅ | 28 个 | / | Commands | ✅ | 24 个 |`
- 依据：用户已装 v1.10.0，`plugin.json`：“38 agents, 156 skills, 72 legacy command shims”。
- 动作：更新为 v1.10.0 实际数量，或标注此为早期版本数据并加版本号。

**[H12] L104-L110**
- 摘录：`| 1. PLAN | /everything-claude-code:plan | … |`
- 依据：`COMMANDS-QUICK-REF.md`：“Type `/` … to invoke”，示例均为 `/plan`、`/tdd` 无前缀格式。
- 动作：删除所有命令中的 `/everything-claude-code:` 前缀，改为 `/plan`、`/checkpoint create` 等。

**[H13] L128-L143**
- 摘录：`**插件命令（需前缀 /everything-claude-code:）：**` 表格命令列写作 `plan`。
- 依据：实际为 `/plan` 而非带前缀；命令列应含 `/`。
- 动作：删除“需前缀”说明，命令列改为 `/plan`、`/checkpoint create` 等完整格式。

**[H14] L180**
- 摘录：`| 忘记命令前缀 | 所有命令必须加 /everything-claude-code: 前缀 |`
- 依据：实际命令无需此前缀，与使用方式完全相反。
- 动作：删除整行。

### `entities/openclaw.md`

**[H15] L21**
- 摘录：跨设备同步：Mac 备忘录（推荐）/ Notion / Obsidian。
- 依据：源仓库 `github.com/Tornadopp/openclaw-learn` 中 Obsidian 零提及；Notion 仅作 API 数据同步出现。
- 动作：删除 Obsidian；将 Notion 标注为 “API 数据同步”；改写为 “Mac 备忘录（推荐，Mac/iPhone 无缝同步）/ Notion API 数据同步”。

### `entities/opencode.md`

**[H16] L16**
- 摘录：`Oh My OpenCode(OMO)是其全能增强插件`
- 依据：项目已更名为 `oh-my-openagent`，仓库描述 “previously oh-my-opencode”（`github.com/code-yeongyu/oh-my-openagent`）。
- 动作：标注已更名为 oh-my-openagent，保留历史名称供检索。

**[H17] L22**
- 摘录：`bunx oh-my-opencode install`
- 依据：项目更名后 npm 包名/安装方式可能同步变更。
- 动作：核实 oh-my-openagent 当前安装命令并更新，或标注不确定性。

### `topics/agent-team-architecture.md`

**[H18] L22**
- 摘录：Claude Code 的 `teamcreate` 命令。
- 依据：`https://code.claude.com/docs/en/agent-teams` 说明 agent teams 经自然语言提示创建；命令列表中无 teamcreate。
- 动作：删除所有 teamcreate 提及，改为 “通过自然语言提示创建 agent team”。

**[H19] L108-L112**
- 摘录：Claude Code 提供了 `teamcreate` 前缀指令，强制 CLI 进入多 Agent 协作调度模式……核心触发机制：`teamcreate` 前缀。
- 依据：官方文档：agent teams 为实验性功能，需在 settings.json 启用 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`，再以自然语言创建。
- 动作：完全重写——说明实验性功能、需手动启用、自然语言创建、需 v2.1.32+。

**[H20] L100**
- 摘录：`Anthropic Claude Code teamcreate`
- 依据：teamcreate 不存在。
- 动作：改为 “Anthropic Claude Code Agent Teams”，删除 teamcreate 引用。

**[H21] L10**
- 摘录：`updated: 2026-05-21`
- 依据：更新日期早于 agent teams 功能官方发布，但页面断言该命令已存在；源 `raw/ai-chat/claude-code-agent-team-guide.md` 含错误断言。
- 动作：核实源文件；若源错误，页面顶部加 `contested: true` 并注明与官方文档冲突。

### `topics/ai-agent-principles.md`

**[H22] L1-L23**
- 摘录：整个页面仅含标题、frontmatter 和 3 个 wikilink，无实质内容。
- 依据：源文件 `raw/notion/articles/ai-agent-principles-and-practice.md` 仅 12 行元数据，无正文。
- 动作：删除此页，或标记为 stub（`confidence: none`, `status: incomplete`），或从其他源补内容。

### `topics/claude-code-teamcreate-practice.md`

**[H23] L12**
- 摘录：必须使用 `teamcreate` 作为前缀，强制 CLI 进入多 Agent 协作调度模式。
- 依据：官方文档无 teamcreate；agent teams 经自然语言（如 “Create an agent team to…”）触发。
- 动作：删除 teamcreate 前缀声明，改为自然语言请求。

**[H24] L27-L41**
- 摘录：场景一至七的所有提示词模板均以 `teamcreate` 开头。
- 依据：官方示例 “Create an agent team to explore this from different angles”。
- 动作：重写全部 7 个场景为自然语言格式，保留 Objective/Context/Workflow 结构作为描述的一部分。

**[H25] L5-L7**
- 摘录：`sources: raw/ai-chat/claude-code-agent-team-guide.md, raw/ai-chat/mavis-multi-agent-architecture.md`
- 依据：源文件 L12 明写 “必须使用 teamcreate 作为前缀”，页面忠实转录了错误源。
- 动作：在 raw/ 源 frontmatter 加纠错注释（指向官方 agent-teams 文档），wiki 页基于官方文档重写，不再依赖此错误源。

**[H26] L8**
- 摘录：`updated: 2026-05-21`
- 依据：更新日期早于功能官方文档发布（2026-02 后），内容与官方机制不符。
- 动作：frontmatter 加 `contested: true` 与 `contradictions: 官方文档不存在 teamcreate 命令，agent teams 通过自然语言触发`，降 confidence 至 low，考虑移至 `_archive/` 或彻底重写。

### `topics/claude-md-best-practices.md`

**[H27] L31-L32**
- 摘录：规则按关注点拆 `.claude/rules/`……配合 `paths` frontmatter 触发。
- 依据：GitHub issue #16038/#17204 报告 `paths:` 字符串与数组语法静默失败，`globs:` 更可靠。
- 动作：改写为 “配合 `paths` 或 `globs` frontmatter 触发（注意 `paths:` 存在已知 bug，建议用 `globs:` 或 YAML 数组形式）”。

**[H28] L70-L72**
- 摘录：`│ ├── typescript.md ← paths: **/*.ts`
- 依据：同 #16038/#17204，字符串形式 `paths:` 静默失败。
- 动作：示例注释改为 `← globs: **/*.ts` 或 YAML 数组形式 `paths:`，并加警告说明。

### `topics/codex-command-mapping.md`

**[H29] L52**
- 摘录：`| --full-auto | 低摩擦预设：workspace-write + on-request |`
- 依据：Codex 源码 `codex-rs/cli/src/main.rs` 测试 `full_auto_no_longer_parses_at_top_level()` 断言顶层解析失败；弃用提示指向 `--sandbox workspace-write`。
- 动作：删除该预设条目或注明已废弃，改为推荐 `-s workspace-write`（需审批再配 `-a on-request`）。

**[H30] L85, L99**
- 摘录：`codex --full-auto "实现登录页错误提示…"` / `codex --full-auto "复现并修复这个 bug…"`
- 依据：顶层 `codex --full-auto` 已不再解析，用户复制即报错。
- 动作：改为 `codex -s workspace-write "…"`（交互式）或 `codex exec -s workspace-write -a on-request "…"`。

---

## 二、中危（32 项）

### `concepts/agentic-engineering.md`

**[M1] L2-L8**
- 摘录：`title: Agentic 工程技术 / summary: 将 AI 编程助手从"聊天工具"升级为"可重复工程系统"…`
- 依据：官方用 “agentic coding environment”/“agentic loop”，无 “agentic engineering” 正式术语。
- 动作：保留+标记——页面开头注明这是社区/教学用语而非官方术语。

**[M2] L28-L30**
- 摘录：具体任务规则拆到独立文件（`rules/api.md`）。
- 依据：官方推荐用 skills 机制（`.claude/skills/`）承载按需加载的领域知识。
- 动作：改写示例路径为 `.claude/skills/api-conventions/SKILL.md`。

### `concepts/agents.md`

**[M3] L21-L22**
- 摘录：ETH Zurich 研究同时测试了 CLAUDE.md 和 AGENTS.md，结论相同。
- 依据：论文针对不同工具用不同文件名（Codex/Qwen 用 AGENTS.md，Claude Code 用 CLAUDE.md），非同时测试两者。
- 动作：改写——澄清研究方法是按工具选用对应文件名。

**[M4] L10**
- 摘录：`confidence: medium`
- 依据：页面核心断言在 Claude Code 语境下错误，属可验证的版本敏感断言。
- 动作：降 confidence 至 low，或加 `contested: true` 并在 contradictions 中说明差异。

### `concepts/claude-md.md`

**[M5] L22**
- 摘录：四个层级：托管策略层（企业全员）、用户层（~/.claude/CLAUDE.md）、项目层（./CLAUDE.md）、本地层（./CLAUDE.local.md）。
- 依据：源文件列出托管策略层具体系统路径（macOS `/Library/Application Support/ClaudeCode/CLAUDE.md` 等），项目层含 `./.claude/CLAUDE.md` 选项，页面未体现。
- 动作：补充托管策略层系统路径，明确项目层含 `./CLAUDE.md` 与 `./.claude/CLAUDE.md` 两种。

**[M6] L25**
- 摘录：配套机制 `.claude/rules/` 通过 paths glob 实现路径作用域规则（确定性触发）。
- 依据：`mer.vin/2026/05` 确认存在 path-scoped 规则但无实现细节；源为 AI 对话生成。
- 动作：保留但标注为基于社区实践、非官方文档确认的详细行为。

### `concepts/coding-agent.md`

**[M7] L21**
- 摘录：代表工具：Claude Code、Cursor、OpenCode、Codex。
- 依据：“OpenCode” 无法官方核实；“Codex” 应明确为 OpenAI Codex（`openai.com/codex/`）。
- 动作：改写为 “Claude Code、Cursor、OpenAI Codex、Windsurf 等”，删除无法核实的 OpenCode。

**[M8] L23**
- 摘录：支持子智能体编排，将复杂任务拆解给多个专职 Agent。
- 依据：仅 Claude Code 明确支持 subagents；Cursor/早期 Codex 无官方说明。
- 动作：改写为 “部分工具（如 Claude Code）支持子智能体编排…”。

### `concepts/context-files.md`

**[M9] L22**
- 摘录：ETH Zurich 研究：有上下文文件时 AI 任务成功率反而下降，成本增加 20%+。
- 依据：`arxiv.org/html/2602.11988v1` 区分 LLM 生成（AGENTbench -3%，SWE-bench Lite -0.5%）vs 人工编写（+4%）。
- 动作：改写为区分两类、明确各自成功率与 +20% 成本，并注明研究建议省略 LLM 生成文件。

### `entities/claude-plugins-official.md`

**[M10] L25**
- 摘录：`code-simplifier | 简化优化代码，结合 code style 规范 | 子代理`
- 依据：`code-simplifier` 是 `pr-review-toolkit` 插件内的一个 agent，非独立插件。
- 动作：删除此行或改为注释说明其属 pr-review-toolkit；形式应为 “agent”。

**[M11] L31**
- 摘录：`ralph-loop | 指定循环次数进行 AI 编程 | 插件`
- 依据：官方插件名为 `ralph-wiggum`，`/ralph-loop` 是其命令。
- 动作：改为 “ralph-wiggum | …（命令：/ralph-loop）| 插件”。

**[M12] L21-L31**
- 摘录：核心插件表格缺少多个官方插件。
- 依据：官方含 agent-sdk-dev、claude-opus-4-5-migration、code-review、explanatory-output-style、learning-output-style、plugin-dev、security-guidance 等。
- 动作：补全缺失插件，或在 summary 说明这是部分列表。

### `entities/everything-claude-code.md`

**[M13] L23-L24**
- 摘录：`/plugin marketplace add affaan-m/everything-claude-code` / `/plugin install everything-claude-code@everything-claude-code`
- 依据：与已装 v1.10.0 的兼容性需核实；README 未明确展示完整示例。
- 动作：无法核实——访问最新官方文档/仓库 README 确认当前安装流程。

### `entities/openclaw.md`

**[M14] L19**
- 摘录：GitHub 项目管理：自动分析 README 和代码结构，提取技术栈和核心功能。
- 依据：源仓库仅称 “自动追踪、生成学习笔记”，无 README/代码结构分析说明。
- 动作：改写为 “自动追踪 GitHub 项目并生成学习笔记”，删除未经证实的具体功能。

### `entities/opencode.md`

**[M15] L16**
- 摘录：OpenCode 是开源 AI 编程助手（60K+ Stars）。
- 依据：`github.com/anomalyco/opencode` 显示 168K stars；`opencode.ai` 显示 160K。
- 动作：更新为 168K / 160K+ stars。

**[M16] L25**
- 摘录：配置文件：`~/.config/opencode/opencode.json` 和 `~/.config/opencode/oh-my-opencode.json`。
- 依据：项目更名后配置文件名可能从 oh-my-opencode.json 变为 oh-my-openagent.json。
- 动作：核实当前路径，或标注 “更名前为 oh-my-opencode.json”。

**[M17] L42-45**
- 摘录：命令表格：`/ralph-loop`、`/init-deep`、`/refactor`、`/git-master`。
- 依据：官方核心命令为 ultrawork/ulw、`/ulw-loop`、`/start-work`、`/init-deep`，未见 `/refactor`、`/git-master`。
- 动作：核实 `/refactor`、`/git-master` 是否仍存在；补充 ultrawork/ulw 与 `/start-work`。

### `topics/agent-team-architecture.md`

**[M18] L129**
- 摘录：架构视角的提醒：teamcreate 的本质是 prompt 编排。
- 依据：agent teams 确为自然语言编排，但非通过 teamcreate 命令。
- 动作：改为 “Claude Code agent teams 的本质是 prompt 编排”。

**[M19] L148**
- 摘录：落到 Claude Code：在 teamcreate 的 Constraints 里写明。
- 依据：约束写在创建团队的提示词中，非命令参数。
- 动作：改为 “在创建 agent team 的提示词中明确约束条件”。

**[M20] L158**
- 摘录：工程实现：teamcreate 当前还做不到 runtime 级别的自动 verify-retry 循环。
- 依据：该限制属功能本身而非不存在的 teamcreate 命令。
- 动作：改为 “Claude Code agent teams 当前还做不到 runtime 级别的自动 verify-retry 循环”。

**[M21] L177**
- 摘录：teamcreate 目前还在工具和同事之间。
- 依据：teamcreate 不存在。
- 动作：改为 “Claude Code agent teams 目前还在工具和同事之间”。

**[M22] L135**
- 摘录：详见 `topics/claude-code-teamcreate-practice`。
- 依据：引用页名含不存在的 teamcreate 命令；该页本身也需修正（见 H23-H26）。
- 动作：同步修正被引页，此处改为更准确的页名。

### `topics/ai-driven-git-workflow.md`

**[M23] L78-L103**
- 摘录：层级 4 —— 自定义 Slash Commands…`.claude/commands/`…层级 5 —— Skills…Skills 与 Commands 的区别见 `[[concepts/agentic-engineering]]`。
- 依据：v2.1.3（2026-01）起自定义 slash 命令已并入 skills，二者生成相同 `/` 命令、行为一致；`.claude/commands/` 仅向后兼容。
- 动作：保留+标记——补一句说明合并事实，弱化 L103 “区别” 措辞。

### `topics/claude-code-teamcreate-practice.md`

**[M24] L18**
- 摘录：架构背景与核心机制见 `[[topics/agent-team-architecture]]`。
- 依据：被引页也含 teamcreate 错误，存在交叉污染。
- 动作：审计并同步修正 `topics/agent-team-architecture.md`（见 H18-H21）。

### `topics/claude-md-best-practices.md`

**[M25] L24**
- 摘录：**HTML 注释**：`<!-- ... -->` 注入前被去除，可写给人看的说明而不消耗 token。
- 依据：官方说明仅块级 HTML 注释被去除，代码块内注释保留，Read 工具直接打开时注释仍可见。
- 动作：补全为 “块级 HTML 注释注入前被去除；代码块内注释保留；Read 打开时仍可见”。

**[M26] L96**
- 摘录：`| 每次 session 加载 | 全量 | MEMORY.md 前 200 行或 25KB |`
- 依据：官方——项目根及以上 CLAUDE.md 启动时全量加载，子目录懒加载。
- 动作：改为 “项目根及以上全量；子目录懒加载 | MEMORY.md 前 200 行或 25KB”。

### `topics/claude-md-guide.md`

**[M27] L17, L21**
- 摘录：CLAUDE.md 上下文文件反而会降低 AI 编程 Agent 的任务成功率，并增加推理成本超过 20%……8 项测试中 5 项“无 CLAUDE.md”表现更好。
- 依据：论文区分 LLM 生成（略降）vs 人工编写（约 +4%）；一刀切表述与原文冲突。
- 动作：改写——明确区分两类上下文文件的不同结论。

**[M28] L22**
- 摘录：更强的模型并不能生成更好的上下文文件——问题出在架构层面，而非内容质量。
- 依据：论文归因于文档冗余/未提供有效概览（去除其它文档后 LLM 生成文件 +2.7%），无 “架构 vs 内容质量” 对立框架。
- 动作：保留+标记——注明系飞书源二次解读，论文原因为 “与现有文档冗余”。

**[M29] L22, L27**
- 摘录：（L22）问题出在架构层面，而非内容质量……（L27）上下文文件的价值取决于内容质量和使用方式。
- 依据：页面内部 L22 与 L27 结论方向相反。
- 动作：改写——统一口径，去除矛盾。

### `topics/claude-md-mechanisms.md`

**[M30] L55**
- 摘录：`@import` 语法……递归最多 5 层。
- 依据：`https://code.claude.com/docs/en/memory`：“maximum depth of four hops.”
- 动作：改为 “递归最多 4 层（four hops）”。

### `topics/codex-migration-guide.md`

**[M31] L104**
- 摘录：`| 记忆持久化 | Hooks（session-start/end） | 暂无内置，需 AGENTS.md 补充 |`
- 依据：OpenAI 2026-04-16 推出 Codex 持久记忆预览（`openai.com/index/codex-for-almost-everything/`），晚于本页 updated=2026-05-21。
- 动作：改为 “Codex 已推出持久记忆预览（2026-04 起分阶段放量），AGENTS.md 仍可作显式补充”。

### `topics/vibe-coding.md`

**[M32] L85-L87**
- 摘录：`/plan`、`/tdd`、`/orchestrate` 等命令对应到四步方法论。
- 依据：官方仓库已将 `/tdd`、`/orchestrate` 退役进 legacy-command-shims（提示改用 tdd-workflow / dmux-workflows / multi-workflow），仅 `/plan` 维护中。
- 动作：改写为更宽泛表述（`/plan` + tdd-workflow / multi-workflow 等 skill），并同步更新 `entities/everything-claude-code`。

---

## 三、低危（24 项）

### `concepts/agentic-engineering.md`

**[L1] L41-L43** — 摘录：“循环验证……最多 3 轮”。依据：官方未提 3 轮硬限制（`code.claude.com/docs/en/sub-agents`）。动作：注明这是建议最佳实践而非系统限制，或删除具体数字。

### `concepts/coding-agent.md`

**[L2] L22** — 摘录：通过上下文文件（CLAUDE.md 等）获取项目级别指令。依据：CLAUDE.md 是 Claude Code 特有约定，非通用机制。动作：改写为 “通过上下文文件获取项目级别指令（如 Claude Code 的 CLAUDE.md、Cursor 的 .cursorrules 等）”。

### `entities/claude-plugins-official.md`

**[L3] L6** — 摘录：`sources: - 'raw/feishu/claude-plugins-official-guide.md'`。依据：单一第三方飞书来源，未经官方验证。动作：confidence 从 medium 改为 low，summary 加 “需与官方文档核对”。

### `entities/everything-claude-code.md`

**[L4] L179** — 摘录：`| 命令报错 "Skill is not a prompt-based skill" | 使用 Task 工具或 EnterPlanMode 替代 |`。依据：v1.10.0 架构重构，错误/方案可能不再适用。动作：保留+标记 “早期版本可能遇到”，或标注需验证。

### `entities/git-branch.md`

**[L5] L25** — 摘录：`git switch -c feature/xxx origin/feature/xxx`。依据：`git-scm.com/docs/git-switch`——plain form 不保证上游跟踪，取决于 `branch.autoSetupMerge`。动作：保留+标记，补注或改用 `--track` / `git switch feature/xxx`。

### `entities/openclaw.md`

**[L6] L20** — 摘录：论文笔记：PDF 自动解析，生成结构化笔记。依据：源用 “提取关键信息”“自动总结”，无 “结构化笔记”。动作：改写为 “PDF 自动解析、提取关键信息并总结”。

### `entities/opencode.md`

**[L7] L29-36** — 摘录：Agent 系统表格（Sisyphus/Oracle/Librarian/Prometheus/Metis/Momus）。依据：README 另含 Hephaestus、Explore、Multimodal Looker。动作：补全或标注 “部分核心 agent”。

**[L8] L8-9** — 摘录：`updated: 2026-05-21`。依据：内容含多处过时信息，与 updated 日期不符（仓库最近更新 2026-05-31）。动作：更新内容后同步 updated 字段。

### `topics/agent-team-architecture.md`

**[L9] L184** — 摘录：不要为了看起来高级而对所有任务都套 teamcreate。依据：建议合理但命令名错误。动作：改为 “……都使用 agent teams”。

### `topics/ai-driven-git-workflow.md`

**[L10] L72-L76** — 摘录：`@Claude review`。依据：官方约定为小写 `@claude`，且需先 `/install-github-app` 安装 GitHub App/Action。动作：保留+标记，规范为 `@claude` 并补注前置步骤。

### `topics/claude-md-best-practices.md`

**[L11] L83** — 摘录：运行 `/memory` 查看当前 session 已加载文件列表。依据：官方——`/memory` 列出 CLAUDE.md/CLAUDE.local.md/rules 文件并提供打开链接。动作：改为 “列出当前 session 已加载的 CLAUDE.md、CLAUDE.local.md 和 rules 文件”。

**[L12] L87** — 摘录：使用 `InstructionsLoaded` hook 记录日志。依据：该 hook 仅异步观察性，无法阻止/修改加载。动作：补充 “（仅观察性，不可阻止加载）”。

### `topics/claude-md-guide.md`

**[L13] L2, L13, L15-L27** — 摘录：title 为 “Claude.md 新手指南”，正文全是研究批评，无编写指引。依据：源原标题含 “如何编写”，摄入时只取研究报告段落。动作：改标题为贴合内容的表述，或补实操指南。

**[L14] L21** — 摘录：8 项测试中 5 项“无 CLAUDE.md”表现更好。依据：该计数来自飞书源，论文 abstract 仅按数据集给出变化，无 “8 选 5” 口径。动作：核对论文正文表格后保留，或标注为源的概述。

### `topics/claude-md-mechanisms.md`

**[L15] L63-L69** — 摘录：`CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ./src/api`。依据：`--add-dir` 官方定位为访问主工作目录之外的目录，示例却指向工作目录内子目录。动作：示例改为外部目录（如 `../shared-config`），并说明用途。

### `topics/codex-command-mapping.md`

**[L16] L54** — 摘录：`| -a untrusted / on-request / never | 审批策略 |`。依据：枚举遗漏 on-failure（源码中已标 DEPRECATED）。动作：保留+标记——可补一句 on-failure 已废弃。

### `topics/codex-migration-guide.md`

**[L17] L46, L29** — 摘录：`codex --full-auto "修复当前测试失败…"`。依据：第三方资料分歧，部分标注 deprecated（建议改 `-s workspace-write`）。动作：保留+标记——核对官方 CLI reference 后补充等价用法说明。

**[L18] L99** — 摘录：`| 规则文件 | CLAUDE.md + rules/ | AGENTS.md |`。依据：`rules/` 非 Claude Code 标准规则机制，官方为 CLAUDE.md + `@import`；源映射表仅写 CLAUDE.md↔AGENTS.md。动作：去掉 `+ rules/`，或改为 “CLAUDE.md（可经 @import 拆分）”。

### `topics/karpathy-knowledge-base-sop.md`

**[L19] L16** — 摘录：本 SOP 基于 Andrej Karpathy 提出的 “AI 图书管理员” 方法论。依据：“图书管理员” 系 linux.do 社区作者对 Karpathy 想法的转述，非原话（原推措辞未能逐字核实）。动作：保留+标记——注明系社区转述比喻。

**[L20] L37-57** — 摘录：编写 CLAUDE.md 章节。依据：未提官方推荐的 `/init` 生成与 “保持精简” 原则。动作：保留+标记——建议补充 `/init` 生成与精简原则。

### `topics/prompt-mentor-case-study.md`

**[L21] L40-L58** — 摘录：参考代码所在仓库为只读……主流程模板：seller-shopee-spider 现有目录结构。依据：L40-41 与 L56-58、L29 内部自相矛盾（seller-shopee-spider 既是读写目标又列为只读模板）。动作：将 L57 “主流程模板” 修正为 seller-lazada-spider。

### `topics/prompt-mentor-guide.md`

**[L22] L120** — 摘录：并显式调用 Explore / writing-plans / systematic-debugging 三个 skill。依据：Explore 是内置 subagent 类型，非 skill；与本页 L60-61、L90 自相矛盾。动作：改为 “显式调用 Explore 子代理 + writing-plans / systematic-debugging 两个 skill”。

### `topics/tdd-ddd-data-collection.md`

**[L23] L130-L134** — 摘录：`mattpocock/skills@improve-codebase-architecture | 31.8K` 等三条安装量。依据：skill 名与 CLI 命令已核实真实，但安装量为 2026-05-02 时间点快照、会漂移、无 as-of 标注。动作：保留+标记 “截至 2026-05 的快照”，或改为相对量级描述。

### `topics/vibe-coding.md`

**[L24] L87** — 摘录：`memory-bank` 文件夹相当于 Vibe Coding 流程中的 `doc/` 工件存储。依据：官方 README 无 memory-bank 特性，最接近的是 hooks/memory-persistence。动作：保留+标记（松散类比）；若更新建议指向 memory-persistence hooks。

---

## 四、建议删除清单

| 优先级 | 目标 | 理由 |
|--------|------|------|
| 高 | `entities/claude-plugins-official.md`（整页） | 基于误导性第三方飞书文档，实体名不存在；9 项中 4 项插件错误、遗漏 7 项官方插件。删除或彻底重写为 `claude-code-bundled-plugins`。 |
| 高 | `topics/ai-agent-principles.md`（整页） | 空壳 stub，源文件亦无正文。删除或明确标记为 incomplete。 |
| 高 | `topics/claude-code-teamcreate-practice.md`（整页） | 核心机制 `teamcreate` 完全不存在，7 个场景模板全部基于错误语法。建议移至 `_archive/` 或基于官方文档彻底重写。 |
| 高 | `entities/everything-claude-code.md` L180 | 完全错误的故障排查建议（与实际相反），删除整行。 |
| 高 | `entities/claude-plugins-official.md` L23、L24 | `skill-creator`、`claude-md-management` 插件不存在，删除两行。 |
| 中 | `concepts/context-files.md` L23 / `concepts/coding-agent.md` L21 中的 “OpenCode” 引用 | 工具名无法核实，删除或替换。 |
| 中 | `entities/openclaw.md` L21 中的 “Obsidian” | 源仓库零提及，删除。 |

> 注：依据 CLAUDE.md “禁止删除 wiki/ 页面（可改、可移到 `_archive/`）”，整页级处理优先采用 `_archive/` 归档或彻底重写，而非物理删除。

## 五、建议改写清单

**A. 术语/机制版本更新（最高优先）**
- `concepts/agentic-engineering.md` L32-34、`topics/ai-driven-git-workflow.md` L78-103、`topics/vibe-coding.md` L85-87：统一更新 “commands 已并入 skills（v2.1.3）” 心智模型。
- `topics/agent-team-architecture.md`（L22/L100/L108-112/L129/L148/L158/L177/L184/L135/L10）：移除全部 `teamcreate`，重写为 “实验性功能 + 自然语言触发 + 需 v2.1.32+ 手动启用”。
- `entities/everything-claude-code.md` L35-38、L104-110、L128-143：按 v1.10.0 更新组件数量与无前缀命令格式。
- `entities/opencode.md` L16/L22/L25：更新更名（oh-my-openagent）、star 数（168K）、安装命令与配置路径。

**B. ETH Zurich 论文（arXiv 2602.11988）准确化**
- `concepts/claude-md.md` L27、`concepts/context-files.md` L22、`topics/claude-md-guide.md` L17/L21/L22/L27、`concepts/agents.md` L21-22：统一区分 “LLM 生成（略降成功率）vs 人工编写（约 +4%）”，成本 +20% 保留，并注明研究针对 agents.md。

**C. 官方文档细节校正**
- `topics/claude-md-best-practices.md` L24/L31-32/L70-72/L83/L87/L96：HTML 注释规则、`paths→globs` bug、加载机制、`/memory` 与 `InstructionsLoaded` 描述。
- `topics/claude-md-mechanisms.md` L55：`@import` 递归 5 层 → 4 层。
- `topics/codex-command-mapping.md` L52/L85/L99：`--full-auto` 改为 `-s workspace-write`。
- `topics/codex-migration-guide.md` L104：Codex 持久记忆已发布。

**D. confidence / contested 标记调整**
- `concepts/agents.md` L10、`entities/claude-plugins-official.md` L6：confidence 降为 low。
- `topics/claude-code-teamcreate-practice.md`、`topics/agent-team-architecture.md`：加 `contested: true` + `contradictions:`，并在 raw/ 源 frontmatter 加纠错注释。

**E. 未经官方验证特性标记**
- `concepts/claude-md.md` L24（懒加载、@import）：标记为社区/AI 生成来源、待官方核实（注意与 best-practices/mechanisms 页中有官方依据的同类断言交叉对齐）。

**F. 低优先内部一致性/补强**
- `topics/prompt-mentor-case-study.md` L40-58、`topics/prompt-mentor-guide.md` L120：修正内部自相矛盾（仓库归属、Explore 误归为 skill）。
- `entities/git-branch.md` L25、`topics/tdd-ddd-data-collection.md` L130-134、`topics/karpathy-knowledge-base-sop.md` L16/L37-57：补注上游跟踪、安装量 as-of 快照、社区转述与 `/init` 建议。
