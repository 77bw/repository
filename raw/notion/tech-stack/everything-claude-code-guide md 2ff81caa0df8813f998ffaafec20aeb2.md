# everything-claude-code-guide.md

# Everything Claude Code 使用指南

> 基于 Anthropic 黑客马拉松冠军配置的 Claude Code 开发工作流
> 

---

## 一、导师提示词

将以下提示词发送给 AI，让它扮演 everything-claude-code 导师角色：

```markdown
# Role
你现在是 **Anthropic Hackathon 获胜者**，也是 GitHub 项目 [everything-claude-code](https://github.com/affaan-m/everything-claude-code) 的作者。你不仅精通代码，更深刻理解如何通过 Claude Code 结合 MCP 工具来通过 Memory Bank 模式构建复杂软件。

# Context & Knowledge Base
请基于以下信息作为你的知识库：
1. **GitHub 项目**: https://github.com/affaan-m/everything-claude-code
2. **参考文档**: .claude/reference/Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充.md

# User Status
我已经完成了以下配置：
1. 已通过插件方式安装 `everything-claude-code`
2. 已执行 `cp -r everything-claude-code/rules/* ~/.claude/rules`
3. 项目根目录下已初始化 `memory-bank` 文件夹

# Task
请以"导师"身份，教我如何使用 everything-claude-code 工作流进行开发。回答我的问题时：
1. 给出具体可执行的 Prompt 范例
2. 解释每个步骤的目的
3. 使用规范的命令格式（带 `/everything-claude-code:` 前缀）
```

---

## 二、沟通规划方法

### 2.1 启动项目开发

**第一步：让 AI 了解项目上下文**

```
请阅读 memory-bank/todo.md 和 memory-bank/architecture.md，
然后帮我规划第一个任务。
```

**第二步：使用 /plan 进行规划**

```
/everything-claude-code:plan

请规划 [Phase 名称] 的实现方案。

参考文档：
- memory-bank/PRD.md 第 X.X 节
- memory-bank/architecture.md

任务列表：
1. [具体任务1]
2. [具体任务2]
...

请输出详细规划，包括：
- 每个模块的核心类/函数签名
- 模块间依赖关系
- 风险点和应对策略

规划确认后，使用 TDD 模式实现。
```

### 2.2 规划确认后的沟通

**小步快跑模式**（适合学习/调试）：

```
是的，请执行这个计划。完成后：
1. /everything-claude-code:checkpoint verify 验证
2. 更新 memory-bank/todo.md 标记完成
3. 更新 memory-bank/progress.md 记录里程碑
4. 创建 git commit
```

**一步到位模式**（适合批量开发）：

```
是的，请继续完成整个 Phase X 的所有任务。

工作流程：
- 每完成一个模块立即 commit（不用等我确认）
- 目标测试覆盖率 60%+
- 全部完成后：
  1. /everything-claude-code:checkpoint verify
  2. /everything-claude-code:python-review
  3. 更新 memory-bank/progress.md 和 todo.md
  4. 给我一个完成报告

直接开始，无需再次确认。
```

---

## 三、开发流程

### 3.1 标准开发循环

```
┌─────────────────────────────────────────────────────┐
│  1. PLAN (规划)                                     │
│     /everything-claude-code:plan                    │
│     输出实现方案，等待用户确认                       │
└─────────────────────────────────────────────────────┘
                        ↓ 用户确认
┌─────────────────────────────────────────────────────┐
│  2. CHECKPOINT (检查点)                             │
│     /everything-claude-code:checkpoint create       │
│     保存初始状态                                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  3. TDD (测试驱动开发)                              │
│     /everything-claude-code:tdd                     │
│     测试先行，逐模块实现                             │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  4. VERIFY (验证)                                   │
│     /everything-claude-code:checkpoint verify       │
│     验证所有变更                                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  5. REVIEW (审查)                                   │
│     /everything-claude-code:python-review           │
│     代码审查（或 go-review）                         │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  6. LEARN (学习)                                    │
│     /everything-claude-code:learn                   │
│     提取可复用模式                                   │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  7. RECORD (记录)                                   │
│     更新 memory-bank + git commit                   │
│     记录进度，提交代码                               │
└─────────────────────────────────────────────────────┘
```

### 3.2 Memory Bank 更新规则

| 文件 | 更新时机 | 内容 |
| --- | --- | --- |
| `todo.md` | 任务完成后 | 标记 ✅，添加新任务 |
| `progress.md` | Phase 完成后 | 记录里程碑、commit SHA、关键特性 |
| `architecture.md` | 新增/删除文件后 | 更新项目结构说明 |
| `PRD.md` | 需求变更时 | 更新需求定义 |

---

## 四、常用命令速查表

### 4.1 插件命令（需要前缀）

| 命令 | 用途 | 使用时机 |
| --- | --- | --- |
| `/everything-claude-code:plan` | 规划实现方案 | 开始新功能前 |
| `/everything-claude-code:checkpoint create` | 保存当前状态 | 写代码前 |
| `/everything-claude-code:checkpoint verify` | 验证变更 | 写代码后 |
| `/everything-claude-code:tdd` | 测试驱动开发 | 实现功能时 |
| `/everything-claude-code:python-review` | Python 代码审查 | 代码完成后 |
| `/everything-claude-code:go-review` | Go 代码审查 | Go 代码完成后 |
| `/everything-claude-code:learn` | 保存学习记录 | 发现好模式时 |
| `/everything-claude-code:build-fix` | 修复构建错误 | 构建失败时 |
| `/everything-claude-code:refactor-clean` | 清理冗余代码 | 代码维护时 |
| `/everything-claude-code:verify quick` | 快速验证 | 快速检查时 |
| `/everything-claude-code:e2e` | E2E 测试 | 端到端测试时 |
| `/everything-claude-code:orchestrate` | 编排子智能体 | 复杂任务时 |

### 4.2 原生命令（无需前缀）

| 命令 | 用途 |
| --- | --- |
| `/clear` | 清除上下文 |
| `/compact` | 压缩上下文 |
| `/help` | 帮助信息 |
| `/cost` | 查看成本 |
| `/resume` | 恢复会话 |

---

## 五、常用技巧

### 5.1 高效批量开发

当你需要让 AI 一次性完成多个任务时：

```
是的，请继续完成 Phase X 的所有剩余任务：

1. [任务1]
2. [任务2]
3. [任务3]

工作流程：
- 每完成一个模块立即 commit（不用等我确认）
- 所有任务完成后统一更新 memory-bank
- 最后给我完成报告，包含：
  - 创建的文件列表
  - 测试覆盖率
  - 所有 commit SHA

直接开始，无需再次确认。
```

### 5.2 规划模板

```
/everything-claude-code:plan

请规划 [功能名称] 的实现方案。

参考文档：
- memory-bank/PRD.md 第 X.X 节（[具体内容]）
- memory-bank/architecture.md（[模块设计]）
- [已有相关代码文件]

任务列表：
1. [文件路径] - [功能描述]（[具体要求]）
2. [文件路径] - [功能描述]（[具体要求]）
...

关键约束：
- [约束1]
- [约束2]
...

请输出详细规划，包括：
- 每个模块的核心类/函数签名
- 模块间依赖关系
- 风险点和应对策略

规划确认后，使用 TDD 模式实现。
```

### 5.3 提取学习成果

当你完成一个 Phase 后，使用 `/learn` 保存可复用的模式：

```
/everything-claude-code:learn

请从本次 Phase X 开发中提取以下学习成果：
1. [模式1]（如：Pydantic Settings 配置模式）
2. [模式2]（如：asyncio.Semaphore 并发限制）
3. [模式3]（如：FFmpeg subprocess 心跳检测）
```

### 5.4 子智能体编排

当任务复杂时，使用编排命令：

```
/everything-claude-code:orchestrate custom "planner,tdd-guide,code-reviewer" "[任务描述]"
```

### 5.5 构建失败快速修复

```
/everything-claude-code:build-fix

构建失败，请分析错误并修复。
```

---

## 六、知识沉淀体系

### 6.1 两种知识类型

| 类型 | 级别 | 位置 | 目的 |
| --- | --- | --- | --- |
| 学习记录 (Learn Skills) | 用户级 | `~/.claude/skills/learned/` | 跨项目复用的通用模式 |
| 项目状态 (Memory Bank) | 项目级 | `./memory-bank/` | 项目特定的工作状态 |

### 6.2 学习记录 vs 项目约束

| 存放位置 | 内容类型 | 例子 |
| --- | --- | --- |
| `~/.claude/skills/learned/` | 通用编程模式 | Pydantic 单例模式 |
| `./CLAUDE.md` | 项目约束规则 | OSS 路径格式 |
| `./memory-bank/` | 项目状态和进度 | Phase 完成情况 |

---

## 七、Hooks 自动化

以下 Hooks 会自动执行，无需手动调用：

| Hook | 触发时机 | 功能 |
| --- | --- | --- |
| `session-start.js` | 会话开始 | 自动加载上次会话状态 |
| `session-end.js` | 会话结束 | 保存会话状态 |
| `pre-compact.js` | 上下文压缩前 | 保存重要信息 |
| `PostToolUse` | 工具使用后 | 自动格式化、类型检查 |

---

## 八、最佳实践总结

1. **写代码前必读 memory-bank** - 确保了解项目上下文
2. **每次执行前 checkpoint create** - 保存初始状态
3. **每次完成后 checkpoint verify** - 验证变更
4. **Phase 完成后更新 progress.md** - 记录里程碑
5. **发现好模式就 /learn** - 沉淀可复用知识
6. **使用 TDD 模式** - 测试先行，保证质量
7. **代码审查不能少** - `/python-review` 或 `/go-review`

---

## 九、故障排查

### 9.1 命令报错 "Skill is not a prompt-based skill"

使用 `Task` 工具或 `EnterPlanMode` 替代直接调用。

### 9.2 忘记命令前缀

所有 everything-claude-code 命令必须加 `/everything-claude-code:` 前缀。

### 9.3 上下文过长

使用 `/compact` 压缩上下文，或 `/clear` 清除后重新开始。

---

*文档版本: v1.0 | 更新日期: 2026-02-06*