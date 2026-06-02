---
title: reverse-skill
type: entity
created: 2026-06-02
updated: 2026-06-02
tags: [spider, ai-coding, tooling, methodology, github]
sources:
  - https://github.com/715494637/reverse-skill
  - raw/github/HanZzzzz000iv8.md
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
summary: 面向 AI Agent 的 JS 逆向中文方法论与 jsr-reverse skill 仓库
confidence: medium
---

# reverse-skill

## 简介

reverse-skill 是一个**面向 AI agent 的 JS 逆向 skill 仓库**，把"侦察 → 定位加密函数 → 补环境 → 跑出参数 → 回灌真实请求"这条 JS 逆向流水线封装成 AI 可直接调用的能力包。它通过单一入口 `jsr-reverse` 触发，运行时**依赖 [[entities/js-reverse-mcp]]** 提供真实浏览器插桩（CDP 层断点、调用栈、网络抓取），把"目标 JS 在哪、怎么调、参数长啥样"交给 MCP 侦察，把"脱机跑通算法、批量出参"交给补环境引擎。

reverse-skill 默认走 **Node 补环境路线**（jsdom / sdenv / Proxy 劫持那套心智）；本库用户的主流落地引擎是 [[entities/iv8]]（Python 原生 V8 扩展），因此**默认 Node 手段需折算到 iv8 等价方案**使用 —— 引擎无关的通用思维直接复利，Node 特定补环境手段则映射到 iv8 内置 API（见下文等价表）。

> 诚实标注：本页对 reverse-skill **仓库本体**的定位来自 `README.md` 与 `jsr-reverse/SKILL.md` 一手实读；依赖组件（js-reverse-mcp、iv8、examples 案例）经 GitHub / PyPI / npm 核实。仓库未在本库 `raw/` 留存独立摘录，故整体仍保守标为 `confidence: medium`。

## 核心功能

### jsr-reverse 阶段主干（仓库本体一手核实）

`jsr-reverse/SKILL.md` 的核心不是一组底层工具，而是一条工程状态驱动的流程主干：

```text
intake -> evidence -> locate -> recover -> runtime -> validation -> handoff
```

关键规则：

- 先用 intake 固定 URL、目标字段/请求、触发动作、现象、证据、目标与约束。
- evidence gate 在请求链仍被猜测时必须先跑，并更新 `reverse-records/请求链路.md`。
- `locate/recover/runtime/validation` 由当前工程状态选择，不由 `412/token/worker/wasm/basearr` 这类线索词直接决定。
- 每次阶段切换输出 handoff card，并只加载当前阶段最小 reference 集。

### js-reverse-mcp 依赖能力（21 个工具，5 类）

reverse-skill 安装前置是 [[entities/js-reverse-mcp]]。这些是 MCP 提供给 skill 调度的侦察工具，不是 reverse-skill 本身实现的补环境能力：

| 能力 | 工具（节选） | 用途 |
|------|------|------|
| 页面导航 | `select_page` / `new_page` / `navigate_page` / `select_frame` / `take_screenshot` | 驱动真实 Chrome 打开目标页 |
| 脚本分析 | `list_scripts` / `get_script_source` / `save_script_source` / `search_in_sources` | 拉取/全文搜索目标 JS |
| 断点执行控制 | `set_breakpoint_on_text`（压缩代码按文本下断）/ `break_on_xhr` / `get_paused_info` / `step` | 在加密函数处断下、单步 |
| 网络与 WS | `list_network_requests` / `get_request_initiator`（取请求的 JS 调用栈）/ `get_websocket_messages` | 定位请求由哪段 JS 发起 |
| 运行时检查 | `evaluate_script` / `list_console_messages` | 在页面上下文求值、读 console |

`get_request_initiator` 取调用栈 + `set_breakpoint_on_text` 对压缩代码按文本下断，是定位加密入口的杀手锏。

### jsr-reverse —— 唯一入口

仓库 README 明确推荐唯一入口：`jsr-reverse`。它面向真实浏览器样本、请求链证据、桥接契约、运行时事实与检查点验证；使用者不需要逐个记忆底层 MCP 工具。

### 侦察层与补环境层的边界

- **MCP 侦察层**：由 [[entities/js-reverse-mcp]] 负责，默认有头驱动本机真实 Chrome；`--cloak` 是强反爬时的可选升级。
- **补环境运行时层**：reverse-skill 原始资料多以 Node/sdenv 心智表达；本 wiki 的落地主线折算到 [[entities/iv8]]，用 `environment/config/page.load/eventLoop/netLog/wrapNative` 跑离线出参。

## 关键事实与时间线

### 依赖组件核实

- **js-reverse-mcp**（zhizhuodemao/js-reverse-mcp）：2026-06-02 GitHub 快照约 1.6k stars / 230 forks，TypeScript，Apache-2.0，npm 包 `js-reverse-mcp` 最新 **2.0.2**，自我定位"为 AI Agent 设计、内置反检测、基于 chrome-devtools-mcp 重构"。
- **iv8**（HanZzzzz000/iv8，**已核实**）：补环境落地引擎，社区版闭源预编译二进制（**禁反编译 + 仅个人/教育/非商业**，商业需 Pro），仅 Windows/Linux x64、**暂无 macOS**。详见 [[entities/iv8]]。

### 安装方式（已核实命令）

配置依赖 MCP（reverse-skill 运行前置）：

```bash
# Claude Code
claude mcp add js-reverse npx js-reverse-mcp
# Codex（注意 -- 分隔符）
codex mcp add js-reverse -- npx js-reverse-mcp
```

- 系统要求：**Node.js v20.19+** 与 **Chrome stable**。
- 强反爬站点：args 加 `--cloak`，并先 `npx cloakbrowser install` 预下载约 200MB。
- 补环境引擎 iv8（Python 侧）：清华镜像未收录，需 `pip install -i https://pypi.org/simple iv8`。

### 项目资料与案例池：keep / skip 判断

reverse-skill 仓库自带的 `项目资料/` 是 Node/sdenv/rs/ai-reverse-toolkit 等参考池，不等于 iv8 官方案例。iv8-first 落地的站点模板主要来自 `HanZzzzz000/iv8/examples`，二者要分开使用：reverse-skill 学阶段路由和证据工件，iv8 examples 学离线出参代码。

**KEEP（有料，可直接套模板）**

- `HanZzzzz000/iv8/examples/海关.py` + `税务.py` —— **瑞数 rs 两跳 canonical 模板**（最高复利）：load → sleep → 收 cookie → 带 cookie 重 load → XHR 触发 hook → `netLog.entries[-1]` 收割 signed-url + `cookieHeader`。政府/国企站最常见风控。
- `欧冶.py` —— 瑞数单跳，演示 `innerHTML + 手动分段 eval` 精确控制脚本执行顺序（rs 对顺序敏感时优于 page.load）。
- `abogus.py` —— 抖音 a_bogus：`MessageChannel` polyfill + **模式 B**（发 dummy XHR 让 SDK 自 hook 改 URL，再从 netLog 收割）。
- `h5st.py` —— 京东 h5st：**模式 A**（入口已定位时直调 `ParamsSignMain._$sdnmd().h5st`）+ **curl_cffi 管 TLS**（京东对 JA3 敏感，stdlib requests 不够）。
- `tdc.py` —— 腾讯防水墙滑块：**isTrusted=true 可信输入**（`input.dispatchPointerEvent` + `eventLoop.sleep` 还原轨迹），纯 JS hook 补不出 isTrusted，是 iv8 的优势点。
- `zp_stoken.py` —— BOSS 直聘 `__zp_stoken__`：response-driven challenge（code=37 → seed/name/ts → 动态 JS 文件名）+ canvas 指纹配置槽。
- `药监局.py` —— **最小化补环境反面教材**：签名能纯 Python MD5 算的就别上 V8，iv8 只用来拿那个 JS 种的反爬 cookie。

> 站点归类订正（**已核实**）：`tdc.py` 是**腾讯防水墙**非瑞数；`zp_stoken.py` 是 **BOSS 直聘**非智联。建簇 B 子页时按真实站点归类。

**SKIP（噪音）**

- reverse-skill `js逆向/...` 大量链接汇总与 `knowledge.md` —— 适合作题库/材料池，不直接等同稳定 SOP。
- reverse-skill `项目资料/sdenv-main`、`rs-reverse-main`、`js-reverse-automation--skill-main` —— 主要提供 Node/sdenv/rs 路线参考；进入本 wiki 时只抽方法论，不照搬实现。
- iv8 `examples/js` 下 fixture（bdms 147KB / jd_index 151KB / js_security_v3 241KB）—— 是逆向**对象**非方法论，需自行从目标站抓取，不入库。
- GitHub 同名 `iv8` 结果 —— 多为低相关项目或噪音；本轮唯一相关实现是 `HanZzzzz000/iv8`，勿误装。

### 默认 Node 路线 → iv8 等价折算

reverse-skill 默认演示 Node 补环境心智，用户折算到 iv8 落地 ^[raw/github/HanZzzzz000iv8.md] ^[raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md]：

| 引擎无关思维（直接复利） | Node 默认手段 | iv8 等价方案 |
|------|------|------|
| 缺啥补啥探针 | Proxy 包裹 window/navigator 打日志 | iv8 内 V8 原生 Proxy 同款探针 |
| 浏览器 API 声明式覆盖 | 手写 getter/jsdom | `environment` dict（C++ 层，不可被 JS 探测）|
| 函数 native 伪装 | `Function.prototype.toString` Proxy hook | `__iv8__.wrapNative` → `[native code]` |
| 时间控制对抗 POW | sinon fake timers | `eventLoop.advance/drain` + `time.mode=logical` |
| 离线资源解耦网络 | mock fetch/XHR + axios | `add_resource` / `netLog` + curl_cffi 真发包 |
| 反调试 | AST 去 `debugger` | 内置禁用 `debugger;`，用 `vdebugger;` |

iv8 特有优势：浏览器 API 在 C++ 层而非 JS 层 proxy 补丁，常见原型链/属性描述符/native 外观可由 README/demo 支撑，省去 Node 路线大量手工补丁。代价：社区版无真实网络栈（需 curl_cffi 自接），Worker/ServiceWorker 实际可用深度仍需实测。

## 相关实体与概念

- [[entities/iv8]] —— reverse-skill 补环境落地引擎（Python 原生 V8），Node 默认路线的折算目标
- [[entities/js-reverse-mcp]] —— reverse-skill 的运行时依赖，真实 Chrome CDP 插桩侦察层
- [[topics/ai-js-reverse-workflow]] —— "MCP 侦察 → iv8 补环境出参 → curl_cffi 回灌"的完整六阶段 SOP
- [[concepts/js-reverse-supplement-env]] —— 补环境元方法（缺啥补啥探针、toString 一致性、跨字段一致性五条铁律）

## 来源

- `raw/github/HanZzzzz000iv8.md` —— iv8 一手仓库素材（补环境引擎）
- `raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md` —— iv8 中文介绍源
- js-reverse-mcp 仓库（**已核实**）：`gh api repos/zhizhuodemao/js-reverse-mcp` + `https://github.com/zhizhuodemao/js-reverse-mcp` README + `https://registry.npmjs.org/js-reverse-mcp`
- iv8 examples 目录（**已核实**）：`https://github.com/HanZzzzz000/iv8/tree/main/examples`（海关/税务/欧冶/abogus/h5st/tdc/zp_stoken/药监局）
- iv8 包与许可（**已核实**）：`https://pypi.org/pypi/iv8/json` + LICENSE
- reverse-skill 仓库本体：`README.md` 与 `jsr-reverse/SKILL.md` 一手实读（未落 raw 摘录）
