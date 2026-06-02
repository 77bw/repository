---
title: iv8 API 速查
type: topic
created: 2026-06-02
updated: 2026-06-02
tags: [spider, tooling, sop, mature]
sources:
  - raw/github/HanZzzzz000iv8.md
  - https://github.com/HanZzzzz000/iv8/tree/main/demos
summary: JSContext、environment/config 与 __iv8__ 工具对象的 iv8 调用速查表
confidence: high
---

# iv8 API 速查：JSContext / environment / __iv8__

> 配套 [[topics/iv8-supplement-env-handbook]]（补环境方法论）与 [[topics/iv8-reverse-cases]]（实战案例）。本页只保留 API / 字段 / 调用方式，方便 AI 查表。

## 概述

iv8 的配置分两层：`environment` 描述浏览器画像（JS 可观测属性），`config` 描述引擎行为；JS 侧工具统一挂在 `window.__iv8__`。全量字段以 `iv8.JSContext.get_defaults()` 运行时枚举为准。

## 分析

### API 速查

#### 1.1 `JSContext` 构造参数（6 个）

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `mode` | str | `"prod"` | `"prod"` 零监控开销 / `"debug"` 启用 API 调用链追踪 + 反射拦截 + DevTools |
| `environment` | dict | `None` | **浏览器画像**（指纹），映射 JS 可观测属性，支持标量/数组/嵌套，创建后不可变 |
| `config` | dict | `None` | **引擎行为**配置，仅标量，与 environment 独立通道 |
| `ignore_apis` | list | 内置默认 | 从 debug 监控日志中排除的 API 列表（默认对 Math/JSON/Array/类型化数组静音） |
| `time_mode` | str | `"logical"` | `"logical"` 逻辑时间（`sleep(5000)` 瞬间完成）/ `"system"` 系统时间锚定（POW/时间差校验用）。等价于 `config.time.mode` |
| `js_api` | str | `"__iv8__"` | JS 侧工具对象挂载名，可改名防检测 |

#### 1.2 `JSContext` 方法表

| 方法 | 签名 / 关键点 |
|------|--------------|
| `eval` | `eval(source, name="", line=-1, col=-1, to_py=False, devtools=True)`。`name` 给脚本赋真实 URL（影响错误栈/某些指纹探测）；`to_py=True` 递归转 Python 嵌套对象；`devtools=False` 该次 eval 跳过调试器 |
| `close` | `close(gc="none")`。`gc` 可选 `"low_memory"`（或 `"v8"`/`True`）/ `"aggressive"` 触发 GC。推荐用 `with` 上下文管理器自动释放 |
| `add_resource` | `add_resource(url, body, status=200, headers=None)`。注入离线 HTTP 响应到 resource bundle |
| `with_devtools` | `with_devtools(port=9229, watch_apis=None, enable_console=True)`。**返回 self 可链式**：`JSContext(mode='debug').with_devtools(...)` |
| `expose` | `expose(obj, name?)` / `expose(**kwargs)`。注入 Python 对象到 JS 的 `__iv8__.data` 命名空间（不污染 window） |
| `get_defaults` | 类方法，返回所有受支持 environment/config 路径及默认值的扁平 dict，是动态枚举可配项的权威来源 |

#### 1.3 `environment` 字段全清单（浏览器画像，映射 JS 可观测属性）

`environment` 按 dot-path 读取，未指定字段走内置 200+ 默认值（Chrome124/Win64 基线）。映射规则：`environment={'navigator':{'userAgent':'X'}}` → JS `navigator.userAgent == 'X'`。支持标量与数组。

| 顶层域 | 关键字段（节选，全量用 `get_defaults()` 枚举） |
|--------|-----------------------------------------------|
| `location` | protocol / hostname / port / pathname / href / search / hash / origin |
| `navigator` | userAgent / appVersion / platform / vendor / productSub / language / **languages[数组]** / doNotTrack / onLine / cookieEnabled / **webdriver(默认 False)** / pdfViewerEnabled / hardwareConcurrency / maxTouchPoints / deviceMemory；嵌套 `userAgentData`{platform/architecture/bitness/model/platformVersion/mobile/wow64}、`connection`{type/effectiveType/downlink/rtt/saveData}、`plugins.enabled` |
| `window` | innerWidth/Height / outerWidth/Height / screenX/Y / devicePixelRatio / isSecureContext / crossOriginIsolated / originAgentCluster |
| `screen` | width / height / availWidth/Height / colorDepth(24) / pixelDepth(24) / isExtended / orientation{type,angle} |
| `document` | domain / referrer / readyState / visibilityState / lastModified / designMode |
| `media`（matchMedia） | prefersColorScheme / prefersReducedMotion / forcedColors / colorGamut / pointer / hover / anyPointer 等全套 |
| `webgl` | README/demos 可证的 WebGL 指纹配置：VENDOR / RENDERER / VERSION / SHADING_LANGUAGE_VERSION / UNMASKED_VENDOR_WEBGL / UNMASKED_RENDERER_WEBGL / 大量 MAX_* / `SHADER_PRECISION` 精度表；真实 GPU 语义与强检测需实测 |
| `webgl2` | 约 23 个 MAX_*/MIN_* uniform/varying/transform-feedback/draw-buffer 指纹参数 |
| `webgpu` | preferredCanvasFormat / wgslLanguageFeatures / adapterFeatures / adapterInfo / adapterLimits(约 35 字段) |
| `chrome` | `loadTimes`{navigationType/npnNegotiatedProtocol/connectionInfo/wasFetchedViaSpdy/wasNpnNegotiated/wasAlternateProtocolAvailable} — 即 `window.chrome` 对象**确实存在** |
| 其他 | audioContext / visualViewport / storage{usage,quota} / geolocation / performance{navigation,memory} / batteryManager / history / credentials / clipboard / `webrtc.ice`{defaultHostIPv4,defaultMdnsHostname} / video / html.img / `canvas`{blob, fingerprint.toDataURL.{png,jpeg,webp}} / managed(企业设备) |

> 反爬重灾区是 `webgl.*`、`webrtc.ice.*`、`chrome.loadTimes`、`canvas.fingerprint.*` —— 声明式覆盖这几组即可对抗大多数指纹采集。

#### 1.4 `config` 字段全清单（引擎行为，与 environment 独立）

`config` 只支持标量（str/int/float/bool），**数组类必须走 `environment`**（如 `navigator.languages`）。Python key 中的 `.`（如 `csp.reportOnly`）会拼进内部 dot-path。

| 分组 | 字段 |
|------|------|
| `time.mode` | `logical` \| `system` |
| `fingerprint.seed` | int，**0 = 固定 seed 确定性复现噪声**（canvas/timing 可复现，便于 diff）；省略 = isolate-local 随机 |
| `security` | csp / csp.reportOnly / csp.upgradeInsecureRequests / csp.blockAllMixedContent |
| `cookies.blockThirdParty` | bool |
| `wasm.streaming.strictMime` | bool |
| `webgl.fingerprint` | mode(`clear`\|`seeded`) / seed / data；外加 GPU_DISJOINT_EXT / contextLost |
| `canvas` | toDataURL；context.{2d,webgl,webgl2,bitmaprenderer}.enabled；toBlob / transferControlToOffscreen / captureStream |
| `webgpu.requestAdapter.available` | bool |
| `streams.readable.defaultChunkSize` | int(默认 65536) |
| `media.canPlayType.proprietaryCodecsEnabled` | bool |
| `permissions.*` | 值域 `granted`\|`denied`\|`prompt`，影响 `navigator.permissions.query()` |
| `iframe.parentOrigin` | str |
| `features.profile` | 默认 `chrome124_win` + 约 35 个 Chromium Feature Flag 布尔（FencedFrames / SharedArrayBufferEnabled / Fledge / AttributionReporting 等） |
| `modelExecution.genericSession` | availability / responseMode / fixedResponse / defaultTopK / defaultTemperature |

> **二分法（iv8 独有设计，传错通道会失效）**：`environment` = 浏览器长什么样（画像）；`config` = iv8 引擎怎么跑（行为）。

#### 1.5 JS 侧 `__iv8__` 工具对象全表

创建 Context 时全局挂 `window.__iv8__`（可经 `js_api` 改名）。README 将其设计为对目标 JS 隐身；遇到强反射检测时仍应在目标站实测，必要时用 `js_api` 改名降低可观测风险。

| 工具 | 能力 |
|------|------|
| `__iv8__.eventLoop.advance(total, step?)` | 分帧推进虚拟时间（默认步长 ~16.67ms 模拟 rAF） |
| `__iv8__.eventLoop.sleep(ms?, max?)` | 推进虚拟时间并按时间线排空任务 |
| `__iv8__.eventLoop.tick(ms?)` | 推进 ms 并执行一轮事件循环（单步调试） |
| `__iv8__.eventLoop.drain(max?)` | 排空所有已到期任务（宏+微），不推进时间 |
| `__iv8__.eventLoop.drainMicrotasks()` | 仅排空微任务（Promise.then / queueMicrotask / MutationObserver） |
| `__iv8__.eventLoop.drainTimers()` | 仅处理已到期定时器 |
| `__iv8__.eventLoop.setAutoAdvanceStep(ms)` | 设 `performance.now()` 自增量（默认 0.001ms） |
| `__iv8__.eventLoop.setDateAdvanceStep(ms)` | 设 `Date.now()` 自增量（默认 1ms）—— 对抗时间差校验 |
| `__iv8__.page.load(snapshot)` | 流式加载 HTML（见下表参数） |
| `__iv8__.input.dispatchMouseEvent(init)` | 派发 `isTrusted=true` 鼠标事件 |
| `__iv8__.input.dispatchPointerEvent(init)` | 派发 `isTrusted=true` 指针事件 |
| `__iv8__.netLog.entries` | 自动记录 JS 侧全部 XHR/fetch/导航，每条含 method/url/headers/body/cookieHeader |
| `__iv8__.wrapNative(fn, name)` | 把 JS 函数伪装为 `function name() { [native code] }` |
| `__iv8__.data.*` | `expose()` 注入的 Python 对象命名空间 |
| `__iv8__.help()` | 打印所有工具 |

**`page.load(snapshot)` 参数**：`baseURL`(必填，同步 document.URL/location.href) · `html`(必填) · `resources`(选填，URL→内容映射，`<script src>`/`<link>`/运行时 XHR-fetch 都从此匹配) · `headers`(选填，主文档响应头 CSP/Set-Cookie)。

宏/微任务分级（对齐 HTML spec）：**宏** = setTimeout / setInterval / rAF / XHR-fetch 回调；**微** = Promise.then / queueMicrotask / MutationObserver。

#### 1.6 反调试替代工具

| 原生 | iv8 替代 | 原因 |
|------|----------|------|
| `debugger;` | `vdebugger;` | iv8 禁用原生 `debugger;`（防目标 JS 无限 debugger 反调试循环），vdebugger 行为与标准一致 |
| `console.*` | `vconsole.*`（配 `enable_console=False`） | 防反爬用 console 行为差异检测调试环境，vconsole 仅 DevTools 可见、对目标 JS 隐身 |

## 结论

日常补环境优先查三张表：`JSContext` 构造参数、`environment/config` 字段、`__iv8__` 工具对象。真正落地时先调用 `get_defaults()` 校准当前版本，再按 [[topics/iv8-supplement-env-handbook]] 的「发现→补→验证」闭环推进。

## 来源

- `raw/github/HanZzzzz000iv8.md`
- HanZzzzz000/iv8 `demos/full_configuration.py` 与网络相关 demos（2026-06 实读）
- [[entities/iv8]]
