---
title: JS 逆向补环境
type: concept
created: 2026-06-02
updated: 2026-06-02
tags: [spider, methodology, mature]
sources:
  - raw/github/HanZzzzz000iv8.md
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
summary: 让目标 JS 以为仍在真实浏览器中运行，以产出 cookie、签名或 token
confidence: high
---

# JS 逆向补环境

## 定义

补环境（environment supplementation / 环境补全）是 JS 逆向工程中的一类技术：把一段原本依赖浏览器宿主环境的目标 JS（通常是加密、签名或风控 SDK）从浏览器里抠出来，放到非浏览器的 JS 执行环境（Node.js、原生 V8、或 headless 浏览器）里跑通，并为它补齐所探测和依赖的宿主对象（`window` / `document` / `navigator` / `screen` / `canvas` / `webgl` 等）。

本质是**让目标 JS 以为自己仍跑在真实浏览器里**——只要它探测到的环境与真机一致，就会照常吐出 cookie / 签名 / token，而无需真正启动一个浏览器。

## 核心要点

### 为什么需要补环境

- 风控/反爬 JS 在浏览器端大量采集环境指纹（`navigator.webdriver`、`navigator.plugins`、`window.chrome`、Canvas/WebGL 指纹、`Function.prototype.toString` 的 `[native code]`、屏幕参数、UA 与 platform 一致性等），据此铸造 cookie 与签名（如瑞数 cookie、Akamai `_abck`、Cloudflare `cf_clearance`、抖音 `a_bogus`、京东 `h5st`、BOSS 直聘 `__zp_stoken__`）。
- 纯 HTTP 客户端（如 requests）拿不到这些**运行时产物**——它们由 JS 在浏览器环境里现场算出，所以必须真正执行那段 JS。
- 完整启动真实浏览器太重、太慢、难并发。补环境是「只跑 JS、伪造它需要的环境」的轻量折中。

### 补环境的本质（五条铁律）

不是把属性补上就行，要同时满足：

1. **补值**——属性存在且值合理（`navigator.webdriver = false`）。
2. **补类型**——返回正确的对象结构（`PluginArray` / `Plugin` / `MimeType` 非空且形态对）。
3. **补原型**——原型链正确，经得起 `Object.getPrototypeOf` / `instanceof` 校验。
4. **防 toString 检测**——被 hook 的原生函数 `toString()` 仍要返回 `function name() { [native code] }`，且 `toString` 自身防递归探测。
5. **跨字段一致性**——UA / platform / screen / 时区互不矛盾（`Win32` 配 `Windows NT 10.0`）。

发现要补什么靠**「缺啥补啥」迭代法**：用 `Proxy` 包裹环境对象，在 `get` / `has` / `getOwnPropertyDescriptor` 等 trap 上打日志，跑一遍目标 JS 看它探测了哪些属性，缺失项返回会记录的占位再精确补。盲目补一大堆反而制造新的可识别指纹。

### 三条技术路线（中立对比）

| 维度 | Node 手动补（jsdom 等） | Headless Chrome + CDP | iv8 原生 V8 |
|------|------------------------|----------------------|-------------|
| 宿主对象来源 | JS 层手写 / jsdom + Proxy 逐属性补 | 真实 Chromium，原生齐全 | C++ 层原生实现 BOM/DOM/CSSOM |
| 保真度 | 低–中，易在原型链 / toString 穿帮 | 最高（真浏览器） | 较高；README/demo 可证常见面，强对抗组合指纹仍需实测 |
| 性能 / 并发 | 轻，但补环境工作量大 | 重（进程级），并发成本高 | 轻（约 3.3ms/context，8 线程约 4.7x） |
| `isTrusted` 可信事件 | 造不出（`dispatchEvent` 恒 `false`） | 真实 | `input.dispatch*` 派发 `isTrusted=true` |
| 反调试 | 需手 patch `debugger;` | DevTools 暴露面大 | 内置禁用 `debugger;`，改用 `vdebugger` |
| TLS / JA3 指纹 | 不解决（另配 curl_cffi 等） | 浏览器自带真实 TLS | 不解决（需自接 curl_cffi 等） |
| 语言生态 | Node.js | 任意（经 CDP） | Python 原生 |
| 典型代表 | jsdom、sdenv、vm2（已停维护） | Puppeteer / Playwright + stealth、Patchright | iv8 |

> Node 路线的具体手段（jsdom 补 DOM、`Proxy` / `defineProperty` 劫持、AST 去 `debugger`、`sinon` fake timers、手写 `toString` 伪装）属于**引擎特定实现**，仅作思路参考；其中「缺啥补啥」「native 伪装」「时间控制」「一致性校验」等**思维是引擎无关的，可直接复利**到任何路线。

### iv8 路线的定位优势

[[entities/iv8]] 把补环境从「JS 层手写桩」升级为「C++ 层原生 API」：浏览器 API 在 C++ 实现而非 JS 层 proxy 补丁，README 与 demo 可证常见原型链、属性描述符与 `[native code]` 外观能力，能省去 Node 路线大量手工补丁；但仓库闭源且缺第三方独立评测，CreepJS 类强对抗组合指纹仍要以目标站实测为准。同时它比真浏览器轻得多（无进程开销、每 Context 独占 Isolate 可多线程真并行）。`isTrusted=true` 可信事件、内置反调试（`vdebugger`/`vconsole`）也是纯 JS 注入路线很难补出的优势。^[raw/github/HanZzzzz000iv8.md]

代价同样要写清楚：社区版闭源、不可审计、不内置真实网络传输栈（请求需自接 Python HTTP 客户端再注入响应）；部分高级接口（如完整 Worker / ServiceWorker）在社区版为接口级桩、实际可用深度需实测；目前仅 Windows / Linux x64、无 macOS 版。详见 [[entities/iv8]] 与 [[topics/iv8-supplement-env-handbook]]。^[raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md]

### 检测面与对抗

- **属性级**：`navigator.webdriver` / `plugins` / `mimeTypes`、`window.chrome(.runtime)`——补结构正确的对象，而非只补一个数字或布尔。
- **渲染级**：Canvas `toDataURL` 哈希、WebGL `vendor`/`renderer`（`WEBGL_debug_renderer_info`）、AudioContext——纯 JS 引擎无真实 GPU 渲染，需返回稳定可信的预录真机哈希。
- **一致性级**：`Function.prototype.toString` 的 `[native code]`、UA vs platform、CreepJS 式跨信号比对。
- **驱动注入物**：`cdc_*` / `_selenium` / `$cdc_...`——这些是 Chromedriver/Selenium 注入物，纯 JS 执行环境天然没有，**注意别误补出来**反而暴露。
- **补环境覆盖不到的**：行为信号（鼠标 / 键盘 timing）、边缘层 TLS / JA3 / JA4（归 curl_cffi 等 impersonate 客户端）、Cloudflare Turnstile 的 PoW + 行为校验（纯补环境性价比低，优先考虑真实浏览器或打码方案）。

## 相关概念

- [[entities/iv8]]
- [[topics/iv8-supplement-env-handbook]]
- [[topics/tdd-ddd-data-collection]]
- [[topics/prompt-mentor-case-study]]

## 来源

- `raw/github/HanZzzzz000iv8.md`
- `raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md`
