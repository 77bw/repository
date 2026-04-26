# GitHub 功能术语速查

## 核心功能

| 功能 | 说明 | 使用场景 |
| --- | --- | --- |
| Issues | 任务/Bug 追踪，支持标签、checklist、自动关闭 | 记录待办、Bug、功能需求 |
| Tags | 给某个 commit 打版本标记 | v1.0.0、v2.1.3 |
| Releases | 基于 Tag 的正式发布页，可附带下载包和更新说明 | 项目版本发布 |
| Actions | 内置 CI/CD，推送代码后自动执行脚本 | 自动跑测试、自动部署 |
| Projects | 内置看板工具（类似 Trello），拖拽管理 Issues | 可视化任务管理 |
| Wiki | 仓库自带多页文档站，有侧边栏导航 | API 文档、使用手册 |
| Discussions | 仓库自带论坛/问答区 | 开放式讨论、社区问答 |
| Pages | 免费静态网站托管，从仓库直接发布 | 项目官网、博客、文档站 |

## 优先级建议

| 现在用 | 以后考虑 | 个人项目不需要 |
| --- | --- | --- |
| Issues | Tags / Releases（版本管理时） | Discussions |
|  | Actions（加自动测试时） |  |
|  | Projects（Issues 多了后） |  |
|  | Wiki（文档多了后） |  |
|  | Pages（做开源文档站时） |  |