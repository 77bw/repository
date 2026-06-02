---
title: iv8
type: entity
created: 2026-06-02
updated: 2026-06-02
tags: [spider, tooling, github, mature, contested]
sources:
  - raw/github/HanZzzzz000iv8.md
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
  - https://github.com/HanZzzzz000/iv8
summary: Python 原生 V8 + 浏览器 API 运行时，用于 JS 逆向补环境与高并发跑参
confidence: high
contested: true
contradictions:
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
---

# iv8

## 简介


> ⚠️ **平台限制(据 2026-06 issue #3)**:iv8 社区版仅支持 **Windows x64** 与 **Linux x64**(manylinux),**暂无 macOS 版**。macOS 用户需通过 WSL2 / Linux 容器 / 远程 Linux 机使用。Pro 版平台支持详询 admin@iv8.dev。

iv8 是基于 V8 引擎的高性能 **Python 原生 C++ 扩展**，在 C++ 层原生实现 BOM/DOM/CSSOM，可在 Python 进程内直接运行依赖 Web 环境的 JavaScript，无需启动浏览器。它面向 JS 逆向中的「补环境」场景：把目标站抠出来的加密/签名/反爬 JS 放进一个高保真的浏览器环境跑通，拿到动态参数（token、cookie、签名 URL）后交给 Python HTTP 客户端回放请求。

与 Node 路线（`jsdom` + `Proxy` 魔改 / Headless Chrome CDP）最本质的差异：iv8 的浏览器 API 是 **C++ 层原生提供**，而非 JS 层逐属性 proxy 补丁。README 与 demos 可证常见原型链、属性描述符、`[native code]` 外观与可信输入能力；但闭源二进制缺第三方评测，高强度反射/组合指纹仍需在目标站实测。

本次检索中唯一相关权威实现 = GitHub `HanZzzzz000/iv8` + PyPI 同名包 `iv8`。`gh search repos iv8` 返回的同名结果多为无关项目或低相关噪音，避免误装。这是用户主流逆向方案；Node 补环境路线仅作思路参考。

## 核心功能

### 架构：JSContext → C++ Bridge → v8::Isolate（已核实）

Python 通过 `iv8.JSContext` 进入 C++ Bridge，**每个 Context 独占一个 `v8::Isolate`**，可在多个 Python 线程上并行且无需额外加锁。关键 GIL 模型：

```
Python eval() → 释放 GIL → V8 执行 JS
                          → (JS 调 expose 的 Python 函数 → 重获 GIL → 执行 → 释放 GIL)
              ← 获取 GIL ← V8 返回 → 返回 Python
```

即 V8 执行纯 JS 期间释放 GIL（多线程真并行，8 线程 ~4.71x）；但凡 JS 回调桥接回 Python 的调用会重获 GIL（串行阻塞当前 V8 与其他 Python 线程）。这决定了 `expose` 桥接 HTTP 不适合高并发，应优先用内置 `add_resource`/`netLog`。

### 三大武器

1. **environment 指纹（声明式覆盖）** — 内置默认 200+ 字段（Chrome124/Windows 基线），`environment` dict 直接映射 JS 可观测属性（`environment.navigator.userAgent` → `navigator.userAgent`），覆盖 navigator/screen/`webgl`/`webgl2`/webgpu/audio/battery/`webrtc.ice`/`chrome.loadTimes`/canvas 等常见指纹面。它能降低手写补环境成本，但不等价于通过所有 CreepJS/高对抗组合检测。关键设计是 **environment（浏览器画像，支持数组/嵌套）vs config（引擎行为，仅标量）二分通道**，传错通道会失效。`environment` 创建后不可变，换指纹须新建 Context。`navigator.webdriver` 默认 `False`。

2. **netLog 网络捕获** — `window.__iv8__.netLog.entries` 自动记录 JS 侧全部 XHR/fetch/导航请求，每条含 `method`/`url`/`headers`/`cookieHeader`。配合「发 dummy XHR 让风控自己 hook 改写 URL，再从 netLog 收割带签名的完整 URL/cookie」的模式，是瑞数等 hook-XHR 型风控的核心收割手法。

3. **eventLoop 逻辑时间** — `window.__iv8__.eventLoop` 把事件循环交给 Python 手动驱动：`advance(total,step?)` 按帧推进虚拟时间、`sleep(ms)`、`tick()` 单轮、`drain(max?)` 排空到期任务、`drainMicrotasks()` 仅微任务、`drainTimers()` 仅定时器、`setAutoAdvanceStep`/`setDateAdvanceStep` 控 `performance.now()`/`Date.now()` 自增。配合 `config.time.mode=logical|system`，可瞬间跳过 `sleep`、确定性复现宏/微任务执行顺序，是对抗 PoW / 时间差校验的核心。微/宏任务分级严格对齐 HTML spec。

### 其余反检测原语（已核实）

- `__iv8__.wrapNative(fn,'name')` — 把 JS 函数伪装成 `function name() { [native code] }`；README/demo 可证基础 `toString()` 外观，复杂二阶反射与目标站自定义探测需实测。
- `__iv8__.input.dispatchMouseEvent/dispatchPointerEvent(init)` — 派发 **`isTrusted===true`** 的可信事件（捕获→目标→冒泡链对齐 Chrome），普通 `el.dispatchEvent` 造不出，这是绕过滑块/点击手势校验（如腾讯防水墙 TDC）的硬优势。注意拖拽 API `dragMouse`/`dragPointer` 仅 Pro。
- `ctx.expose(obj,'name')` — Python 对象挂到 JS 的 `__iv8__.data` 命名空间（不污染 `window`）；桥接调用持有 GIL 同步阻塞。
- `ctx.add_resource(url,body,status,headers)` — 离线注入 HTTP 响应；`__iv8__.page.load(snapshot)` 流式装载页面（执行脚本 + 派发 DOMContentLoaded/load + 同步 `location.href`）。
- **debug 模式**：`mode='debug'.with_devtools(port,watch_apis,enable_console)` 启 CDP；原生 `debugger;` 被禁用（防无限 debugger 反调试）改用 `vdebugger;`；`enable_console=False` + `vconsole` 隐蔽日志通道；`watch_apis` 在指定指纹访问处自动断点回溯调用栈；自动拦截 `Object.keys`/`Reflect`/`Function.prototype.toString` 等反射探测路径——这是定位「目标 JS 在探测哪个环境点」的杀手锏。

## 关键事实与时间线

### 时间线（已核实，是裁决版本矛盾的关键基线）

| 日期 | 事件 |
|------|------|
| 2026-04-07 | GitHub 仓库 `HanZzzzz000/iv8` 创建 |
| 2026-04-22 | PyPI 首次出现下载 |
| 2026-04-29 | README 现有内容最后定稿 |
| 2026-05-07 | 仓库最后 push |
| 2026-05-21 | 微信文章发布（**晚于** README 定稿与最后 push） |

时间线推论：当前 README = 微信文章发布前的 README，任何「文章发布后才补上某能力」的假设在时间上不成立。

### 性能数据（已核实，作者自测 i7-14700 / Win10 / Py3.11）

- 创建 + eval + 销毁 ~3.3ms/次（~300 ctx/s），官方明示无需刻意复用，每次新建得干净状态。
- 简单 `eval(1+1)` ~950,000 ops/s；浏览器 API 调用 340,000–570,000 ops/s。
- 真实网页 DOM 解析（维基百科 ~440KB）~7ms（含创建销毁 ~11.5ms）。
- 内存（iv8 边际增量）：首次 +15MB；100 轮长跑累计仅 +2MB（无明显泄漏）。
- 多线程加速比：2/4/8 线程 = 1.86x / 3.26x / **4.71x**（GIL 释放兑现，相对 Node 子进程方案的吞吐优势）。
- 调优第一原则：只需提数据用 `innerHTML`（仅建 DOM 树），需脚本+生命周期才用 `page.load`（开销高）。

> 热度校准：PyPI `iv8` 0.1.2，40 天下载 3675 / 近 30 天 2112；GitHub star 数在 2026-06-02 快照约 210–212，属于会漂移的热度指标。项目 2026 年 4 月才发布，**无第三方独立评测**，性能/反检测数据均为作者自述，需自行验证。

### 社区版 vs Pro 边界（以 README 为准，已核实）

- **社区版**（本仓库，免费预编译二进制）：基础浏览器环境模拟，覆盖绝大多数日常场景。**不内置真实网络传输栈**——XHR/fetch/外联资源走离线 bundle，真实请求由用户 Python HTTP 客户端完成再 `add_resource` 注入。这是「iv8 算参数 + Python 管网络」分工的底层原因，也是社区版最大的非自动点。
- **Pro 版新增**：① CSS 布局引擎（级联/继承/盒模型）② CSS 动画与过渡 ③ 基于 Chromium net 深度裁剪的**真实协议栈（明确非 Cronet 封装）**，可直发带 Chromium 级协议指纹的请求 ④ 多上下文 Worker 并行执行 ⑤ API 语义/时序/边界对齐增强 ⑥ 性能/内存深度优化 ⑦ `input.dragMouse`/`dragPointer` 拖拽 API。社区版持续迭代，Pro 成熟特性逐步回流；作者当前主力开发「布局树」（issue #1）。

### 授权：闭源商业（已核实，高优先级约束）

`iv8 Community Edition License (Copyright (c) 2025 iv8 Authors)`，**非 SPDX 标准开源许可，是私有 EULA**：社区版以预编译二进制免费提供，仅限个人/教育/非商业用途；明确禁止反编译/反汇编/逆向获取源码、再分发/转售、移除版权声明；商业用途/OEM/企业部署须单独商业授权（联系 `admin@iv8.dev`，issue #4）；违约即自动终止须销毁副本。

落地影响：① 商业爬虫项目用社区版有法律风险；② 禁反编译意味着无法对二进制做静态分析补全文档未覆盖行为，只能黑盒 + 官方文档；③ 二进制不可自行修补 bug。

### 与 STPyV8 的渊源（已核实）

iv8 README acknowledgments 明确：Python↔V8 互操作层**参考 STPyV8（`cloudflare/stpyv8`，Apache-2.0）的设计思路**并在此基础上优化。本质差异：STPyV8 = 裸 V8 绑定，本身不含任何 browser/DOM/BOM 模拟，浏览器环境须用户自建（`JSClass` 注入 global）；iv8 = STPyV8 式绑定 **+ 自建 C++ 原生 BOM/DOM/CSSOM 模拟层**（70+ HTML 元素、25+ CSS 规则、80+ 事件类型、crypto/canvas/webgl）。iv8 替代的是「STPyV8/PyMiniRacer + 手写补环境代码」这套组合。

### 浏览器 API 覆盖与争议点

README 声明覆盖 DOM/HTML/SVG/CSSOM/事件/网络/存储/加密/Canvas/媒体/定时器/Workers 等，但总括标注「**部分为接口级桩实现**」。三处与微信文章的矛盾，以实时仓库为准裁决：

- **WebGL/WebGL2 — 文章“完全缺失”表述不准确。** README 明列 Canvas2D / WebGL/WebGL2（30+ 扩展，参数来自 `environment.webgl.*`）/ WebGPU / OffscreenCanvas，`demos/full_configuration.py` 给出 VENDOR/RENDERER/UNMASKED_*/SHADER_PRECISION/大量 MAX_* 等可配字段；该段内容早于微信文章。可确认的是“字段与上下文能力存在、可声明式配置”，但真实 GPU 渲染语义、Canvas 哈希稳定性和 CreepJS 类强检测仍需目标站实测。
- **`window.chrome` — 对象存在；`chrome.runtime` 待核实（contested）。** `environment.chrome.loadTimes` 是官方可配字段（navigationType/npnNegotiatedProtocol/connectionInfo/wasFetchedViaSpdy/...），故文章「完全缺 `window.chrome`」不准确。但 headless 检测常查的 `chrome.runtime` 子对象在 README/demos 中无任何明证，无法证实也无法证伪。
- **ServiceWorker — 灰区（contested）。** README 列出 `Worker/SharedWorker/ServiceWorker/Worklet` 接口，但 line 101 自承「部分为接口级桩」，line 19 注明「多上下文 Worker 并行」属 Pro 独有，demo 仅有一个 SW 注册计数的存储估算默认值。功能可用性层面文章「缺 ServiceWorker」方向正确（社区版无完整 SW），但措辞过度——接口存在而非完全缺失。社区版实际可用深度需实测。

### 已知坑（issues 实证，落地前必看）

- **#7** `btoa` 对非 Latin1 字符报 `characters outside of Latin1 range`，作者确认是输入字符范围判断过窄 bug，多人复现 [open]，暂只能外部 hook `btoa`。
- **#2** `page.load` 后 `document.write()` 内容不持久化（`getElementById` 返回 None）[open]，官方临时方案：`page.load` 前用 `__iv8__.wrapNative` 在 document 原型 hook `write`/`writeln` 改用 `insertAdjacentHTML`。
- **#8** DevTools 打印 `localhost` 但底层 WS 监听 IPv4，部分 Win/Chrome 下 `localhost` 解析到 IPv6 致连上即断 [open]，解法改用 `ws=127.0.0.1:9229`。
- **#3** 无 macOS 版，仅 Win/Linux x64 [open]，作者计划接 GitHub Actions 出 mac arm64。
- **#9** 瑞数站 AJAX 翻页 cookie 长度差异（iv8 364 字符 vs 真机 449）[closed 无公开结论]，推测某环境检测点未过致 JS 走降级分支——这是社区版指纹完整度边界的信号。

`examples/` 含实战样本（注：原任务标签有误，按实际站点归类）：海关/税务/欧冶/药监局（瑞数 rs）、`zp_stoken.py`（**BOSS 直聘**，非智联）、`abogus.py`（抖音 a_bogus）、`h5st.py`（京东，须 `curl_cffi` 管 TLS）、`tdc.py`（**腾讯防水墙验证码**，非瑞数）。作者立场（#6/#9）：接受运行时兼容问题/缺失 API/最小复现，但不为特定网站提供完整绕过方案。

## 相关实体与概念

- [[concepts/js-reverse-supplement-env]] — 补环境的引擎无关通用思维（声明式覆盖 / `[native code]` 伪装 / 隐藏注入对象 / 时间控制 / 离线资源解耦），iv8 三大武器即其声明式落地
- [[topics/iv8-supplement-env-handbook]] — iv8 补环境实战指南（6 阶段 SOP、瑞数两跳模板、模式 A/B、`MessageChannel` polyfill 等）
- [[entities/reverse-skill]] — JS 逆向技能/工具集锚点（iv8 在其中扮演离线脱机执行引擎）

> 同生态但暂未建页（纯文本提及）：**STPyV8**（直接上游参考）、**curl_cffi**（TLS/JA3 指纹层，与 iv8「算参数」分工「发包」）、**js-reverse-mcp**（真实 Chrome 在线插桩侦察，与 iv8 离线执行互补）。

## 来源

- `raw/github/HanZzzzz000iv8.md` — 官方仓库 README（架构/API/性能/Pro 边界/API 覆盖），权威一手来源
- `raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md` — 第三方介绍文（2026-05-21），WebGL/`window.chrome`/ServiceWorker 三处与仓库矛盾，已按时间线裁决以仓库为准 ^[raw/github/HanZzzzz000iv8.md]
- 旁证（2026-06-02 快照）：GitHub `HanZzzzz000/iv8`（约 210–212 stars，2026-04-07 创建）、PyPI `iv8` 0.1.2、`cloudflare/stpyv8`、issues #1–#9 与 `examples/`/`demos/` 目录
