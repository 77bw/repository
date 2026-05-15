---
title: CLAUDE.md 完整机制与最佳实践
type: topic
tags: [ai-coding, ai, knowledge-management]
sources: [raw/ai-chat/claude_md_complete_guide.md]
created: 2026-05-15
updated: 2026-05-15
summary: 完整剖析 CLAUDE.md 的文件层级、加载机制、rules 路径作用域、渐进式披露指令与五大最佳实践。
---

# CLAUDE.md 完整机制与最佳实践

## 概述

[[concepts/claude-md]] 是 Claude Code 的上下文配置文件，但其加载机制远比"放在项目根目录"复杂：涉及多层级拼接、子目录懒加载、`.claude/rules/` 路径作用域、渐进式披露指令等多种机制。本主题完整梳理 CLAUDE.md 的运作方式与使用模式，为团队制定项目级 AI 协作规范提供完整参考。

## 分析

### 一、文件层级与作用域

CLAUDE.md 存在四个层级，加载顺序从宽到窄，越靠近启动目录权重越高：

| 层级 | 路径 | 作用范围 | 共享方式 |
|------|------|----------|----------|
| **托管策略层** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 企业全员，不可被任何设置排除 | IT/DevOps 统一部署 |
| **用户层** | `~/.claude/CLAUDE.md` | 当前用户所有项目 | 仅自己 |
| **项目层** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享 | 提交 git |
| **本地层** | `./CLAUDE.local.md` | 仅自己当前项目 | 加入 `.gitignore` |

**优先级规则**：
- 所有文件内容**拼接**（concat），不是覆盖
- 越靠近启动目录越晚读取，权重越高（后读的指令更受重视）
- 同级目录中 `CLAUDE.local.md` 追加在 `CLAUDE.md` 之后

### 二、加载机制

**启动时加载**：从当前工作目录向上遍历目录树，收集每层的 `CLAUDE.md` 和 `CLAUDE.local.md`：

```
启动目录: /project/src/api/
加载顺序（越下越晚读，权重越高）:
  /CLAUDE.md
  /project/CLAUDE.md
  /project/src/CLAUDE.md
  /project/src/api/CLAUDE.md        ← 最晚读
  /project/src/api/CLAUDE.local.md  ← 同级 local 最后追加
```

**子目录懒加载**：工作目录**以下**的子目录 CLAUDE.md 不在启动时加载，只有当 Claude 读取该子目录中任意文件时才触发。这是设计意图，目的是减少不必要的上下文消耗。

**强制加载子目录的三种方法**：

1. **`@import` 语法（推荐）**——在根 CLAUDE.md 中直接引用，启动时立即展开。支持相对/绝对路径，递归最多 5 层。代价是所有内容进入上下文消耗 token。

   ```markdown
   ## 项目文档
   @src/api/CLAUDE.md
   @packages/shared/CLAUDE.md
   ```

2. **`--add-dir` + 环境变量**：

   ```bash
   CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ./src/api
   ```

3. **接受懒加载，主动触发**：session 开始时指示 Claude 读取目标子目录文件即可。

**/compact 后的行为**：

| 文件类型 | /compact 后行为 |
|----------|-----------------|
| 根目录 CLAUDE.md | 自动存活，重新从磁盘读取注入 |
| 子目录嵌套 CLAUDE.md | 不自动重注入，需再次触发 |
| 对话中临时指令 | 丢失，需重新告知 |

### 三、`.claude/rules/` 路径作用域机制

**目录结构**：

```
your-project/
├── CLAUDE.md
└── .claude/
    └── rules/
        ├── code-style.md     # 无 paths → 启动无条件加载
        ├── testing.md        # 有 paths → 匹配文件时加载
        ├── security.md
        └── frontend/
            └── react.md     # 支持子目录递归发现
```

**有 `paths` frontmatter**（路径作用域规则）——Claude 读取匹配路径的文件时才加载：

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "tests/**/*.test.ts"
---

# API 开发规范
- 所有接口必须包含入参校验
- 返回统一错误格式 { code, message, data }
```

**无 `paths` frontmatter**——启动时无条件加载，等价于写在 `.claude/CLAUDE.md` 里。

**常用 glob 模式**：

| 模式 | 匹配范围 |
|------|----------|
| `**/*.ts` | 所有 TypeScript 文件 |
| `src/**/*` | src 目录下所有文件 |
| `src/components/*.tsx` | 特定目录的 React 组件 |
| `**/*.{ts,tsx}` | 多扩展名（花括号展开） |

**用户级 rules**（`~/.claude/rules/`）先于项目 rules 加载，项目 rules 优先级更高。可通过 symlink 跨项目共享：`ln -s ~/shared-claude-rules .claude/rules/shared`。

### 四、渐进式披露指令（CLAUDE.md 自然语言控制）

与文件系统机制的根本区别：

| 维度 | 文件系统机制（rules/paths） | CLAUDE.md 自然语言指令 |
|------|------------------------|------------------------|
| **控制者** | Claude Code 引擎 | Claude 的理解与判断 |
| **触发方式** | 读取文件路径自动匹配 | 读自然语言规则后自行决定 |
| **粒度** | glob 路径匹配 | 语义匹配（"当你需要了解 API 时"） |
| **可靠性** | 确定性触发 | 依赖 Claude 理解准确度 |
| **适合场景** | 技术约束、编码规范 | 业务文档、架构说明、项目状态 |

**四种写法模式**：

1. **文档地图型**——给出按需读取的文档清单：
   ```markdown
   ## 参考文档（按需读取，不要每次对话开始时全部加载）
   - [project_spec.md] — 需要了解完整需求/API 规格时读
   - [docs/architecture.md] — 需要理解系统设计和数据流时读
   - [docs/changelog.md] — 需要查版本历史时读
   ```

2. **任务触发型**——按动作触发：
   ```markdown
   ## 工作流程规则
   - 修改任何 API 接口前，先读 docs/api-contracts.md
   - 遇到数据库相关任务时，先读 docs/schema.md
   - 涉及权限相关逻辑时，先读 docs/auth-design.md
   ```

3. **目录规范声明型**——约束新建文件位置：
   ```markdown
   ## 项目结构规范
   - Slash Command 文件放在 .claude/commands/ 下
   - Skill 文件命名为 SKILL.md，放在 .claude/skills/<skill-name>/ 下
   - Agent 文件放在 .claude/agents/ 下
   ```

4. **条件读取型（最精确的渐进式披露）**——明确每个场景读什么：
   ```markdown
   ## 上下文加载规则

   | 场景 | 读取文件 |
   |------|---------|
   | 新功能开发 | project_spec.md + docs/architecture.md |
   | Bug 修复 | docs/changelog.md + 相关模块 CLAUDE.md |
   | 数据库操作 | docs/schema.md |
   ```

### 五、最佳实践

**写作规范**：
- **大小**：每个 CLAUDE.md 控制在 200 行以内；超出后 Claude 遵从度下降
- **结构**：用 Markdown headers 和 bullet 分组，结构越清晰遵从越好
- **具体性**：写可验证的指令（✅ "使用 2 空格缩进" / ❌ "格式化好代码"）
- **HTML 注释**：`<!-- ... -->` 注入前被去除，可写给人看的说明而不消耗 token

**两种机制的选择**：

```
规则是"针对某种文件类型的编写规范"
  → .claude/rules/ + paths（确定性触发）
  例：TypeScript 规范、测试文件规范、SQL 规范

规则是"针对某个模块/功能域的背景知识"
  → 子目录 CLAUDE.md（懒加载）
  例：API 模块的设计背景、前端组件规范

规则是"任何时候都要遵守的全局约定"
  → 根 CLAUDE.md 或无 paths 的 rules 文件
  例：代码提交规范、命名约定

规则是"某些场景下才需要的业务/架构文档"
  → CLAUDE.md 自然语言指令（渐进式披露）
  例：项目规格、架构设计、API 合约
```

**推荐完整目录结构**：

```
project/
├── CLAUDE.md                    ← 全局：架构总览、build 命令、文档地图
├── CLAUDE.local.md              ← 个人：沙箱 URL、测试数据（gitignore）
├── src/
│   ├── api/
│   │   └── CLAUDE.md            ← 模块级：API 设计背景（懒加载）
│   └── frontend/
│       └── CLAUDE.md            ← 模块级：前端组件规范（懒加载）
└── .claude/
    ├── CLAUDE.md                ← 补充项目级说明
    ├── settings.json
    ├── settings.local.json      ← 个人配置（gitignore）
    ├── rules/
    │   ├── typescript.md        ← paths: **/*.ts
    │   ├── testing.md           ← paths: **/*.test.*
    │   ├── sql.md               ← paths: **/*.sql
    │   └── global-style.md      ← 无 paths：全局风格
    ├── commands/
    ├── agents/
    └── skills/
```

**调试方法**：

| 问题 | 解决方案 |
|------|----------|
| 不知道哪些文件已加载 | 运行 `/memory` 查看当前 session 已加载文件列表 |
| 子目录规则没生效 | 先让 Claude 读该目录的一个文件，再试 |
| /compact 后指令消失 | 检查是否在根 CLAUDE.md 中，子目录需重新触发 |
| 规则被其他团队的 CLAUDE.md 干扰 | 在 `.claude/settings.local.json` 配置 `claudeMdExcludes` |
| 想精确追踪加载时机 | 使用 `InstructionsLoaded` hook 记录日志 |

### 六、Auto Memory 与 CLAUDE.md 的互补关系

| | CLAUDE.md | Auto Memory |
|---|---|---|
| 谁写 | 开发者 | Claude 自己 |
| 内容 | 规则和指令 | 学习到的模式和偏好 |
| 存储位置 | 项目仓库 | `~/.claude/projects/<project>/memory/` |
| 每次 session 加载 | 全量 | MEMORY.md 前 200 行或 25KB |

通过 `/memory` 命令可查看和管理 Auto Memory 文件。

## 结论

CLAUDE.md 不是单一文件，而是一套包含**多层级拼接**、**子目录懒加载**、**rules 路径作用域**与**渐进式披露指令**的完整体系。

实践建议：
- **简单项目**：根目录单个 CLAUDE.md（< 200 行）+ 文档地图型自然语言指令
- **复杂项目**：分层结构——根 CLAUDE.md 做总览，子目录 CLAUDE.md 做模块背景，`.claude/rules/` 做技术约束，再以渐进式披露引导按需读取业务文档
- **企业团队**：托管策略层统一基线 + 项目层团队约定 + 本地层个人偏好

需要警惕的是 [[topics/claude-md-guide]] 中提到的研究警示——**盲目添加 CLAUDE.md 反而可能降低任务成功率并增加成本 20%+**。结构清晰、内容精炼、按需加载，比"塞满全局规则"更有效。

## 相关概念

- [[concepts/claude-md]]
- [[concepts/agents]]
- [[concepts/context-files]]
- [[concepts/coding-agent]]

## 相关主题

- [[topics/claude-md-guide]]

## 来源

- `raw/ai-chat/claude_md_complete_guide.md`
- 官方文档：https://code.claude.com/docs/en/memory
