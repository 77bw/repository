# OpenCode + Oh My OpenCode 完整使用指南

# lOpenCode + Oh My OpenCode 完整使用指南

> 本文档记录了 OpenCode 及其增强插件 Oh My OpenCode (OMO) 的安装配置和使用技巧。
> 

---

# 一、什么是 OpenCode？

**OpenCode** 是一个开源的 AI 编程助手，提供终端界面、桌面应用和 IDE 扩展。

- 官网：[https://opencode.ai](https://opencode.ai)
- GitHub：60K+ Stars，650,000+ 开发者使用

---

# 二、什么是 Oh My OpenCode (OMO)？

**Oh My OpenCode** 是 OpenCode 的"全能增强包"插件，提供：

- **多模型编排**：智能调度 Claude、OpenAI、Gemini 等不同模型
- **专业 Agent 矩阵**：Sisyphus、Oracle、Librarian、Prometheus 等
- **工作流自动化**：20+ 内置 Hooks 和自循环开发模式
- **深度代码理解**：集成 AST 搜索和 LSP 分析工具

---

# 三、安装步骤

## 3.1 安装 OpenCode

```
curl -fsSL https://opencode.ai/install | bash
```

## 3.2 安装 Oh My OpenCode

```
bunx oh-my-opencode install
```

---

# 四、配置文件位置

- OpenCode: ~/.config/opencode/opencode.json
- Oh My OpenCode: ~/.config/opencode/oh-my-opencode.json

---

# 五、模型配置重要说明

## 5.1 关于模型兼容性

**重要提醒**：OpenCode 的模型配置需要与你使用的 API 提供商格式兼容！

- **不能直接复制** Claude Desktop、Codex CLI 等工具的配置
- 不同工具的配置格式不同，直接复制会导致无法使用

## 5.2 我的配置说明

我的配置使用的是 **Antigravity Tools 反代格式**：

- 本地反代地址：[http://127.0.0.1:8045/v1](http://127.0.0.1:8045/v1)
- 使用 @ai-sdk/anthropic npm 包作为 SDK
- 支持 Claude、Gemini 等多种模型的统一接口

---

# 六、OMO Agent 系统

## 6.1 核心 Agent 列表

- **Sisyphus** - 主编排器：任务分析、拆解和委派执行
- **Oracle** - 神谕：高难度调试、复杂架构设计咨询（只读）
- **Librarian** - 馆长：查找外部文档、库用法、开源实现
- **Explore** - 探索者：代码库上下文搜索
- **Prometheus** - 规划者：在编码前制定详尽技术方案
- **Metis** - 预规划顾问：分析请求，识别隐藏意图和歧义
- **Momus** - 评审专家：评估工作计划的完整性和可行性

## 6.2 任务委派分类

- visual-engineering: 前端、UI/UX、样式、动画
- ultrabrain: 深度逻辑推理、复杂架构决策
- quick: 简单任务、单文件修改
- writing: 文档、技术写作

---

# 七、常用斜杠命令

- / - 触发自动补全，显示所有可用命令
- /ralph-loop - 启动自循环开发模式
- /ulw-loop - 启动极速工作模式
- /refactor - 智能重构
- /playwright - 浏览器自动化
- /init-deep - 初始化项目知识库
- /git-master - Git 操作专家模式

---

# 八、Skills 技能系统

- playwright - 浏览器自动化
- git-master - Git 操作专家
- frontend-ui-ux - 前端 UI/UX 设计开发
- dev-browser - 持久化浏览器自动化

---

# 九、OCCM 配置管理器

OCCM 是可视化配置管理工具

GitHub: [https://github.com/icysaintdx/OpenCode-Config-Manager](https://github.com/icysaintdx/OpenCode-Config-Manager)

## WSL 安装

```
wget https://github.com/icysaintdx/OpenCode-Config-Manager/releases/latest/download/OpenCode-Config-Manager-Linux-x64.tar.gz
tar -xzvf OpenCode-Config-Manager-Linux-x64.tar.gz
./OCCM_v1.7.1/OCCM_v1.7.1
```

## WSL 中文乱码解决

```
sudo apt install fonts-wqy-zenhei fonts-wqy-microhei fonts-noto-cjk
fc-cache -fv
```

---

# 十、使用技巧

1. 启动：在项目目录运行 opencode
2. 初始化：首次使用运行 /init-deep
3. 简单任务：直接描述，Sisyphus 自动处理
4. 复杂任务：使用 /ralph-loop 持续工作
5. 架构咨询：明确说"咨询 Oracle"

---

文档创建时间：2026-01-29

[opencode.json 配置文件](OpenCode%20+%20Oh%20My%20OpenCode%20%E5%AE%8C%E6%95%B4%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/opencode%20json%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%202f781caa0df881888b2de0d8840ac1ac.md)

[oh-my-opencode.json 配置文件](OpenCode%20+%20Oh%20My%20OpenCode%20%E5%AE%8C%E6%95%B4%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/oh-my-opencode%20json%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%202f781caa0df88158a0f5d752253c758a.md)

[使用小tips](OpenCode%20+%20Oh%20My%20OpenCode%20%E5%AE%8C%E6%95%B4%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/%E4%BD%BF%E7%94%A8%E5%B0%8Ftips%202f581caa0df88050ac20db637bfb912c.md)