# Claude Code CLAUDE.md 完整知识体系

> 整理自完整对话内容，涵盖文件层级、加载机制、rules 目录、渐进式披露指令、最佳实践五大模块。

---

## 一、文件层级与作用域

CLAUDE.md 文件可以存放在多个位置，每个层级有不同的作用范围。加载顺序从宽到窄，越靠近启动目录权重越高。

| 层级 | 路径 | 作用范围 | 共享方式 |
|------|------|----------|----------|
| **托管策略层** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 企业全员，不可被任何设置排除 | IT/DevOps 统一部署 |
| **用户层** | `~/.claude/CLAUDE.md` | 当前用户所有项目 | 仅自己 |
| **项目层** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享 | 提交 git，团队成员共享 |
| **本地层** | `./CLAUDE.local.md` | 仅自己当前项目 | 加入 `.gitignore`，不提交 |

**优先级规则**：
- 所有文件内容**拼接**（concat），不是覆盖
- 越靠近启动目录的文件越晚读取，权重越高（后读的指令更受重视）
- 同级目录中，`CLAUDE.local.md` 追加在 `CLAUDE.md` 之后

---

## 二、加载机制

### 2.1 启动时加载逻辑

Claude Code 启动时，从当前工作目录**向上遍历目录树**，收集每个目录中的 `CLAUDE.md` 和 `CLAUDE.local.md`：

```
启动目录: /project/src/api/
加载顺序（从上往下，越下越晚读权重越高）:
  /CLAUDE.md
  /project/CLAUDE.md
  /project/src/CLAUDE.md
  /project/src/api/CLAUDE.md        ← 最晚读，权重最高
  /project/src/api/CLAUDE.local.md  ← 同级 local 最后追加
```

### 2.2 子目录懒加载（渐进式文件系统触发）

工作目录**以下**的子目录 CLAUDE.md **不在启动时加载**，而是当 Claude 读取该子目录中任意文件时才触发：

```
在 /project 启动 → 不会立即加载 /project/src/api/CLAUDE.md
Claude 读取 /project/src/api/handler.ts → 自动加载 /project/src/api/CLAUDE.md
```

**这是设计意图**，目的是减少不必要的上下文消耗。

### 2.3 强制加载子目录的三种方法

**方法一：@import 语法（推荐，简单直接）**

在根目录 `CLAUDE.md` 中直接引用：

```markdown
## 项目文档
@src/api/CLAUDE.md
@src/frontend/CLAUDE.md
@packages/shared/CLAUDE.md
```

特性：
- 启动时立即展开加载进上下文
- 支持相对路径和绝对路径
- 递归 import 最多 5 层
- **代价**：所有内容都进入上下文，消耗 token

**方法二：`--add-dir` + 环境变量**

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ./src/api
```

会加载额外目录中的 `CLAUDE.md`、`.claude/CLAUDE.md`、`.claude/rules/*.md`、`CLAUDE.local.md`。

**方法三：接受懒加载，主动触发**

在 session 开始时指示 Claude 读取目标子目录中的文件：

```
"先帮我看一下 src/api/index.ts 的结构"
```

这会自动触发 `src/api/CLAUDE.md` 的加载。

### 2.4 /compact 后的行为

| 文件类型 | /compact 后行为 |
|----------|-----------------|
| 根目录 CLAUDE.md | **自动存活**，重新从磁盘读取注入 |
| 子目录嵌套 CLAUDE.md | **不自动重注入**，需再次触发（Claude 读该目录文件） |
| 对话中临时指令 | 丢失，需重新告知 |

---

## 三、.claude/rules/ 路径作用域机制

### 3.1 目录结构

```
your-project/
├── CLAUDE.md
└── .claude/
    └── rules/
        ├── code-style.md     # 代码风格（无 paths → 启动无条件加载）
        ├── testing.md        # 测试规范（有 paths → 匹配文件时加载）
        ├── security.md       # 安全要求
        └── frontend/
            └── react.md     # 支持子目录递归发现
```

### 3.2 两类规则的行为差异

**有 `paths` frontmatter（路径作用域规则）**

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "tests/**/*.test.ts"
---

# API 开发规范
- 所有接口必须包含入参校验
- 返回统一错误格式 { code, message, data }
- 包含 OpenAPI 注释
```

触发条件：Claude 读取匹配路径的文件时才加载（不是每次工具调用都触发）

**无 `paths` frontmatter**

```markdown
# 通用规范（无 frontmatter）
- 所有 PR 必须通过 CI 才能合并
- 变量命名使用驼峰命名法
```

行为：启动时无条件加载，等价于写在 `.claude/CLAUDE.md` 里

### 3.3 常用 glob 模式

| 模式 | 匹配范围 |
|------|----------|
| `**/*.ts` | 所有 TypeScript 文件 |
| `src/**/*` | src 目录下所有文件 |
| `*.md` | 项目根目录的 Markdown 文件 |
| `src/components/*.tsx` | 特定目录的 React 组件 |
| `**/*.{ts,tsx}` | 多扩展名（花括号展开） |

### 3.4 用户级 rules

```
~/.claude/rules/
├── preferences.md    # 个人编码偏好，所有项目生效
└── workflows.md      # 个人工作流
```

用户级 rules 先于项目 rules 加载，项目 rules 优先级更高。

### 3.5 symlink 跨项目共享

```bash
# 将共享规则链接到多个项目
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

---

## 四、渐进式披露指令（CLAUDE.md 自然语言控制）

### 4.1 与文件系统机制的根本区别

| 维度 | 文件系统机制（rules/paths） | CLAUDE.md 自然语言指令 |
|------|----|----|
| **控制者** | Claude Code 引擎 | Claude 的理解与判断 |
| **触发方式** | 读取文件路径自动匹配 | 读自然语言规则后自行决定 |
| **适用内容** | `.md` 规则文件 | 任意文件（文档、代码、JSON…） |
| **粒度** | glob 路径匹配 | 语义匹配（"当你需要了解 API 时"） |
| **可靠性** | 确定性触发 | 依赖 Claude 理解准确度 |
| **适合场景** | 技术约束、编码规范 | 业务文档、架构说明、项目状态 |

### 4.2 四种写法模式

**模式一：文档地图型**（对应截图一的写法）

```markdown
## 参考文档

按需读取，不要在每次对话开始时全部加载：

- [project_spec.md](project_spec.md) — 需要了解完整需求/API 规格时读
- [docs/architecture.md](docs/architecture.md) — 需要理解系统设计和数据流时读
- [docs/changelog.md](docs/changelog.md) — 需要查版本历史时读
- [docs/project_status.md](docs/project_status.md) — 需要了解当前进度时读

重大里程碑后更新 docs 文件夹中的文档。
提交代码时使用 /update-docs-and-commit 命令。
```

**模式二：任务触发型**

```markdown
## 工作流程规则

- 修改任何 API 接口前，先读 docs/api-contracts.md
- 遇到数据库相关任务时，先读 docs/schema.md
- 创建新组件前，先读 src/components/README.md 了解现有组件
- 涉及权限相关逻辑时，先读 docs/auth-design.md
```

**模式三：目录规范声明型**（对应截图二的写法）

```markdown
## 项目结构规范

- Slash Command 文件使用 Markdown 格式，放在 .claude/commands/ 下
- Skill 文件命名为 SKILL.md，放在 .claude/skills/<skill-name>/ 下
- Agent 文件放在 .claude/agents/ 下
- 文档和参考资料放在 .claude/docs/ 下

创建新文件时必须遵循以上规范，不得自行决定路径。
```

**模式四：条件读取型（最精确的渐进式披露）**

```markdown
## 上下文加载规则

### 始终加载（已在本文件中）
- 项目名称、技术栈、核心编码约定

### 按场景主动读取对应文件

| 场景 | 读取文件 |
|------|---------|
| 新功能开发 | project_spec.md + docs/architecture.md |
| Bug 修复 | docs/changelog.md + 相关模块 CLAUDE.md |
| 数据库操作 | docs/schema.md |
| 部署相关 | docs/deployment.md |
| 权限/认证相关 | docs/auth-design.md |

### 永远不需要主动读取
- node_modules/** 下的文件
- .env 文件（敏感信息）
- dist/ 或 build/ 目录下的产物
```

---

## 五、最佳实践

### 5.1 写作规范

- **大小**：每个 CLAUDE.md 文件目标控制在 200 行以内；超出后 Claude 遵从度下降
- **结构**：用 Markdown headers 和 bullet 分组相关指令，结构越清晰遵从越好
- **具体性**：写可验证的指令
  - ✅ "使用 2 空格缩进"
  - ❌ "格式化好代码"
  - ✅ "提交前运行 `npm test`"
  - ❌ "测试你的改动"
- **一致性**：定期检查多个 CLAUDE.md 文件间是否有矛盾指令
- **HTML 注释**：`<!-- 维护者说明 -->` 会在注入前被去除，可用于写给人看的说明，不消耗 token

### 5.2 两种机制的选择原则

```
规则是"针对某种文件类型的编写规范"
  → .claude/rules/ + paths（文件系统触发）
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

### 5.3 推荐完整目录结构

```
project/
├── CLAUDE.md                    ← 全局：架构总览、build 命令、团队约定、文档地图
├── CLAUDE.local.md              ← 个人：沙箱 URL、测试数据（gitignore）
├── src/
│   ├── api/
│   │   └── CLAUDE.md            ← 模块级：API 设计背景（懒加载）
│   └── frontend/
│       └── CLAUDE.md            ← 模块级：前端组件规范（懒加载）
└── .claude/
    ├── CLAUDE.md                ← 补充项目级说明
    ├── settings.json            ← 权限、模型配置
    ├── settings.local.json      ← 个人配置（gitignore）
    ├── rules/
    │   ├── typescript.md        ← paths: **/*.ts
    │   ├── testing.md           ← paths: **/*.test.*
    │   ├── sql.md               ← paths: **/*.sql
    │   └── global-style.md      ← 无 paths：全局风格
    ├── commands/
    │   └── update-docs-and-commit.md
    ├── agents/
    │   └── code-reviewer.md
    └── skills/
        └── deploy/
            └── SKILL.md
```

### 5.4 调试方法

| 问题 | 解决方案 |
|------|----------|
| 不知道哪些文件已加载 | 运行 `/memory`，查看当前 session 已加载文件列表 |
| 子目录规则没生效 | 先让 Claude 读该目录的一个文件，再试 |
| /compact 后指令消失 | 检查是否在根 CLAUDE.md 中，子目录需重新触发 |
| 规则被其他团队的 CLAUDE.md 干扰 | 在 `.claude/settings.local.json` 中配置 `claudeMdExcludes` |
| 想精确追踪加载时机 | 使用 `InstructionsLoaded` hook 记录日志 |

### 5.5 Auto Memory 补充说明

Auto Memory 是另一套独立机制，与 CLAUDE.md 互补：

| | CLAUDE.md | Auto Memory |
|---|---|---|
| 谁写 | 开发者 | Claude 自己 |
| 内容 | 规则和指令 | 学习到的模式和偏好 |
| 存储位置 | 项目仓库 | `~/.claude/projects/<project>/memory/` |
| 每次 session 加载 | 全量 | MEMORY.md 前 200 行或 25KB |

查看和管理：运行 `/memory` → 选择 auto memory 文件夹即可浏览、编辑、删除。

---

## 六、核心机制总览

```
Claude Code 启动
      │
      ▼
加载 CLAUDE.md 文件（向上遍历 + 无 paths 的 rules）
      │
      ├─ 托管策略层 CLAUDE.md
      ├─ 用户层 ~/.claude/CLAUDE.md
      ├─ 工作目录上级各层 CLAUDE.md
      ├─ 工作目录 CLAUDE.md / .claude/CLAUDE.md
      ├─ 工作目录 CLAUDE.local.md
      └─ .claude/rules/ 无 paths 的规则文件
      │
      ▼
注入 Auto Memory（MEMORY.md 前 200 行）
      │
      ▼
Session 开始，等待用户输入
      │
      ├─ 用户任务涉及某子目录文件
      │        └─ 触发：子目录 CLAUDE.md 懒加载
      │
      └─ Claude 读取匹配 glob 的文件
               └─ 触发：.claude/rules/ 中对应 paths 规则加载
```

---

*文档生成时间：2026年5月*
*来源：Claude Code 官方文档 https://code.claude.com/docs/en/memory*
