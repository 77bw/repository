---
title: ai-reverse-toolkit
type: entity
created: 2026-06-02
updated: 2026-06-02
tags: [spider, ai-coding, tooling, methodology, github]
sources:
  - https://github.com/zhizhuodemao/ai-reverse-toolkit
summary: 将 JS 逆向流程拆成 find-crypto-entry、env-patch、AST 等专项 skill 的工具集
confidence: high
---

# ai-reverse-toolkit

## 简介

**ai-reverse-toolkit** 是一套把逆向经验编码为 AI 可执行指令的工具集,理念"工具解决怎么做,经验解决做什么"。它将"侦察 → 定位加密函数 → 补环境 → 跑出参数 → 回灌真实请求"这条 JS 逆向流水线封装成 4 个独立 Skill + 1 份 rules 手册。

仓库归属 `zhizhuodemao/ai-reverse-toolkit`（2026-06-02 GitHub 快照约 296 stars，MIT，created 2026-02-05）。运行时依赖 [[entities/js-reverse-mcp]] 提供真实浏览器插桩。

与 [[entities/reverse-skill]]（2026-06-02 快照约 242 stars，jsr-reverse 中文方法论 skill）定位不同：reverse-skill 是单一入口的统一逆向流程，ai-reverse-toolkit 是模块化的 4 个专项 skill，各自可独立调用。

## 核心功能

### 4 个 Skill + 1 份 rules

据 `gh api repos/zhizhuodemao/ai-reverse-toolkit` + README 实读:

| 组件 | 作用 | 输入 → 输出 |
|------|------|------------|
| `find-crypto-entry` | 定位加密参数生成入口 | 参数名(如 `x-sign`) → 脚本 URL+行列+函数+调用链+加密类型 |
| `env-patch` | 补环境:模块提取 + Proxy 监控 + 封装接口 | 已知加密入口 → 可独立运行的签名接口 |
| `ast-deobfuscate` | AST 解混淆(字符串还原/控制流还原) | 混淆 JS → 可读 JS,**20KB 单篇**(本仓最大 AST 方法论) |
| `skill-creator` | 创建/评测/优化 Skill | 需求描述 → 完整 Skill |
| `rules/js-reverse.md` | JS 逆向技术手册:6 阶段流程 + 混淆/算法识别表 + MCP 速查 + VM/WASM 高级场景 | 背景知识,非流程 |

### 唯一入口设计

每个 `SKILL.md` 的 frontmatter `description` 内置 `TRIGGER when` / `DO NOT TRIGGER when` 双向条件 + `argument-hint`。入口统一为自然语言或斜杠命令——用户说"补环境"自动路由到 `env-patch`,或显式 `/env-patch [项目名]`,由 description 匹配器决定调度,agent 无需记命令清单。

### 铁律五条(env-patch 精华)

据 `skills/env-patch/SKILL.md` 实读:

1. **VMP 边界决定补环境范围**: VMP 入口的依赖数组是补环境天花板,超出部分无法纯补环境
2. **加载顺序是致命的**: VMP 在 `require()`/脚本执行瞬间读环境,之后不可改,必须"先建环境 → 再加载目标 JS"
3. **格式验证优先于请求验证**: 签名长度/前缀与浏览器一致,即使 HTTP 200,签名降级也是假阳性
4. **至少 3 组输入对比**: 防止假阳性,对比 JS 与 Python 输出
5. **toString 一致性五条铁律**: Function.prototype.toString/typeof/constructor/Symbol.toStringTag/Object.prototype.toString 全对齐

### 默认 Node 路线 → iv8 折算

ai-reverse-toolkit 默认演示 **Node 补环境**心智,用户折算到 [[entities/iv8]] 落地:

| 引擎无关思维(直接复利) | Node 默认手段 | iv8 等价方案 |
|------|------|------|
| 缺啥补啥探针 | Proxy 包裹 window/navigator 打日志 | iv8 内 V8 原生 Proxy 同款 |
| 浏览器 API 声明式覆盖 | 手写 getter/jsdom polyfill | `environment` dict(C++ 层,不可被 JS 探测) |
| 函数 native 伪装 | `Function.prototype.toString` Proxy hook | `__iv8__.wrapNative` → `[native code]` |
| 时间控制对抗 POW | sinon fake timers | `eventLoop.advance` + `time_mode=logical` |
| 离线资源解耦网络 | mock fetch/XHR + axios | `add_resource` / `netLog` + curl_cffi 真发包 |

## 关键事实与时间线

### 安装方式(README 实读)

配置依赖 MCP(运行前置):

```bash
# Claude Code
claude mcp add js-reverse npx js-reverse-mcp
# Codex
codex mcp add js-reverse -- npx js-reverse-mcp
```

安装 skill(注意:本知识库用户选择只入库不安装,以下为参考步骤):

```bash
# Claude Code:复制到项目 .claude/
cp -r skills/* your-project/.claude/skills/
cp -r rules/* your-project/.claude/rules/
# 之后斜杠调用: /find-crypto-entry x-sign → /env-patch → /ast-deobfuscate

# Codex / 其他工具:直接把对应 .md 当 system prompt / context 注入
```

### 项目约定(CLAUDE.md 实读)

- Python 环境管理: `uv`
- 进度交接: 写 `PROGRESS.md`(跨会话续接靠它)
- 解混淆脚本: `scripts/deobfuscate_{target}.js`
- Python 实现: `output/sign.py`
- 内容语言: 简体中文

### 知乎 x-zse-96 demo(README 实证)

3 条递进指令 15 分钟跑通:
1. 找接口签名参数
2. `/find-crypto-entry x-zse-96`
3. `/env-patch`

这套"全自动"是用 skill 链式调用实现的,非单命令。

### 与 reverse-skill 的关系

二者协作但独立:
- **reverse-skill**(715494637/reverse-skill，2026-06-02 快照约 242 stars，无 license): jsr-reverse 单一入口，强调请求链证据化与运行时对齐
- **ai-reverse-toolkit**(本库，2026-06-02 快照约 296 stars，MIT): 4 个模块化 skill，强调补环境铁律与 AST 实操

用户可按需选用:全自动流程用 jsr-reverse,专项任务(如只需解混淆)用 ast-deobfuscate。二者共享 MCP 依赖([[entities/js-reverse-mcp]])。

## 相关实体与概念

- [[entities/js-reverse-mcp]] — 本 toolkit 的运行时依赖,真实 Chrome CDP 插桩
- [[entities/reverse-skill]] — 另一套逆向 skill(单一入口,中文方法论),与本库定位不同但可协作
- [[entities/iv8]] — 补环境落地引擎,Node 默认路线的折算目标
- [[topics/ai-js-reverse-workflow]] — "MCP 侦察 → iv8 补环境 → 回灌"完整 SOP,整合本 toolkit 用法
- [[topics/js-reverse-general-techniques]] — 通用逆向技巧深度手册(JSVMP/AST/webpack/WASM),本 toolkit 的方法论底座

## 来源

- GitHub 仓库(2026-06 实读): `gh api repos/zhizhuodemao/ai-reverse-toolkit` + README + skills/*/SKILL.md + rules/js-reverse.md
- 跨源综合: [[topics/ai-js-reverse-workflow]] 详细用法段落
