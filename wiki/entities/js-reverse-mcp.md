---
title: js-reverse-mcp
type: entity
created: 2026-06-02
updated: 2026-06-02
tags: [spider, mcp, tooling, github, ai-coding]
sources:
  - https://github.com/zhizhuodemao/js-reverse-mcp
summary: 为 AI Agent 设计的真实 Chrome 插桩 MCP，用于 JS 逆向请求链定位
confidence: high
---

# js-reverse-mcp

## 简介

**js-reverse-mcp** 是为 AI Agent 设计的 JS 逆向 MCP Server,提供真实浏览器插桩能力。它驱动本机真实 Google Chrome(有头、可带登录态/扩展),用 Patchright 在 CDP 协议层规避 `Runtime.enable`/`Console.enable` 泄露,包装层零 JS 注入、不做 `Object.defineProperty` hack(hack 本身就是检测信号)。

仓库归属 `zhizhuodemao/js-reverse-mcp`（2026-06-02 GitHub 快照约 1.6k stars / 230 forks，Apache-2.0，TypeScript，created 2025-11-29）。自我定位"内置反检测、基于 chrome-devtools-mcp 重构"。

它是 [[entities/reverse-skill]] 和 [[entities/ai-reverse-toolkit]] 的运行时依赖——逆向 skill 把"目标 JS 在哪、怎么调、参数长啥样"交给 MCP 侦察,把"脱机跑通算法"交给补环境引擎([[entities/iv8]])。

## 核心功能

### 21 个工具(5 类)

据 `gh api repos/zhizhuodemao/js-reverse-mcp` + `docs/tool-reference.md` 实读:

| 类别 | 工具 | 用途 |
|------|------|------|
| 页面导航 | `navigate_page` / `new_page` / `select_page` / `select_frame` / `take_screenshot` | 驱动真实 Chrome 打开目标页 |
| 脚本分析 | `list_scripts` / `get_script_source`(小片段) / `save_script_source`(整文件,自动 prettier 美化) / `search_in_sources`(支持 regex,`excludeMinified=false`) | 拉取/全文搜索目标 JS |
| 断点执行 | `set_breakpoint_on_text`(对压缩代码按文本下断点) / `break_on_xhr` / `get_paused_info` / `pause_or_resume` / `step` / `list_breakpoints` / `remove_breakpoint` | 在加密函数处断下、单步 |
| 网络/WS | `list_network_requests` / `get_request_initiator`(取某请求的 JS 调用栈) / `get_websocket_messages` | 定位请求由哪段 JS 发起 |
| 检查 | `evaluate_script`(暂停时自动在 paused call frame 求值,可指定 `frameIndex`) / `list_console_messages` | 在页面上下文求值、读 console |

**杀手锏**: `get_request_initiator` 取调用栈 + `set_breakpoint_on_text` 对压缩代码按文本下断,无需美化就能定位加密入口。

### 反检测机制(两层)

- **CDP 协议层**: Patchright 规避 `Runtime.enable`/`Console.enable` 泄露点,包装层零 JS 注入、不做 `Object.defineProperty` hack
- **二进制层**: `--cloak` 是可选升级，换 CloakBrowser 二进制(需预 `npx cloakbrowser install` 下载约 200MB),做 C++ 层指纹 patch

与 [[entities/iv8]] 补环境层反检测(禁用 `debugger;`、`wrapNative` 伪装 `[native code]`)形成"侦察+运行时"双层防护。

## 关键事实与时间线

### npm 发布(据 registry.npmjs.org 实读)

- 包名: `js-reverse-mcp`
- 最新版本: **2.0.2**(据 2026-06 npm 实读)
- 系统要求: **Node.js v20.19+** 与 **Chrome stable**

### 接入命令(README 实读,逐字核实)

```bash
# Claude Code
claude mcp add js-reverse npx js-reverse-mcp

# Codex(注意 -- 分隔符)
codex mcp add js-reverse -- npx js-reverse-mcp
```

通用 JSON(Cursor / VS Code):
```json
{"command":"npx","args":["js-reverse-mcp"]}
```

**可选 flags**:
- `--cloak`: 换 CloakBrowser 二进制 + C++ 层指纹 patch(强反爬站点)
- `--isolated`: 临时 profile
- `-u http://127.0.0.1:9222`: 连已运行 Chrome 的 CDP 端点(支持 AdsPower/BitBrowser 等能响应 `/json/version` 的端点)

### 避坑

`ai-reverse-toolkit` README 的 Dependencies 段把 MCP 链接误写成 `anthropics/anthropic-cookbook`(失效占位)。真实仓库是 `zhizhuodemao/js-reverse-mcp`,以此为准。

## 相关实体与概念

- [[entities/reverse-skill]] — jsr-reverse skill,依赖本 MCP 做真实浏览器侦察
- [[entities/ai-reverse-toolkit]] — 4 个逆向 skill(find-crypto-entry/env-patch/ast-deobfuscate/skill-creator),依赖本 MCP
- [[entities/iv8]] — 补环境落地引擎,与本 MCP 形成"侦察+运行时"组合
- [[topics/ai-js-reverse-workflow]] — "MCP 侦察 → iv8 补环境出参 → curl_cffi 回灌"完整 SOP

## 来源

- GitHub 仓库(2026-06 实读): `gh api repos/zhizhuodemao/js-reverse-mcp` + README + docs/tool-reference.md
- npm registry(2026-06 实读): `https://registry.npmjs.org/js-reverse-mcp`
- 跨源综合: [[topics/ai-js-reverse-workflow]] 详细接入步骤段落
