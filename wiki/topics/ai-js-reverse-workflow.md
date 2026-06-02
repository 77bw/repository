---
title: AI 驱动 JS 逆向 0→1 工作流
type: topic
created: 2026-06-02
updated: 2026-06-02
tags: [spider, ai-coding, mcp, methodology, sop]
sources:
  - https://github.com/715494637/reverse-skill
  - https://github.com/zhizhuodemao/js-reverse-mcp
  - https://github.com/zhizhuodemao/ai-reverse-toolkit
  - raw/github/HanZzzzz000iv8.md
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
summary: 真实 Chrome 侦察、iv8 离线出参、Python 回灌的 AI 逆向流水线
confidence: high
---

# AI 驱动 JS 逆向 0→1 工作流（iv8-first）

## 概述（背景与问题）

AI 爬虫逆向的目标不是“把网页 JS 看懂”，而是把目标站的动态参数、cookie、签名或 token 变成可验证、可复用、可批量执行的采集能力。推荐主线是：

```text
真实 Chrome 侦察 -> 阶段路由 -> 壳层压缩 -> iv8 离线出参 -> Python HTTP 回灌
```

分工必须清楚：

| 组件 | 角色 | 不负责 |
|------|------|--------|
| [[entities/js-reverse-mcp]] | 在线侦察：真实 Chrome 中证明请求链、脚本入口、调用栈、样本输入输出 | 不做补环境引擎 |
| [[entities/reverse-skill]] | 方法论路由：`intake -> evidence -> locate -> recover -> runtime -> validation -> handoff` | 不替代一手证据 |
| [[entities/ai-reverse-toolkit]] | 专项工具：入口定位、AST/webpack、Node 补环境经验库 | `env-patch` 默认不是 iv8 原生路线 |
| [[entities/iv8]] | 离线执行：`environment/config/page.load/eventLoop/netLog/wrapNative` 跑出参数 | 不负责真实 TLS/JA3 发包 |
| Python HTTP 层 | 回灌请求：`curl_cffi` / requests session / cookie 池 / 代理 / 重试 | 不负责浏览器 API 模拟 |

Context7 当前未收录 `HanZzzzz000/iv8`，因此 iv8 API 事实以 GitHub README、demos、examples、issues、PyPI 为准；Context7 只适合作 V8/Node/事件循环背景参考。

---

## 一、起手配置

### MCP：真实 Chrome 侦察层

`zhizhuodemao/js-reverse-mcp` 是真实 Chrome 调试 MCP。默认模式使用系统 Google Chrome + Patchright 协议层反检测；`--cloak` 是被强反爬拦截时的升级选项，不是默认必选。

```bash
# Claude Code
claude mcp add js-reverse npx js-reverse-mcp

# Codex
codex mcp add js-reverse -- npx js-reverse-mcp
```

关键工具：`list_network_requests`、`get_request_initiator`、`search_in_sources`、`save_script_source`、`set_breakpoint_on_text`、`break_on_xhr`、`get_paused_info`、`evaluate_script`。典型流程是先静默打开页面过风控，再启用网络/脚本收集并刷新，最后用请求调用栈和断点证明加密入口。

### Skill：阶段路由和专项技能

`reverse-skill` 的价值在于阶段路由和证据工件：先证明真实请求链，再按 `locate / recover / runtime / validation` 分阶段推进。`ai-reverse-toolkit` 更模块化：`find-crypto-entry` 定位入口，`ast-deobfuscate` 做 Babel AST 解混淆，`env-patch` 提供 Node 补环境经验。

仓库里的大批 `js逆向/.../knowledge.md` 和 `项目资料/` 是案例素材池，不等价于稳定方法论；沉淀 wiki 时只抽模式，不搬整仓。

---

## 二、启动提示词模板

```text
目标页面：
目标请求 / 字段 / cookie / 消息：
触发动作：
当前现象：
已有证据：
目标：
约束：

请按 iv8-first JS 逆向工作流推进：
1. 用真实 Chrome 和 js-reverse-mcp 证明目标请求链，不猜字段来源。
2. 按 writer <- builder <- entry <- source 找到写入边界和上游状态。
3. 判断阻塞属于 locate / recover / runtime / validation 哪一阶段。
4. 需要补环境时，优先用 iv8 的 JSContext environment/config/page.load/eventLoop/netLog/wrapNative。
5. 真实请求由 Python HTTP 层回灌；TLS/JA3 敏感站优先用 curl_cffi impersonate。
6. 交付 sign.py 或等价 Python 调用代码、3 组样本对比、一次真实请求验证、失败边界说明。
```

必填字段解释：

| 字段 | 为什么必须写 |
|------|--------------|
| 目标页面 | `location.href/origin/hostname` 是指纹输入，必须和真实页面一致 |
| 目标请求/字段 | 决定搜索关键词、断点位置和最终验收对象 |
| 触发动作 | 页面加载、点击、翻页、登录提交对应不同请求链 |
| 当前现象 | 区分缺对象、缺状态、反调试、不稳定源、风控分支 |
| 已有证据 | 防止重复侦察，明确哪些样本已经可信 |
| 目标 | 只定位入口、跑通出参、还是交付采集脚本，成本完全不同 |
| 约束 | 是否允许浏览器辅助、是否能用 Pro/商业授权、是否必须纯 HTTP |

---

## 三、六阶段 SOP

### 1. Evidence：请求链证据化

先用真实浏览器拿到目标样本。最小证据包括：目标请求 URL、method、headers、query/body、cookie、响应状态、触发动作、上游请求或状态。不要从一个疑似加密函数直接开挖。

MCP 操作顺序：打开页面 -> `list_network_requests` 找目标请求 -> `get_request_initiator` 取调用栈 -> 必要时 `break_on_xhr` 或 `set_breakpoint_on_text` -> `evaluate_script` 在暂停帧读取变量。

### 2. Locate：写入边界证明

按 `writer <- builder <- entry <- source` 反向收缩：

| 层 | 问题 |
|----|------|
| writer | 值最终写进 header/query/body/cookie/WebSocket frame 的哪里 |
| builder | 哪段逻辑组装、加密、签名、序列化 |
| entry | 哪个事件、响应、回调、SDK 调用启动 builder |
| source | 真实输入来自上游响应、cookie、storage、时间、随机数、指纹还是用户行为 |

搜索只能辅助，不能单独算证明。若 source 依赖前置 cookie、challenge、HttpOnly、storage 或服务端响应，立即回到请求链补证据。

### 3. Recover：壳层压缩

只恢复阻塞下游的最小壳层。Webpack/AST/WASM/Worker/JSVMP 的共同原则是边界优先、最小恢复：能直接从 writer 附近拿到 builder 契约，就不要全量反编译。

AST 解混淆仍在 Node/Babel 侧完成；iv8 是运行时，不替代静态转换。产物可以是可读 JS、最小模块集、bridge 契约，或可被 `page.load` 直接执行的完整 bundle。

### 4. Runtime：iv8 补环境与运行时对齐

运行时问题先分类：

| 类别 | iv8 对应手段 |
|------|--------------|
| missing object | `environment` 声明式覆盖，或 `wrapNative` 补 JS polyfill |
| missing state | `page.load.resources` / `add_resource` 注入响应，补 cookie/storage/challenge |
| anti-debugging | iv8 禁用原生 `debugger;`，用 `vdebugger;` 和 `watch_apis` 反查探测点 |
| unstable source | `eventLoop`、`time_mode`、固定随机种子、固定 device/session seed |
| risk branch | 对比浏览器正常态与 iv8 本地态，找到第一次分叉点 |

iv8-first 原则：先用 `JSContext(environment=..., config=...)` 和 `page.load` 还原页面生命周期；缺口被证实时再引入 JS 层 polyfill。不要把 Node 路线的 fakeWindow、delete Buffer/process、Error.prepareStackTrace 补丁照搬过来。

### 5. Replay：Python HTTP 回灌

iv8 社区版不内置真实网络传输栈。它负责跑 JS 出参数，真实请求由 Python 层完成：

| 任务 | 推荐做法 |
|------|----------|
| TLS/JA3/JA4 敏感 | `curl_cffi.requests.Session(impersonate="chrome")` |
| cookie 绑定 | 全程同一 session，不换请求客户端 |
| XHR hook 改 URL | 在 iv8 内发 dummy XHR，从 `__iv8__.netLog.entries` 收割 signed URL / `cookieHeader` |
| 响应驱动状态 | Python 发真实请求后 `ctx.add_resource(...)` 注入，再 `eventLoop.drain()` |

`ctx.expose(pyfn, "name")` 可以让 JS 直接调用 Python 请求函数，但会持有 GIL，同步阻塞，适合调试，不适合高并发生产。

### 6. Validation：验收与交接

验收优先级：

1. 格式验证：长度、前缀、字符集、字段结构与浏览器一致。
2. 中间检查点：关键 builder 输入输出与浏览器样本一致。
3. 多样本验证：至少 3 组输入对比，避免只撞中一个样本。
4. 真实请求：业务响应正确，而不只是 HTTP 200。
5. 失败边界：记录哪些环境点仍待实测，如 `chrome.runtime`、ServiceWorker、瑞数 AJAX 分支。

交接工件建议写 `PROGRESS.md` 或请求链记录，最少包含：已证实请求链、入口位置、source 状态、iv8 环境画像、必需对象、必需状态、固定源、验证记录、下一步阻塞。

---

## 四、Node 经验到 iv8 的折算

| Node 补环境手段 | iv8-first 等价 |
|----------------|----------------|
| 手工拼 `window/document/navigator` | 内置 C++ 浏览器 API + `environment` 覆盖 |
| `Object.defineProperty(global, "navigator", ...)` | `JSContext(environment={"navigator": ...})` |
| 删除 `Buffer/process/Error.prepareStackTrace` | iv8 独立 Isolate，无 Node 泄露，默认不需要 |
| `Function.prototype.toString` hook | `__iv8__.wrapNative(fn, name)` |
| fake timer / sinon | `__iv8__.eventLoop.advance/sleep/drain` + `time_mode` |
| mock fetch/XHR | `add_resource` / `page.load.resources` / `netLog` |
| webpack runtime 抠模块 | 仍可用，但先判断能否 `page.load` 整 bundle |

保留 Node 经验里的方法论：缺啥补啥、加载顺序、格式优先、最小恢复、状态/对象分离。降权 Node 经验里的实现细节：fakeWindow、大量 global patch、隐藏 Node 全局对象、远程 jsdom/sdenv 路线。

---

## 结论

iv8-first 工作流的核心是把“侦察”和“执行”分离：真实 Chrome 负责证明目标 JS 如何生成参数，[[entities/iv8]] 负责在 Python 进程内高并发跑出参数，Python HTTP 层负责真实网络与风控边缘。最容易失败的地方不是 API 不会调，而是请求链没证明、状态和对象混写、Node 补丁照搬、或者只用 HTTP 200 当验收。

继续查 API 细节看 [[topics/iv8-api-reference]]；查补环境手法看 [[topics/iv8-supplement-env-handbook]]；查站点模板看 [[topics/iv8-reverse-cases]]；查通用壳层方法看 [[topics/js-reverse-general-techniques]]。

## 来源

- `raw/github/HanZzzzz000iv8.md`
- `raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md`
- `https://github.com/715494637/reverse-skill`
- `https://github.com/zhizhuodemao/js-reverse-mcp`
- `https://github.com/zhizhuodemao/ai-reverse-toolkit`
