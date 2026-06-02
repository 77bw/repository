---
title: iv8 反爬对抗实战案例库
type: topic
created: 2026-06-02
updated: 2026-06-02
tags: [spider, case-study, tooling, mature]
sources:
  - raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md
  - https://github.com/HanZzzzz000/iv8/tree/main/examples
summary: 抖音、京东、腾讯防水墙、BOSS、瑞数 examples 与 Akamai 迁移参考
confidence: high
---

# iv8 反爬对抗实战案例库

> 配套 [[topics/iv8-supplement-env-handbook]]（API 速查 + 补环境方法论）。本页是 7 个真实站点的 iv8 逆向案例 + 跨案例 SOP + 「风控查 X→iv8 配 Y」映射表，基于官方 `examples/` 一手代码与微信文章。

## 概述

iv8 逆向的核心闭环是「跑 JS 出参 → curl_cffi 回灌」。本页把官方 8 个 `examples/` 与微信 5 步 Akamai 抽象为可迁移模式：先按风控类型对号入座（见各案例），再套用 [[topics/ai-js-reverse-workflow]] 的 6 阶段 SOP。抖音/京东/腾讯防水墙/BOSS/瑞数等来自 GitHub examples 一手代码；Akamai 段来自微信文章，作为迁移参考而非官方 example 实证。

## 分析


> **站点归类纠错（任务标签有误，以实际代码为准）**：`tdc.py` 实为**腾讯防水墙/天御验证码**（turing.captcha.qcloud.com）非瑞数；`zp_stoken.py` 实为 **BOSS 直聘** `__zp_stoken__` 非智联。

#### 3.1 抖音 a_bogus（自定义请求参数签名，非 WAF）

风控类型：bdms SDK 生成挂在 URL query 的 `a_bogus` 签名。`environment` 只配 location + navigator。装载用**直接 eval**（纯 SDK 不依赖 DOM）。补 MessageChannel polyfill。**模式 B**（XHR 触发让 SDK 自 hook 改 URL，从 netLog 收割）：

```python
request_list = ctx.eval(f"""
    var xhr = new XMLHttpRequest();
    xhr.open('GET', "{url}?{urlencode(params, safe='*')}", true);
    xhr.setRequestHeader("Content-Type", 'application/json, text/plain, */*');
    xhr.send(null);
    window.__iv8__.netLog.entries;
""", to_py=True)
response = requests.get(request_list[0]["url"], headers=headers, cookies=cookies)  # 带 ttwid
```

#### 3.2 京东 h5st（直接调签名函数 + curl_cffi 管 TLS）

风控类型：`window.ParamsSignMain` 生成的 h5st 签名。装载用 `innerHTML`（省开销）+ MessageChannel polyfill。**模式 A**（签名入口已定位，直接调）。关键：京东对 TLS 敏感，**必须 curl_cffi 而非 stdlib requests**：

```python
from curl_cffi import requests   # 管 JA3/JA4 TLS 指纹
import hashlib
params['h5st'] = ctx.eval("""
    new window.ParamsSignMain({appId: "2088b"})
        ._$sdnmd({"appid":"jd-cphdeveloper-m","functionId":"recommend_like_m","body":"%s"}).h5st
""" % hashlib.sha256(params['body'].encode()).hexdigest())   # body 的 sha256 在 Python 侧算
response = requests.get(url, headers=headers, params=params, impersonate="chrome")
```

#### 3.3 腾讯防水墙 tdc（iv8 isTrusted 可信输入的杀手锏）

风控类型：滑块验证码，需缺口位置 + POW + 拖拽轨迹 + TDC 环境采集。非 iv8 部分：cv2（Canny+matchTemplate 算缺口）、hashlib MD5 暴力求 POW nonce。iv8 核心是 **`isTrusted=true` 可信拖拽**（TDC 采集会校验事件可信性，纯 JS hook 补不出 isTrusted=true，这是 iv8/原生 V8 框架相对纯 vm 执行的护城河）：

```python
ctx.expose(traj, "traj")   # Python 轨迹数组喂进 JS
ctx.eval("""
const st = window.__iv8__, input = st.input, traj = st.data.traj;
const base = { target: document, pointerId:1, pointerType:'mouse', isPrimary:true, button:0 };
let start = { clientX: 50, clientY: 410 }, lx = start.clientX, ly = start.clientY, lastT = 0;
input.dispatchPointerEvent({...base, clientX:lx, clientY:ly, buttons:1, type:'pointerdown'});
for (const p of traj) {
  const dt = Math.max(0, (p[2]||0) - lastT); if (dt > 0) st.eventLoop.sleep(dt);  // 按时间戳还原时序
  lx = start.clientX + (p[0]||0); ly = start.clientY + (p[1]||0); lastT = p[2]||0;
  input.dispatchPointerEvent({...base, clientX:lx, clientY:ly, buttons:1, type:'pointermove'});
}
input.dispatchPointerEvent({...base, clientX:lx, clientY:ly, buttons:0, type:'pointerup'});
""")
res = ctx.eval("({collect: decodeURIComponent(window.TDC.getData(true)), eks: window.TDC.getInfo()})", to_py=True)
```

> 注：连续拖拽轨迹 API `dragMouse`/`dragPointer` 仅 Pro 版；社区版用 `dispatchPointerEvent` 逐点 pointermove 模拟（如上）。

#### 3.4 BOSS 直聘 __zp_stoken__（响应驱动挑战 + 模式 A）

风控类型：首次 POST 返回 `code=37` + `zpData{seed,name,ts}`，按 `name` 下载 per-request 动态文件名的 security-js。装载用 `page.load`（手拼 `<script src>` + resources 注入外链 JS）。**模式 A** 直接调已定位入口 `window.ABC.z(seed, ts)`：

```python
ctx.expose({"baseURL": challenge_url, "html": html_with_script_tag,
            "headers": [], "resources": {js_url: js_code}}, "snapshot")
ctx.eval("__iv8__.page.load(__iv8__.data.snapshot)")
token = ctx.eval(f"encodeURIComponent((new window.ABC).z({json.dumps(seed)}, {int(ts)}))", to_py=True)
session.cookies.set('__zp_stoken__', token)   # 回灌后重 POST, code==0 为成功
```

#### 3.5 瑞数 rs 两跳路由（海关/税务 canonical 模板，最高复利）

风控类型：瑞数（首请求 `status_code==202`），政府/国企站最常见。海关与税务代码近乎同构 = 标准两跳模板：`load → sleep → 收 cookie → 带 cookie 重 load → XHR 触发 → netLog 收割 signed-url + cookie`：

```python
with iv8.JSContext(environment=environment, config={"timezone":"Asia/Shanghai"}) as ctx:
    # 第 1 跳: 种 rs cookie
    resp1 = requests.get(page_url, headers=headers)
    js_url = urljoin(page_url, re.search(r'src="([^"]+\.js)"[^>]*r=\'m\'', resp1.text).group(1))
    js_code = requests.get(js_url, headers=headers, cookies=resp1.cookies.get_dict()).text
    ctx.expose({"baseURL":page_url, "html":resp1.text,
                "headers": [[k, v] for (k, v) in resp1.raw.headers.items()],
                "resources":{js_url:js_code}}, "s1")
    ctx.eval("window.__iv8__.page.load(window.__iv8__.data.s1)")
    ctx.eval("window.__iv8__.eventLoop.sleep(100)")   # 让 rs 的 setTimeout 链跑完种 cookie
    cookies_str = ctx.eval("window.__iv8__.netLog.entries[window.__iv8__.netLog.entries.length-1].cookieHeader")
    # 第 2 跳: 带 cookie 拿带 XHR-hook 的真实页面 JS, 发 XHR 触发 hook
    resp2 = requests.get(page_url, headers={**headers, "Cookie": cookies_str})
    # ... 同样 page.load(s2) ...
    ctx.eval(f"""var xhr=new XMLHttpRequest();xhr.open('POST','{url}');
        xhr.setRequestHeader('X-Requested-With','XMLHttpRequest');xhr.send('{body_str}');""")
    entry = ctx.eval("window.__iv8__.netLog.entries[window.__iv8__.netLog.entries.length-1]")
    api_url = f"{origin}{entry['url']}" if entry['url'].startswith('/') else entry['url']
    requests.post(api_url, data=body_str, headers={**headers, "Cookie": entry.get('cookieHeader') or cookies_str})
```

单跳变体（欧冶钢铁，rs 对脚本顺序敏感）：用 `innerHTML + 手动分段 eval` 精确控制 inline/外链脚本执行顺序，比 page.load 更可控。极简变体（药监局 NMPA）：签名是**纯 Python MD5**（`&nmpasecret2020` + urllib quote），iv8 只在首请求非 200 时跑页面脚本拿那一个反爬 cookie —— **能 Python 纯算的就别上 V8**（最小化补环境原则）。

#### 3.6 微信文章：通用 5 步 Akamai 绕过

微信文章归纳的 Akamai v2/v3 流程，与 rs 两跳在“先种 cookie / 再跑 JS / 再回传”结构上相似（curl_cffi 种 cookie → 提取脚本 → iv8 跑 → netLog 抓参 → 回传）。注意：该段不是 iv8 官方 examples，经目标站落地前需重新核实脚本路径、cookie 绑定和响应判定。

| 步骤 | 动作 | 关键坑点 |
|------|------|----------|
| 1 | `curl_cffi.Session(impersonate="chrome")` GET 页面种 `_abck`/`ak_bmsc`/`bm_sz` | curl_cffi 负责 TLS 层 JA3/JA4，避免边缘识别 |
| 2 | 正则动态提取 Akamai 脚本路径（`/akam/` bootstrap + `h8u_e` sensor） | **不能硬编码**，Akamai 约每周轮换路径 |
| 3 | **同一 session** 下载脚本（跑 3 轮） | `_abck` 与 sensor_data 绑定校验，换 session 永远过不去 |
| 4 | iv8：`page.load(html+resources)` → `eventLoop.advance(5000)` → 读 `netLog.entries` | 逻辑时间瞬间跑完指纹采集定时器 |
| 5 | 把 netLog 里 `h8u_e` 的 POST body(sensor_data) 用 session 回传 → `_abck` 转已验证 | POST 回去后 `status_code==200` 为绕过成功 |

#### 3.7 跨案例通用模式：6 阶段 SOP

把分散在 8 个文件的手法抽象为可迁移流程（详见 [[topics/ai-js-reverse-workflow]]）：

- **S0 裸请求探风控**：HTTP 客户端裸打看信号定路线 —— 202(瑞数) / code=37(BOSS) / 非 200(药监 cookie) / 验证码 json(腾讯)。
- **S1 配 environment 指纹**：location 必配且全；按需 navigator/config{timezone}/canvas.fingerprint；其余吃 200+ 默认。
- **S2 装载 JS（三选一）**：`page.load`（需脚本生命周期）| `innerHTML + 手动分段 eval`（需控序或只要 DOM）| 直接 `eval`（纯 SDK）。
- **S3 补缺失环境**：`eval` 注入 polyfill + `wrapNative` 伪装。共性补丁：MessageChannel。
- **S4 跑参数（两模式）**：模式 A 直接调签名入口（入口已定位）| 模式 B 发 dummy XHR 让风控自 hook 改 URL，读 `netLog.entries[-1]` 收割。
- **S5 取运行态产物**：`document.cookie` / `netLog entry.cookieHeader` / `TDC.getData()`；用 `eventLoop.sleep|drain` 推进异步。
- **S6 回灌真实 HTTP**：iv8 社区版不发真实请求；token/cookie/signed-url 喂 Python 客户端。TLS 敏感站(京东/Akamai) 用 `curl_cffi`，普通站 `requests`。

#### 3.8 「风控查 X → iv8 配 Y」映射表 `[通用思维]`

| 风控探测点 | iv8 对策 |
|-----------|----------|
| `navigator.webdriver` | `environment.navigator.webdriver=False`（默认已 False） |
| `navigator.plugins/mimeTypes` 长度+原型 | 补结构正确的非空 PluginArray（补类型+原型，非只补 length） |
| `window.chrome.loadTimes` | `environment.chrome.loadTimes.*`（对象确实存在） |
| UA vs platform 矛盾 | userAgent / platform / userAgentData 三者一致（Win64 ↔ Win32 ↔ Windows） |
| Canvas `toDataURL` 哈希 | `environment.canvas.fingerprint.toDataURL.*` 配稳定哈希；`config.canvas.*` 控行为 |
| WebGL vendor/renderer | `environment.webgl.UNMASKED_*`（字段可配；真机 GPU 串与 Canvas/WebGL 哈希仍需目标站验证） |
| `Function.prototype.toString` `[native code]` | 所有 hook 函数用 `wrapNative` 伪装 |
| 无限 `debugger` 反调试 | iv8 已禁用原生 debugger，用 `vdebugger;` |
| console 行为差异检测 | `enable_console=False` + `vconsole` |
| POW / 时间差校验 | `time_mode='system'` 或 `eventLoop.setDateAdvanceStep` |
| 滑块手势可信性 | `input.dispatchPointerEvent` 派 `isTrusted=true` |
| TLS JA3/JA4（边缘层） | **不归 iv8** → `curl_cffi(impersonate='chrome')` |
| 鼠标轨迹/键盘 timing（行为层） | iv8 覆盖不到 → 自造合理轨迹序列或改真实浏览器 |
| `cdc_*`/`_selenium`（驱动注入物） | iv8 天然没有 → 确认**别误补出来** |

## 结论

这些材料归纳出 iv8 逆向的可迁移模板：**瑞数两跳**（海关/税务 canonical）、**响应驱动挑战**（BOSS）、**可信输入**（腾讯防水墙 isTrusted）、**直调签名**（京东 h5st 模式 A）、**自 hook 收割**（抖音 a_bogus 模式 B），以及来自微信文章的 **Akamai 5 步迁移参考**。核心心法见 [[topics/iv8-supplement-env-handbook]]：environment(画像)/config(行为) 二分 → 三选一装载 → wrapNative 补缺 → 两模式跑参 → curl_cffi 回灌。最小化原则：能 Python 纯算的（药监局 MD5）就别上 V8。

## 来源

- 官方 `examples/`（abogus / h5st / tdc / zp_stoken / 海关 / 税务 / 欧冶 / 药监局）—— 真实案例代码（2026-06 GitHub 实读，含 `cookieHeader` 等字段经海关.py/税务.py 验证）
- `^[raw/wechat/Python V8 原生扩展，重新定义 JS 逆向的玩法.md]` —— 5 步 Akamai 绕过迁移参考（非官方 examples）
- 相关页：[[topics/iv8-supplement-env-handbook]] · [[topics/ai-js-reverse-workflow]] · [[entities/iv8]] · [[concepts/js-reverse-supplement-env]]
