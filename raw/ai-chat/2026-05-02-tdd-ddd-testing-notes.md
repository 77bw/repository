# TDD / DDD / 三种测试 — 数据采集工程师视角

日期: 2026-05-02

---

## 背景：为什么会聊到这个话题

原始需求是用架构师 Skill 来优化项目代码架构。但在讨论过程中发现，真正的瓶颈不是"架构模式"——而是作为一个数据采集工程师，日常面对的项目有几个特点：

1. **I/O 密集**：70%+ 的代码在调 FFmpeg、发 HTTP、解析 HTML、读写文件
2. **外部依赖不稳定**：直播平台接口随时变，FLV 流地址会过期
3. **测试需要真实数据**：你没法 mock 一个"假的抖音直播间"

而 TDD/DDD 这些主流方法论，最初是为"业务逻辑复杂、团队协作多"的场景设计的。直接硬套到数据采集项目上会觉得格格不入——**这个感觉是对的，不是你的问题**。

所以真正要回答的问题是：**从这些方法论里取什么、扔什么？什么替代方案更适合数据采集？**

---

## 一、TDD（测试驱动开发）

### 是什么

**核心循环：红 → 绿 → 重构**

```
1. 先写一个失败的测试（红）      → 定义"这段代码应该做什么"
2. 写最少代码让测试通过（绿）    → 实现功能
3. 整理代码结构（重构）          → 不改变行为，只改善设计
```

### 为什么这么干

代码写完之后再补测试，你会下意识地"按代码的行为写测试"，而不是"按需求写测试"。这样测试只能证明"代码做了它做的事"，不能证明"代码做了它该做的事"。

**先写测试的本质是在写代码之前，逼自己把"什么叫正确"定义清楚。**

### 例子

```python
# 先写测试 —— 此时函数还不存在，但你已经在定义"什么是正确"
def test_parse_live_room_id():
    """从抖音分享链接中提取直播间ID"""
    assert parse_live_room_id("https://live.douyin.com/123456789") == "123456789"

# 再写最简单实现
def parse_live_room_id(url):
    return url.split("/")[-1]

# 补充边界 —— 这就是"红→绿→重构"在迭代
def test_parse_live_room_id_with_params():
    """带查询参数的链接"""
    assert parse_live_room_id("https://live.douyin.com/123456789?enter_from=feed") == "123456789"

# 发现第一版挂掉 → 修正
def parse_live_room_id(url):
    return url.split("/")[-1].split("?")[0]
```

### 在数据采集项目中的适用边界

TDD 的隐藏前提是：**你能精确控制输入、精确断言输出。** 一旦输入输出不可控，TDD 循环就转不动了。

| 能精确控制输入输出 | 不能 |
|-------------------|------|
| 字符串解析、文件名生成 | FFmpeg 调用（网络流时长不可控） |
| 时间戳计算、去重逻辑 | HTTP 请求（远程返回随时变） |
| JSON 字段提取、数据校验 | HTML 解析（平台改 DOM 结构） |

**结论**：纯逻辑函数用 TDD 很舒服，但你的项目里纯逻辑可能只占 20%。另外 80% 需要换策略。

---

## 二、DDD（领域驱动设计）

### 是什么

**一句话：用业务语言组织代码，而不是用技术语言。**

### 为什么这么干

假设你三个月后回来看这段代码修一个 bug，或者来了一个新同事。他看到：

```
controllers/live_controller.py
services/capture_service.py
models/live_room.py
utils/ffmpeg_util.py
```

他需要先理解 MVC 分层约定，然后猜"直播相关的逻辑分布在四个文件夹里"。

但如果他看到：

```
直播间/开播检测.py
直播间/直播间.py
采集/流拉取.py
采集/流切割.py
```

他直接就能定位到要改的地方，因为**文件夹名字跟你开会时嘴里说的词是一样的**。

### 传统分层 vs 业务模块

```
❌ 传统（按技术层分）:        ✅ DDD 风格（按业务分）:
controllers/                   直播间/
services/                       ├── 开播检测.py
models/                         ├── 直播间.py
utils/                          └── 直播间仓库.py
                               采集/
                                ├── 流拉取.py
                                ├── 流切割.py
                                └── 下载任务.py
                               交付/
                                ├── 发送到COS.py
                                └── 发送到Kafka.py
                               查询/
                                └── 历史记录.py
```

### 核心概念（只挑对数据采集有用的）

| 概念 | 大白话 | 数据采集场景 |
|------|--------|-------------|
| 实体 | 有唯一 ID、状态会变的东西 | 直播间（ID 不变，"未开播"→"开播"） |
| 值对象 | 没 ID、不可变的东西 | 视频切片 MP4（时长/大小/格式固定） |
| 领域事件 | 发生了什么事 | "主播 123 开播了""切片文件已生成" |
| 领域服务 | 不属于某个对象的流程 | "从 FLV 流切成 5 分钟 MP4 片段" |

### 为什么不全盘照搬

DDD 的完整版（聚合根、仓储模式、限界上下文映射……）是为**复杂业务规则**设计的——比如"一笔订单在不同状态下能做什么操作"这种 if-else 爆炸的场景。

数据采集的"业务规则"其实很薄：检测到开播 → 拉流 → 切割 → 发送。**过度建模反而拖慢速度**——一个 200 行脚本能解决的问题，拆成 6 个文件 8 个类就是过度设计。

**取你需要的**：文件夹按业务模块命名 + 用领域事件解耦模块（切割完成 → 发事件 → 上传模块监听）。其他的先不管。

---

## 三、三种测试（回应关键质疑）

### 用户质疑："外部依赖为什么不能做测试？我觉得可以做测试。"

**这个质疑完全正确。** 外部依赖完全可以测试，也应该测试。上一轮我表述有歧义——问题不是"能不能测"，而是"适不适合用 TDD 的循环来测"。

下面拆开讲三种测试的区别，以及为什么每种测试在数据采集项目里扮演不同的角色。

---

### 单元测试 = 测一个齿轮

把齿轮从发动机里拆出来，拿卡尺量。不启动发动机，不给它接油管。

```python
def test_generate_segment_filename():
    """文件名是否正确生成 —— 纯逻辑，不依赖任何外部"""
    result = generate_segment_filename(
        room_id="123456", start_time=900, end_time=1200
    )
    assert result == "123456_000900_001200.mp4"
```

| 特征 | 说明 |
|------|------|
| 速度 | < 1ms |
| 依赖 | 只依赖代码，不联网、不读写文件、不调 FFmpeg |
| 挂了说明 | 这段代码逻辑写错了，定位极快 |
| 适合 TDD？ | ✅ 非常适合。输入输出完全可控 |

---

### 集成测试 = 测两个齿轮的啮合

把"拉流"齿轮和"切割"齿轮装到一起，通上气，看传力顺不顺。**这里开始调 FFmpeg 了，所以不是 TDD 的主场，但它有它不可替代的价值。**

```python
def test_segmenter_can_read_captured_file(tmp_path):
    """切割模块能正确读取拉流模块的输出 —— 测的是接口匹配"""
    # 用 FFmpeg 生成一段 10 秒测试视频
    captured_file = tmp_path / "raw" / "room_001.flv"
    subprocess.run([
        "ffmpeg", "-f", "lavfi", "-i", "testsrc=duration=10:size=320x240",
        "-f", "flv", str(captured_file)
    ], check=True)

    segments = segmenter.segment(str(captured_file), segment_seconds=5)
    assert len(segments) == 2
    assert all(s.exists() for s in segments)
```

| 特征 | 说明 |
|------|------|
| 速度 | 1-30s |
| 依赖 | 调了真实 FFmpeg，读了真实文件 |
| 挂了说明 | 两个模块的接口对不上——拉流输出的是 `.flv`，切割期望的是 `.mp4`；文件没写完切割就开始读了；元数据字段名不一致 |
| 为什么不能简单地用 TDD | 输入是一段视频文件（你没法"先 mock 一个完美的文件再写断言"，文件本身就是测试的一部分），但**这个测试必须写** |

**集成测试抓的是"你以为能对接，实际对不上"的问题**——这恰好是数据采集项目最高频的 bug 类型。

---

### 端到端测试 = 发动整车，上路跑一圈

监控 → 拉流 → 切割 → 上传 → 查询，全部连起来，用本地 RTMP 模拟推流，看全链路有没有断。

```python
def test_full_pipeline_with_local_rtmp():
    """起本地 RTMP → 推测试视频 → 跑全管线 → 验证最终文件可播放"""
    # 1. 起本地 RTMP
    rtmp_server = start_rtmp_server("rtmp://localhost:1935/live/test")
    # 2. 推一段 30 秒测试视频
    push_test_stream(rtmp_server.url, duration=30)
    # 3. 启动监控 + 采集
    monitor = LiveMonitor(platform="test", rtmp_url=rtmp_server.url)
    monitor.start()
    wait_for_pipeline(seconds=40)
    # 4. 验证：有文件，能播放
    output_files = list(Path("./output").glob("*.mp4"))
    assert len(output_files) >= 1
    for f in output_files:
        assert is_valid_mp4(f)
```

| 特征 | 说明 |
|------|------|
| 速度 | > 30s，可能几分钟 |
| 依赖 | 需要本地 RTMP 服务、真实 FFmpeg、完整管线 |
| 挂了说明 | 系统级假设不成立——FFmpeg 拉流过程中崩了、切割出的文件是空壳、上传后文件不完整 |
| 为什么不能 TDD | 你没法"先写一个端到端测试再实现整个管线"——管线的每一段都依赖外部程序，只能先搭起来再验 |

**端到端测试验证的是你代码之外的假设**："抖音的 FLV 流在开播 3 秒内可拉""切割后的 MP4 是真的可播放的""上传 COS 的过程中不会半截中断"。这些东西不是你的代码逻辑，是你的**环境假设**。

---

## 四、为什么你的不适感是合理的

### TDD 的三个隐藏前提，数据采集项目一个都不满足

| TDD 前提 | 数据采集的现实 |
|----------|---------------|
| 输入可精确控制 | FLV 流地址会过期、页面 HTML 会变 |
| 输出可精确断言 | 录 10 秒有时录到 9.8 秒有时 10.2 秒 |
| 失败原因唯一 | 测试挂了是代码 bug？网络？反爬？FFmpeg 版本？ |

**这就是外部依赖"能做测试但不适合 TDD 循环"的根本原因。** 不是方法论错了，是场景不匹配。

### 同样的，DDD 的完整版也不匹配

数据采集的"业务领域"天然是薄的——你做的事情是**搬运和转换**，不是**决策和规则判断**。DDD 解决的是后者的复杂度。

---

## 五、务实的做法：取需要的，扔不合适的

### 从 TDD 取什么

```
✅ 纯逻辑函数写单元测试（URL 解析 / 文件名生成 / 时间计算 / 去重 / 数据校验）
❌ FFmpeg / HTTP / HTML 解析不写单元测试，换集成测试
✅ 但集成测试不遵循 TDD 循环，先写代码再写测试
```

### 从 DDD 取什么

```
✅ 文件夹按业务模块组织（直播间 / 采集 / 交付 / 查询）
✅ 核心实体用业务语言命名（"直播间.py"而非"live_room_entity.py"）
✅ 用领域事件解耦模块（切割完成 → 事件 → 上传模块监听）
❌ 不引入 Aggregate / Repository / Bounded Context 等重概念
```

### 推荐的项目结构

```
live-monitor/
├── 直播间监控/
│   ├── monitor.py           # 轮询 / WebSocket 检测开播
│   ├── room_resolver.py     # URL/ID 解析（← 有单元测试）
│   └── platform/
│       ├── douyin.py
│       └── bilibili.py
├── 采集/
│   ├── stream_capture.py    # FFmpeg 拉流（← 集成测试覆盖）
│   ├── segmenter.py         # 切割逻辑（← 文件名生成有单元测试）
│   └── download_task.py
├── 交付/
│   ├── cos_uploader.py
│   └── kafka_sender.py
├── shared/
│   ├── filename_utils.py    # ← 纯逻辑，单元测试
│   └── time_utils.py        # ← 纯逻辑，单元测试
└── tests/
    ├── unit/
    │   ├── test_filename_utils.py
    │   └── test_room_resolver.py
    └── integration/
        ├── test_segmenter_pipeline.py   # 拉流→切割 对接
        └── test_full_pipeline.py        # 端到端
```

### 更重要的方法论（真正适合数据采集的）

```
防御性编程     → 每个外部调用都假设它会失败：超时、重试、降级
可观测性       → 出问题时你能快速定位：日志、指标、链路追踪
快速故障恢复   → 不是避免出错，而是出错了能自愈：断线重连、僵尸进程清理
```

---

## 六、Skills 生态系统备忘

```bash
# 搜索技能
npx skills find <keyword>

# 安装
npx skills add <owner/repo@skill>

# 安装位置
~/.claude/skills/<skill-name>/      # 全局
.claude/skills/<skill-name>/        # 项目级
```

架构相关：
| 技能 | 安装量 |
|------|--------|
| mattpocock/skills@improve-codebase-architecture | 31.8K |
| wshobson/agents@architecture-patterns | 14.2K |
| wshobson/agents@architecture-decision-records | 7.5K |
