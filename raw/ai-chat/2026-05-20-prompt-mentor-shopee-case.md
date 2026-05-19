# 用户提示词导师案例：seller-shopee-spider 自动登录采集流程

> 用户提供的真实工作流原型，用于沉淀「提示词导师 Agent」方法论。

## 用户的日常工作流

平常使用以下提示词，在对应需要的项目中。一般流程：
- 启动两个 Claude Code CLI 终端
- 一个 CLI 用于「生成提示词」（导师角色）
- 另一个 CLI 用于「执行任务」（执行角色，干净上下文）

## 原始提示词（生成端输入）

```
Goal:
d:\SpiderCode\dev\livelab-crawler\seller-shopee-spider 实现自动登录、采集、上传kafka的流程
实现的数据接口文档：seller-lazada-spider\docs\LAZADA 数据 Kafka 对接数仓文档.md

Context and Workflow:
主要流程 按照 seller-shopee-spider 执行
请求库使用seller-shopee-spider\utils\simple_downloader.py
登录使用playwright

上传kafak的`总数据结构`参考：D:\SpiderCode\VAT\patrick_star\livelab\live_dp\crawlers\http 下的 lazda 实现
自动登录有关参考：D:\SpiderCode\VAT\patrick_star\livelab\live_dp\cookie_keeper\browser_refresher.py 的 _login_sellercenter

Constraint:
注意需要考虑多国家，不同国家域名不同而已
```

## 用户希望解决的问题

- 这版提示词依赖手工组合，缺少 Claude Code 子代理 / skill / MCP 的调度策略
- 没有显式的「需求澄清」阶段，AI 容易直接动手而忽略未明问题
- 没有显式的「验证 / 自检」阶段，AI 写完不验证
- 没有给执行端 Agent 准备充足的上下文交付物（设计文档、任务清单、验收标准）

## 收录目的

作为「提示词导师 Agent」wiki 主题的真实素材，提炼出：
1. 完整的 prompt 生成流程（澄清 → 调研 → 拆解 → 输出）
2. 该使用哪些 skill / subagent / MCP（systematic-debugging、subagent-driven-development、writing-plans、context7、claude-mem 等）
3. 可直接复用的 prompt 模板（导师端启动语 + 执行端交付包）
