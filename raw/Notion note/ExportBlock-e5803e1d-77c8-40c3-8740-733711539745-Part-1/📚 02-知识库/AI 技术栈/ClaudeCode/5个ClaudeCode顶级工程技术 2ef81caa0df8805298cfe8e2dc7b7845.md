# 5个ClaudeCode顶级工程技术

---

### 🎬 笔记来源

- 【5个ClaudeCode顶级工程技术-哔哩哔哩】[https://b23.tv/oSd3STT](https://b23.tv/oSd3STT)
- 备注：你想找“转发的原作者视频”，通常需要点进 B 站视频页查看简介/置顶评论/合集来源或转发链接（如果你把原视频页链接或截图发我，我可以帮你定位原作者信息）。

---

### 📝 学习笔记：顶尖 Agentic 工程师的 5 大核心技巧

**对应视频：**《The 5 Techniques Separating Top Agentic Engineers Right Now》  

**主题：**Agentic Engineering（代理工程）  

**目标：**通过 5 个核心技巧，把 AI 编程助手（Claude Code、Cursor 等）从“聊天工具”升级为“可重复的工程系统”。

---

### ⭐ 核心理念（先建立系统）

不要仅仅把 AI 当作聊天机器人，而是要建立一套**系统（System）**：  

从随意的 Prompting，转变为结构化的工作流与工程化的输入输出。

---

## 1) PRD First Development（产品需求文档优先开发）

### 目标

建立项目的“北极星”文档，让 AI 始终明确目标与范围。

### 什么是 PRD（此语境）

一个 Markdown 文档，用于定义项目的完整工作范围（Scope of Work）。

### 应用场景

- **Green Field（从零开发）**：包含完成 MVP 所需的全部功能定义
- **Brown Field（现有项目）**：记录现有功能与下一步要构建的内容

### 操作流（关键）

不要让 Agent 一次性做太多。  

使用 PRD 将大项目拆解为细颗粒度功能，例如：

- API 实现
- UI 构建
- 鉴权模块

### 作用

PRD 连接你与 Agent 的所有交互，并作为后续开发的基准参考。

---

## 2) Modular Rules Architecture（模块化规则架构）

### 目标

保护 LLM 的上下文窗口（Context Window），避免规则过载。

### 做法

- **轻量化全局规则**：全局规则文件（如 `.cursorrules` 或 [`agents.md`](http://agents.md)）保持极简（建议 < 200 行），只放通用规则，例如：
    - Tech Stack
    - Logging 策略
- **任务专用引用（Reference Context）**：将具体任务规则拆到独立 Markdown 文件，例如：
    - `rules/[api.md](http://api.md)`
    - `rules/[frontend.md](http://frontend.md)`
- **动态加载**：只有在做特定任务（如写 API）时，才让 Agent 读取对应规则文件

### 优势

防止 LLM 被无关规则淹没，提升注意力与准确性。

---

## 3) Commandify Everything（一切皆指令）

### 目标

把重复 Prompt 转换为可复用的工作流指令（Command）。

### 原则

如果同一个 Prompt 输入超过两次，就应该把它变成一个 Command。

### 实现方式

创建 Markdown 文档来定义特定工作流程。

### 核心指令

-（此处你的原笔记还没写完，可以在这里继续补充后续指令列表）

---