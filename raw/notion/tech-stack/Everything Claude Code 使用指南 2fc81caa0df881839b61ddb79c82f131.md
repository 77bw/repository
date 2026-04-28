# Everything Claude Code 使用指南

> 来自 Anthropic 黑客马拉松获胜者的完整 Claude Code 配置集合
> 

---

## 安装

### 插件安装（推荐）

```bash
# 1. 添加市场
/plugin marketplace add affaan-m/everything-claude-code

# 2. 安装插件
/plugin install everything-claude-code@everything-claude-code

# 3. 手动复制规则（必需，插件无法自动分发）
git clone https://github.com/affaan-m/everything-claude-code.git
cp -r everything-claude-code/rules/* ~/.claude/rules/
```

---

## 插件安装后获得的组件

| 组件 | 自动安装 | 数量 | 说明 |
| --- | --- | --- | --- |
| Agents | ✅ 是 | 13 个 | 专业子代理，处理特定任务 |
| Skills | ✅ 是 | 28 个 | 工作流定义和领域知识 |
| Commands | ✅ 是 | 24 个 | 斜杠命令快速执行 |
| Hooks | ✅ 是 | 自动 | 基于事件的自动化触发器 |
| Rules | ❌ 否 | 8 个 | 需手动复制到 ~/.claude/rules/ |
| MCP | ❌ 否 | 可选 | 需手动配置到 ~/.claude.json |

---

## 文章配置详解

```

everything-claude-code/
|-- .claude-plugin/         # 插件和市场清单
|   |-- plugin.json         # 插件元数据和组件路径
|   |-- marketplace.json    # /plugin marketplace add 的市场目录
|
|-- agents/           # 子智能体文件夹
|   |-- planner.md           		 # 功能实现规划
|   |-- architect.md         		 # 系统设计决策
|   |-- tdd-guide.md         		 # 测试驱动开发
|   |-- code-reviewer.md     		 # 质量和安全审查
|   |-- security-reviewer.md     # 漏洞分析
|   |-- build-error-resolver.md  # 构建错误解决
|   |-- e2e-runner.md        		 # Playwright E2E 测试
|   |-- refactor-cleaner.md  		 # 死代码清理
|   |-- doc-updater.md       	 	 # 文档同步
|   |-- go-reviewer.md       		 # Go 代码审查（新增）
|   |-- go-build-resolver.md 		 # Go 构建错误解决（新增）
|
|-- skills/           # 技能
|   |-- coding-standards/           # 编程语言最佳实践
|   |-- backend-patterns/           # API、数据库、缓存模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从会话中自动提取模式（长篇指南）
|   |-- continuous-learning-v2/     # 基于直觉的学习，带置信度评分
|   |-- iterative-retrieval/        # 子智能体的渐进式上下文优化
|   |-- strategic-compact/          # 手动压缩建议（长篇指南）
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- eval-harness/               # 验证循环评估（长篇指南）
|   |-- verification-loop/          # 持续验证（长篇指南）
|   |-- golang-patterns/            # Go 语言习惯用法和最佳实践（新增）
|   |-- golang-testing/             # Go 测试模式、TDD、基准测试（新增）
|
|-- commands/         # 命令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现规划
|   |-- e2e.md              # /e2e - E2E 测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 修复构建错误
|   |-- refactor-clean.md   # /refactor-clean - 死代码移除
|   |-- learn.md            # /learn - 会话中提取模式（长篇指南）
|   |-- checkpoint.md       # /checkpoint - 保存验证状态（长篇指南）
|   |-- verify.md           # /verify - 运行验证循环（长篇指南）
|   |-- setup-pm.md         # /setup-pm - 配置包管理器
|   |-- go-review.md        # /go-review - Go 代码审查（新增）
|   |-- go-test.md          # /go-test - Go TDD 工作流（新增）
|   |-- go-build.md         # /go-build - 修复 Go 构建错误（新增）
|   |-- skill-create.md     # /skill-create - 从 git 历史生成技能（新增）
|   |-- instinct-status.md  # /instinct-status - 查看学习到的直觉（新增）
|   |-- instinct-import.md  # /instinct-import - 导入直觉（新增）
|   |-- instinct-export.md  # /instinct-export - 导出直觉（新增）
|   |-- evolve.md           # /evolve - 将直觉聚类为技能（新增）
|
|-- rules/            # 始终遵循的指南（复制到 ~/.claude/rules/）
|   |-- security.md         # 强制性安全检查
|   |-- coding-style.md     # 不可变性、文件组织
|   |-- testing.md          # TDD、80% 覆盖率要求
|   |-- git-workflow.md     # 提交格式、PR 流程
|   |-- agents.md           # 何时委托给子智能体
|   |-- performance.md      # 模型选择、上下文管理
|
|-- hooks/            # 基于触发器的自动化
|   |-- hooks.json                # 所有钩子配置（PreToolUse、PostToolUse、Stop 等）
|   |-- memory-persistence/       # 会话生命周期钩子（长篇指南）
|   |-- strategic-compact/        # 压缩建议（长篇指南）
|
|-- scripts/          # 跨平台 Node.js 脚本（新增）
|   |-- lib/                     # 共享工具
|   |   |-- utils.js             # 跨平台文件/路径/系统工具
|   |   |-- package-manager.js   # 包管理器检测和选择
|   |-- hooks/                   # 钩子实现
|   |   |-- session-start.js     # 会话开始时加载上下文
|   |   |-- session-end.js       # 会话结束时保存状态
|   |   |-- pre-compact.js       # 压缩前状态保存
|   |   |-- suggest-compact.js   # 战略性压缩建议
|   |   |-- evaluate-session.js  # 从会话中提取模式
|   |-- setup-package-manager.js # 交互式包管理器设置
|
|-- tests/            # 测试套件（新增）
|   |-- lib/                     # 库测试
|   |-- hooks/                   # 钩子测试
|   |-- run-all.js               # 运行所有测试
|
|-- contexts/         # 动态系统提示注入上下文（长篇指南）
|   |-- dev.md              # 开发模式上下文
|   |-- review.md           # 代码审查模式上下文
|   |-- research.md         # 研究/探索模式上下文
|
|-- examples/         # 示例配置和会话
|   |-- CLAUDE.md           # 项目级配置示例
|   |-- user-CLAUDE.md      # 用户级配置示例
|
|-- mcp-configs/      # MCP 服务器配置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- marketplace.json  # 自托管市场配置（用于 /plugin marketplace add）

```

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

按照核心功能拆分使用的话：

1. 跨会话共享内存
    - scripts/hooks/session-start.js：会话开始钩子
    - scripts/hooks/pre-compact.js：会话压缩钩子
    - scripts/hooks/session-end.js：会话完成钩子
2. 持续学习并更新记忆：
    - commands/learn.md
    - skills/continuous-learning
3. 提高项目的可维护性：
    - commands/checkpoint.md
    - commands/verify.md
    - skills/verification-loop
    - commands/refactor-clean.md
    - agents/refactor-cleaner.md
    - agents/doc-updater.md
    - commands/update-codemaps.md
    - commands/update-docs.md
4. 子智能体使用方式：
    - skills/iterative-retrieval
    - commands/orchestrate.md
    - agents/architect.md
    - agents/code-reviewer.md
    - agents/planner.md
    - agents/security-reviewer.md
    - agents/tdd-guide.md

---

## Agents（专业代理）

代理是专门处理委托任务的子代理，具有有限的工具范围。

| 代理 | 用途 |
| --- | --- |
| planner | 功能实现规划 |
| architect | 系统设计决策 |
| tdd-guide | 测试驱动开发指导 |
| code-reviewer | 代码质量和安全审查 |
| security-reviewer | 安全漏洞分析 |
| python-reviewer | Python 代码专项审查 |
| go-reviewer | Go 代码专项审查 |
| database-reviewer | 数据库设计审查 |
| build-error-resolver | 构建错误解决 |
| go-build-resolver | Go 构建错误解决 |
| e2e-runner | Playwright E2E 测试 |
| refactor-cleaner | 死代码清理 |
| doc-updater | 文档同步更新 |

---

## Commands（斜杠命令）

通过 /命令名 快速执行特定工作流。

### 核心命令

| 命令 | 用途 | 示例 |
| --- | --- | --- |
| /plan | 实现规划 | /plan "添加用户认证" |
| /tdd | 测试驱动开发 | /tdd |
| /code-review | 代码质量审查 | /code-review |
| /build-fix | 修复构建错误 | /build-fix |
| /e2e | E2E 测试生成 | /e2e |

### Python 专用

- `/python-review` - Python 代码审查（PEP 8、类型提示、安全性）

### Go 专用

- `/go-review` - Go 代码审查
- `/go-test` - Go TDD 工作流
- `/go-build` - 修复 Go 构建错误

### 学习与进化

- `/learn` - 从会话中提取模式
- `/skill-create` - 从 git 历史生成技能
- `/instinct-status` - 查看学习的直觉
- `/evolve` - 将直觉聚类到技能

---

## Skills（技能知识库）

技能是工作流定义和领域知识的集合。

### 通用技能

- `coding-standards` - 通用编码规范
- `backend-patterns` - 后端架构模式
- `frontend-patterns` - 前端开发模式
- `security-review` - 安全检查清单
- `tdd-workflow` - TDD 方法论

### Python 技能

- `python-patterns` - Python 惯用语、PEP 8、类型提示
- `python-testing` - pytest、TDD、fixtures、mocking

### Go 技能

- `golang-patterns` - Go 惯用语和最佳实践
- `golang-testing` - Go 测试模式、TDD、基准测试

### 数据库技能

- `postgres-patterns` - PostgreSQL 查询优化、索引设计
- `clickhouse-io` - ClickHouse 分析数据库模式

---

## Hooks（自动化钩子）

钩子在特定事件时自动触发。

| 钩子类型 | 触发时机 | 作用 |
| --- | --- | --- |
| SessionStart | 会话开始 | 加载上下文、检测包管理器 |
| SessionEnd | 会话结束 | 保存状态、提取模式 |
| PreToolUse | 工具使用前 | 阻止危险操作、提醒 |
| PostToolUse | 工具使用后 | 格式化、类型检查、警告 |
| Stop | 响应结束 | 检查 console.log |

### 内置钩子功能

- 阻止 dev server 在非 tmux 环境运行
- 提醒使用 tmux 运行长时间命令
- git push 前提醒审查
- 阻止创建不必要的 .md 文件
- JS/TS 文件自动 Prettier 格式化
- TypeScript 类型检查
- console.log 警告

---

## Rules（规则）

规则是始终遵循的强制性指南，需手动复制到 ~/.claude/rules/。

| 规则文件 | 内容 |
| --- | --- |
| security.md | 无硬编码秘密、输入验证 |
| coding-style.md | 不可变性、文件组织 |
| testing.md | TDD、80% 覆盖率要求 |
| git-workflow.md | 提交格式、PR 流程 |
| agents.md | 何时委托给子代理 |
| performance.md | 模型选择、上下文管理 |
| hooks.md | 钩子使用指南 |
| patterns.md | 设计模式 |

---

## Python 后端/爬虫工程师推荐配置

### 必用技能

```
python-patterns      # Python 最佳实践
python-testing       # pytest + TDD
security-review      # 安全检查（爬虫涉及外部请求）
backend-patterns     # API 设计
postgres-patterns    # 数据库优化
```

### 必用命令

```
/plan               # 规划功能实现
/tdd                # 测试驱动开发
/python-review      # Python 代码审查
/code-review        # 通用代码审查
```

### 推荐代理

```
planner             # 功能规划
python-reviewer     # Python 专项审查
security-reviewer   # 安全审查
```

---

## 使用示例

### 规划新功能

```
/plan "设计一个异步爬虫任务队列，支持：
- 任务优先级
- 失败重试
- 并发控制
- 结果持久化"
```

### TDD 开发流程

```
/tdd

# Claude 会引导你：
# 1. 定义接口
# 2. 写失败测试 (RED)
# 3. 实现最小代码 (GREEN)
# 4. 重构 (IMPROVE)
# 5. 验证 80%+ 覆盖率
```

### 代码审查

```
/python-review

# 审查内容：
# - PEP 8 合规
# - 类型提示
# - 安全性
# - Pythonic 风格
```

### 从会话学习

```
/learn

# 从当前会话提取可复用模式
# 保存为技能供后续使用
```

---

## 注意事项

### 上下文窗口管理

- 不要一次启用太多 MCP（200k 可能缩小到 70k）
- 每个项目保持少于 10 个 MCP
- 活动工具少于 80 个

### 定制化建议

1. 从适合你的开始
2. 为你的技术栈修改
3. 删除不用的组件
4. 添加自己的模式

---

## 相关链接

- GitHub 仓库: [https://github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)
- 作者 Twitter: [@affaanmustafa](https://x.com/affaanmustafa)

[使用指南](Everything%20Claude%20Code%20%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97%202fd81caa0df8804ba98bc5292a535179.md)

[everything-claude-code-guide.md](Everything%20Claude%20Code%20%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/everything-claude-code-guide%20md%202ff81caa0df8813f998ffaafec20aeb2.md)