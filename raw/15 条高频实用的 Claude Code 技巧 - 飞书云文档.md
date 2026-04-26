---
title: "15 条高频实用的 Claude Code 技巧 - 飞书云文档"
source: "https://my.feishu.cn/wiki/EfRnwlHcSiLQAHkkDGlcxSgKnGb"
author:
published:
created: 2026-04-26
description:
tags:
  - "clippings"
---
## 飞书云文档

A

AI随风的AI编程快乐屋

互联网公开

问问知识库

目录

- [😘15 条高频实用的 Claude Code 技巧](#BHFede5Bno7oxMx2uN9cRN4ynie)
- [1、设置 cc 别名](#RSY4dtyyboiTdixpRNJcFJVgnsQ)
- [2、用 /init命令重构/初始化CLAUDE.md 文档](#ClpWd0TKRouTBFxfEGQc8TovnMb)
- [3、不喜欢终端但又想使用Claude Code,使用其他 UI](#S7H0dMKRPoB3RxxD5xbcY3GrnMw)
- [4、按 Esc 键是停止。按 Esc+Esc 可以直接进入/rewind 模式进行回滚。](#R36idd15eote9VxH2lRcbUA1ntg)
- [5、在跟 Claude 对话时，提供详细的反馈链路](#DmHDdajCJogIr7x2ctGchcrsnxe)
- [6、安装一个针对你语言的代码智能插件（LSP）](#M7hJdop0RonUKHxqbkmcBepknIh)
- [7、尽可能多的使用技能/创造技能](#TorNd75lFoLLn0xnwqpcBtkEnGg)
- [8、当你不确定如何处理某件事时，使用计划模式](#AJWfdKWVjowGbJx5kc0ckjYMnCf)
- [9、别用自己的话转述 bug，直接把报错贴给 Claude](#RfROd0ZizozQU8xXjiecyKbQnMc)
- [10、开启无关的任务时，进行/clear 操作，清空上下文](#TKl9d8x83okdI3x7cV5ctkBense)
- [11、使用子代理保持主上下文的清晰](#K6godqF7Lo5SB4xxg1sctT3rnHb)
- [12、使用/btw 来进行快速提问](#Psr3dZm1YoWIlJx571Dc8As0nMh)
- [13、使用--worktree 并行开发](#OGu0dhSDrorhL5xdZGZcXnufnZf)
- [14、选择合适 的 MCP 来提升效率](#E1lEd3NEDoJAWjxNC5ccOY3NnFb)
- [15、告诉Claude具体要看哪些文件](#M30tdxHZfonnQRx2DpKcNZ03n7g)

1、设置 cc 别名

我每次都是用 c-d 命令来启动 claude 绕过权限

把这个加到你的 ~/.zshrc（或 ~/.bashrc）里, windows 加到命令中

代码块

alias c-d='claude --dangerously-skip-permissions'

你也可以通过编辑 ～/.claude/setting.json 文件来达到绕过权限的效果，但是 c-d 更酷

代码块

{

"permissions": {

"allow": \[

"WebSearch", "WebFetch", "Bash", "Read", "Write",

"Edit", "Glob", "Grep", "Task", "TodoWrite"

\],

"deny": \[\],

"defaultMode": "bypassPermissions"

},

"skipDangerousModePermissionPrompt": true

}

2、用 /init命令重构/初始化CLAUDE.md 文档

CLAUDE.md 是 Claude Code 非常重要的记忆文档，这个文件是随着开发迭代逐步完善的，如果你不知道怎么写，你可以使用这个命令进行创建，但是建议初始化完成之后进行手动调整。

记住 CLAUDE.md 文档中内容的一个大原则。

可以参考这个结构

代码块

\# Project: ShopFront

Next.js 14 e-commerce application with App Router, Stripe payments, and Prisma ORM.

评论（0）

跳转至首条评论

0 字

- 帮助中心

- 效率指南