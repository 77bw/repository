---
title: JS 逆向通用技巧手册
type: topic
created: 2026-06-02
updated: 2026-06-02
tags: [spider, methodology, sop, mature]
sources:
  - https://github.com/715494637/reverse-skill/tree/main/zh/jsr-reverse/references
  - https://github.com/zhizhuodemao/ai-reverse-toolkit
summary: JSVMP、AST 反混淆、边界选择与最小恢复的通用逆向技巧
confidence: medium
---

# JS 逆向通用技巧手册（JSVMP / AST / WASM / Worker / Webpack）

## 概述（背景与问题）

补环境只是 JS 逆向的"运行"环节；在跑通之前，往往要先穿透层层**壳层保护**：JSVMP 虚拟机、AST 混淆、控制流平坦化、webpack 打包、worker/wasm 桥接。这些技巧与具体运行时引擎无关——无论最终用 [[entities/iv8]] 还是 Node 落地，识别与恢复壳层的方法是同一套。

本页把 [[entities/reverse-skill]] 的 17 篇方法论与 [[entities/ai-reverse-toolkit]] 的 AST 剧本提炼成**可操作步骤**，回答"怎么识别 → 怎么定位 → 怎么恢复"，而非仅点名技术。

贯穿全篇的一条核心纪律：**边界优先、证据驱动、最小恢复**。不为了"看起来干净"而过度还原；只恢复到"能解释目标字段、能支撑下游回放"为止。

---

## 分析（多角度展开）

### 〇、统一心智：恢复深度三级模型（A/B/C）

所有壳层恢复共用一套深度决策。**默认从最浅开始，只有证据证明当前级别撑不住下游，才升级**：

| 级别 | 做什么 | 适用条件 |
|------|--------|----------|
| **Level A** | 只提取触达目标字段的关键 opcode/算子 | 写回边界已明确，少量语义即可解释目标 |
| **Level B** | 恢复 dispatcher + 关键状态载体 | 不恢复 dispatcher 就无法理解 opcode；关键分支依赖寄存器/栈/上下文 |
| **Level C** | 最小反编译 / 最小解释器 | 需重放多条路径、协议重建或批量执行；A/B 都撑不住 |

**硬性升级规则**：不能因为"代码脏、平坦化严重、字符串表复杂"就直接跳 C。A→B 必须满足"opcode 不恢复 dispatcher 无法解释"等具体证据；B→C 必须满足"下游要求重放多条路径"等具体证据。

> 这条纪律的价值：逆向最大的时间黑洞是"过度还原"——把整个虚拟机重建出来，却发现目标字段只用到 3 个 opcode。先证明边界，再决定深度。

---

### 一、边界选择（一切的起点）

在动任何工具前，先选对**边界**。工具是次级选择，边界选择才是核心。先问三个问题：

1. 目标值最终**写入**在哪里？
2. 哪一层最接近最终写点？
3. 哪一层最不容易被伪装？

**常见边界模式速查**（来自 `hook-and-boundary-patterns.md`）：

| 场景 | 优先边界 | 不好的起手式 |
|------|----------|--------------|
| 动态请求体字段 | 序列化前的最终对象 | ❌ 先搜 crypto |
| 动态请求头字段 | header 设置点 / writer | ❌ 先搜 header 名 |
| JS 写入 cookie | cookie 写点 | ❌ 只看 `document.cookie` 读取 |
| 响应 Set-Cookie | 网络响应 | ❌ 在前端搜 cookie 名 |
| WebSocket frame | send 前的 envelope 层 | ❌ 只盯一条 payload |
| worker 生成值 | `postMessage` 契约 | ❌ 先钻 worker 内部 |
| 隐藏 DOM 字段 | 赋值点 + submit 动作 | ❌ 先搜字段明文 |

**边界选错的信号**：眼前一堆疑似 crypto 函数但说不清最终写点 / 观察点离真实发网太远 → 退回到更靠近 sink 的边界。

**最小观察顺序**（以请求体/头为例）：`writer → builder → entry → source`，逆着数据流回溯。

---

### 二、JSVMP / 虚拟机保护解析

JSVMP（JS Virtual Machine Protection）把原始逻辑编译成自定义字节码，由一个解释器循环执行——源码里看不到算法，只看到 dispatcher 在跑 opcode。

**怎么识别**：大量 `switch-case` 或查表跳转 + 一个长循环 + 寄存器数组/栈对象/常量池等状态载体。

**四步顺序**（`jsvmp-and-ast.md`）：

1. **找入口** — bytecode 从哪来、dispatcher loop 从哪开始、哪个函数负责解释执行
2. **识别状态载体** — 寄存器数组 / 栈对象 / 上下文对象 / 常量池与字符串表
3. **提取关键 opcode** — 不要一开始就重建整个虚拟机；先回答：哪些 opcode 直接触达目标字段？哪些负责 hash/加密/序列化/组包？哪些只是搬运状态（可忽略）？
4. **等价性检查** — 每提取一段切片就做输入/输出对比，不凭"看起来像"下判断

**三张必备工件卡**（用于支撑结论、跨会话交接）：
- **入口卡**：VM 入口函数/闭包、bytecode 加载器、dispatcher 入口位置、到目标字段的调用链
- **状态载体卡**：载体类型、初始化位置、与目标相关的读写点、哪些实质影响目标 vs 哪些只搬运
- **关键 opcode 卡**：输入/输出/状态修改/依赖、与目标字段的关系、等价性结论（match/diverge/unproven）

**常见误判**：把 dispatcher 恢复本身当完成；把字符串表恢复当算法恢复完成；剩余阻塞明明是 runtime 分歧却继续深挖 VM。

**iv8 落地优势**：VMP 的依赖数组是补环境天花板（[[entities/ai-reverse-toolkit]] env-patch 铁律 2）。但 iv8 常可直接 `page.load` 整个 bundle 让 VM 在真实 V8 里跑，省去手工重建解释器——把"恢复"问题转化为"补环境让它自己跑"问题。这是 iv8 路线相对纯算法还原的关键省力点。

---

### 三、AST 反混淆实操

当主导壳层是 AST 混淆（字符串表、helper 包装、控制流平坦化、bundle 外壳），按剧本走（`ast-deobfuscation-playbook.md`）。

**第一步永远是指纹判断**——先识别主导壳层，没判断清楚前不做批量变换：

| 信号 | 壳层类型 | 首动作 |
|------|----------|--------|
| 大字符串表 + 访问器 helper | 字符串表混淆 | 先恢复字面量 |
| 大量小 helper 包装操作符 | helper proxy / 对象字典 | 字面量稳定后内联 |
| `while` + `switch` 调度 | 控制流平坦化 | 常量稳定后恢复执行顺序 |
| 模块启动器包裹全量逻辑 | bundle 外壳 | 先拆模块 |

**推荐变换顺序**（除非证据证明依赖不同）：

1. 恢复可读锚点：字面量、对象 key、调用关系
2. 外壳阻断局部推理时，先拆 bundle
3. 内联简单 helper 和对象字典
4. 标准化属性访问和简单表达式
5. 恢复真实执行顺序
6. **只有活路径被证明后**，才删死代码或诱饵分支

**变换台账**（每步必记）：输入产物 / 输出产物 / 执行的变换 / **保持什么不变** / 验证证据。
> 铁律：如果一条变换说不清它"保持什么不变"，就不该执行它。

**控制流平坦化恢复**（`jsvmp-and-ast.md` AST 段）：
1. 先恢复可读锚点（字面量/字符串表/对象 key/调用关系）
2. 识别平坦化 dispatcher——顶层 `switch` + 跳转状态变量 + 死分支/诱饵分支
3. 恢复真实执行顺序——判断依据是**执行顺序**，不是 beautify 后的源码顺序
4. 分离副作用——分清"只是控制流包装"vs"真正修改状态/请求数据/返回值"的分支

**常见误判**：只做 beautify 就宣布完成；拆 bundle/内联 helper/还原控制流混在一起做却没台账；活路径没证明就先删死代码；把"源码变好看了"当证据。

**工具链**：传统走 `@babel/parser` + `@babel/traverse` 写 visitor 替换节点（Node 特定）。**iv8 等价**：iv8 本身不做 AST 变换（它是运行时不是静态分析器），AST 反混淆这一步仍在 Node/babel 侧完成，产出可读 JS 后再交给 iv8 `eval`/`page.load` 运行。这是少数"iv8 不替代 Node"的环节——静态去混淆与动态补环境是正交的两件事。

---

> **运行时桥接、反调试与纯算法迁移前检查** 已拆到 [[topics/js-reverse-runtime-bridge-techniques]]，本页聚焦边界选择、JSVMP 与 AST 反混淆。

## 结论

这套技巧的本质不是"会用某个工具"，而是一套**纪律**：

1. **边界优先**——先证明目标值写在哪、最接近写点的层在哪，再选工具
2. **最小恢复（A/B/C）**——默认最浅，有证据才加深，拒绝过度还原
3. **证据驱动**——每步有"保持什么不变 + 验证证据"，等价性结论写明 match/diverge/unproven
4. **桥接契约**——WASM/Worker/Webpack 优先黑盒复用边界，不轻易全量反编译
5. **分叉前置**——跑不通先分清调试摩擦 vs 风控分支，排除六类状态依赖再宣称纯算法

**与 iv8 的协作分工**：静态去混淆（AST/控制流还原）仍在 Node/babel 侧；动态运行可交给 iv8 `page.load`/`eval` 黑盒复用。运行时桥接细节见 [[topics/js-reverse-runtime-bridge-techniques]]。

---

## 来源

- [[entities/reverse-skill]] zh/jsr-reverse/references（2026-06 实读）：
  - `jsvmp-and-ast.md` — JSVMP 三级恢复 + 工件卡 + AST/控制流顺序
  - `ast-deobfuscation-playbook.md` — AST 去混淆指纹判断 + 变换顺序 + 台账
  - `wasm-worker-webpack.md` — 三类桥接层的 Bridge-contract 卡片
  - `recover-strategy.md` — 恢复目标/级别/黑盒复用边界/停止标准
  - `hook-and-boundary-patterns.md` — 边界选择模式速查 + 何时 hook/下断点
  - `anti-debug-and-risk-branches.md` — 调试摩擦 vs 风控分支 + 分叉图
  - `minimal-env-design.md` — 最小运行时设计纪律
- [[entities/ai-reverse-toolkit]] skills/ast-deobfuscate — AST 解混淆 skill
- [[entities/iv8]] — 动态运行落地引擎，与静态去混淆正交协作