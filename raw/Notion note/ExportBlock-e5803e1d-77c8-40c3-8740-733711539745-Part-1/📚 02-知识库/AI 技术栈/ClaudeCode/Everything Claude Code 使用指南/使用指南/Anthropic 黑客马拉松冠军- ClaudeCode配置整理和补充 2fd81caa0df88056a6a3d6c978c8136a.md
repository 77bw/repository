# Anthropic 黑客马拉松冠军- ClaudeCode配置整理和补充

前几天看了一篇非常好的文章，趁着上头，我就花了几天细细阅读整理了一下，黑客马拉松冠军的ClaudeCode的一些设计我觉得真的非常好，很灵活，它里面设计都不是强制性的流程化，只提供必要的条件，我觉得这是一篇很棒的配置设计思路，像ClaudeCode这类AI编码工具，在使用上是很自由的，因为创意性软件的开发它是自由的，我觉得在自由的东西上面加一层束缚的流程化，大大的削弱了工具的能力，不能完全发挥出来模型的自主性，我借助这套设计思路，是可以搭建某些任务的固定流程、也可以发挥工具的自主性来构建创意性的项目

本文节选自我正在整理的 「上下文工程实践」 项目，该项目已完整发布在 GitHub 上。

如果你希望阅读更多相关的章节与案例，可以前往项目仓库查看：

> 
> 
> 
> [GitHub - WakeUp-Jin/Practical-Guide-to-Context-Engineering: 介绍和描述上下文工程的仓库，上下文工程的定义：是在有限的上下文窗口中，选择、组织并注入与用户输入或任务高度相关的信息，从而让大语言模型（LLM）能够在合理的边界内做出最佳推理和执行。 上下文工程中最关键的是：用最相关的信息填充 LLM 的上下文窗口 - 在线阅读](https://github.com/WakeUp-Jin/Practical-Guide-to-Context-Engineering)
> 

分析参考来源：

- 黑客马拉松冠军 Claude Code 配置合集： [GitHub - affaan-m/everything-claude-code: Complete Claude Code configuration collection - agents, skills, hooks, commands, rules, MCPs. Battle-tested configs from an Anthropic hackathon winner.](https://github.com/affaan-m/everything-claude-code)
- Anthropic官方Skill配置说明： [GitHub - anthropics/skills: Public repository for Agent Skills](https://github.com/anthropics/skills)
- ClaudeCode配置文件： [Create custom subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)

## [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-3) 一、跨会话共享内存**

[\ 跨会话共享内存2137×1026 75 KB](https://linux.do/uploads/default/original/4X/c/4/4/c4445f586bd0547a4bbfce5b4217f509baf7bfd8.webp)

![跨会话共享内存](https://linux.do/uploads/default/optimized/4X/c/4/4/c4445f586bd0547a4bbfce5b4217f509baf7bfd8_2_690x331.webp)

在使用 ClaudeCode 的过程中，会话记录虽然可以被保存到本地文件中，也可以使用 resume 指令继续上一次对话，但是完整会话记录会被压缩，如果经过多次压缩，关键的决策信息会被不断的稀释，直到完全忘记，那么我理解的关键信息就是如下四点：

1. **哪些方法是有效的（有证据可以验证）**
2. **哪些尝试过的方法是无效的**
3. **哪些方法尚未尝试**
4. **哪些未完成的工作**

![:ringed_planet:](https://linux.do/images/emoji/twemoji/ringed_planet.png?v=15)

所以为了保证上面四点关键的信息能够在多个会话共享，就需要有一个

**单独的中间态的临时会话文件**

这个时候是需要完整的 **文件创建和保存的自动化流程，还有填充内容的指令**，可以采用三种 hook 来解决这个文件的创建、加载、保存的问题，

1. 预压缩钩子（ **PreCompact Hook）**：在上下文压缩发生之前，将重要状态保存至文件
2. 会话完成钩子（ **SessionComplete Hook）**：会话结束时，将学习成果持久化至文件或初始化文件
3. 会话开始钩子（ **SessionStart Hook）**：新会话启动时，自动加载先前上下文，并输出最新文件的路径

那么这个临时文件（会话记录文件）我们要填充什么内容进去？，怎么填充？

- 填充的内容：可以根据上面四点信息方向来让 Claude 总结会话历史、你也可以手动编写该文件，整理写入你认为重要的会话关键信息
- 填充的方式：你可以在聊天会话中主动提及，也可以创建相应的 Skill 和 Command 来使用

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

这个模式下使用到的文件 hook 为：

- scripts/hooks/session-start.js：会话开始钩子
- scripts/hooks/pre-compact.js：会话压缩钩子
- scripts/hooks/session-end.js：会话完成钩子

## [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-4) 二、持续学习并更新记忆**

[\ 持续学习流程1397×1047 60 KB](https://linux.do/uploads/default/original/4X/8/f/b/8fb3612f964e4980bd2a129f0b3261f561d52b27.webp)

![持续学习流程](https://linux.do/uploads/default/optimized/4X/8/f/b/8fb3612f964e4980bd2a129f0b3261f561d52b27_2_667x500.webp)

关于持续学习并更新记忆的触发方式有两种： **一种是 Hook 的挂载脚本自动执行的方式、另外一种是 Command 命令手动执行的方式**

![:puzzle_piece:](https://linux.do/images/emoji/twemoji/puzzle_piece.png?v=15)

关于 Hook 的挂载脚本自动执行的方式，该方式使用了三种不同时机的 Hook 来触发

1. **Stop**的 Hook 的具体逻辑：
2. 简单的对于会话列表的长度进行判断
3. 如果长度达标、那么输出提示词信息给用户看
4. **Sessionend**的 Hook 的具体逻辑：
5. 读取完整的会话历史记录
6. 通过claude -p “xxx”，来调用claude生成学习记录
7. **PostToolUse**的 Hook 的具体逻辑：
8. 简单的进行会话列表的长度判断
9. 将“总结学习记录的指令”放入到工具返回结果，以此输入给Claude触发判断

![:puzzle_piece:](https://linux.do/images/emoji/twemoji/puzzle_piece.png?v=15)

关于 Cmmand 命令手动执行的方式、该方式设计了/learn 指令来执行

当用户在会话中完成任务的时候，发现有一些设计方案非常值得保存到记忆中，这个时候就可以触发/learn 指令

**所有总结下来的学习记录都存放在/skill/learn 文件夹中，这样或许 Agent 可以自动根据具体情况使用 Skill 的学习记录的技能**

---

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

模式下使用到的文件 Command 和 Skill 为：

- commands/learn.md
- skills/continuous-learning

![:backhand_index_pointing_right:](https://linux.do/images/emoji/twemoji/backhand_index_pointing_right.png?v=15)

小拓展：

关于会话记录文件(Session Tmp)和学习记录文件(Learn Skill)的区别

- 学习记录文件（Learn Skill）：全局的、永久的、抽象的知识规则， **目的是避免重复犯错，积累经验**
- 会话记录文件（Session Tmp）：局部的、临时的、具体的工作状态， **目的是用于跨会话的连续性**

---

文件所在的位置也不同：

- 学习记录文件（Learn Skill）：/.claude/skills/learned/jsonwebtoken-v9-migration.md
- 会话记录文件（Session Tmp）：/.claude/sessions/2026-01-20-auth-feature.tmp

---

学习记录文件的例子：

- 工具的描述要说明"何时使用"而不只是"做什么"
- 工具参数的设计中必填参数尽量少，可选参数提供默认值
- 件路径参数要说明是相对路径还是绝对路径

---

会话记录文件的例子：

- 创建了 `database_query` 工具的基础定义
- 在工具描述中明确说明"优先使用 simple_query，只有必要时才用 complex_query"

## [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-5) 三、提高项目可维护性 - 检测评估+冗余代码清理**

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-31-6) 3.1、基于检查点的评估**

[\ 检查点评估3055×1602 102 KB](https://linux.do/uploads/default/original/4X/e/1/6/e16b658a10683224f46e691b1472501582d35a1d.webp)

![检查点评估](https://linux.do/uploads/default/optimized/4X/e/1/6/e16b658a10683224f46e691b1472501582d35a1d_2_690x361.webp)

使用基于检查点评估的方式，流程就是： **开始 → 实现 → 验证** 这三步循环，每一个功能都由这三步来约束

1. **开始**：在功能实现之前，先运行一下开始步骤的指令，这样可以保证代码库的工作空间是干净的
2. **实现**：这个时候就可以开始编写功能代码了，你自己编写或使用 AI 编写都可以
3. **验证**：在这个功能完成的差不多了，或者已经编写一段时间的代码啦，可以运行验证指令，进行简单的代码评估，评估验证你这段时间写的代码如何，是否合格，是否符合要求规范

![:puzzle_piece:](https://linux.do/images/emoji/twemoji/puzzle_piece.png?v=15)

那么我们来细细的说一下开始指令( /checkpoints create )

1. 执行/verify quick 的指令，只检查构建和类型的错误，这样执行起来快
2. 执行 git stash 或 commit ，保存当前的代码状态，让功能开始之前工作空间是干净的
3. 执行`git rev-parse --short HEAD` 把 SHA 写入到日志文件`checkpoints.log`中

![:puzzle_piece:](https://linux.do/images/emoji/twemoji/puzzle_piece.png?v=15)

接下来我们来说一下验证指令( /checkpoint verify )

1. 先从 checkpoints.log 中去除最近的 SHA
2. 大模型自己根据 **具体情况调用 git diff 和运行代码相关测试的命令**
3. 根据“ **关键指标**”的要求输出报告，报告中要有：新增文件、修改文件、测试通过率、覆盖率等

这种方式真的非常好，非常优雅， **因为所有的流程不是强制自动化的，只是提供了最小的必要条件**“SHA”。

至于如何获取新增和修改文件，测试和覆盖率这些指标，都是由模型自己来决定的，最大程度上保证了模型自主性，目前模型能力已经很厉害了，保持模型的自主性，我们未来可以用最小的改动代价换来最大的能力提升

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

该模式下使用到的文件 Command 和 Skill 为：

- commands/checkpoint.md
- commands/verify.md

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-32-7) 3.2、持续评估**

[\ 持续评估2463×1899 91.8 KB](https://linux.do/uploads/default/original/4X/0/7/5/07519fabea25a820e9bb5b9afcb99bd4645a22d5.webp)

![持续评估](https://linux.do/uploads/default/optimized/4X/0/7/5/07519fabea25a820e9bb5b9afcb99bd4645a22d5_2_648x500.webp)

在持续评估的模式下，具体的流程为：

1. **触发点**：每 N 分钟或重大变更后运行
2. **运行点**：运行完整的测试套件、构建状态、代码检查等，使用 Skill 或 Command
3. **运行结果**：输出完整详细的检测报告
4. **判断结束点**：根据输出的报告判断当前的代码是否合格，如果合格就结束，不合格的话就要进行修复
5. **修复点**：对于报告中不合格的检测点进行修复，成功修复之后结束

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

该模式下使用到的文件 Command 和 Skill 为：

- skills/verification-loop
- commands/verify.md

两种检测评估方式的区别：

1. 基于检查点评估的方式：适合用于具有明确里程碑的线性工作流程
2. 持久评估的方式：适合长时间运行的会话

![:glowing_star:](https://linux.do/images/emoji/twemoji/glowing_star.png?v=15)

**所以决定因素是任务的性质，基于检查点评估方式适用于具有明确阶段的特性实现，持续性的评估方式适用于探索性重构或维护，这类工作是没有明确的里程碑和结束点的**

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-33-8) 3.3、清理冗余代码**

清理冗余代码的功能是使用一个子 Agent 和一个 Command， **区别的话就是 Agent 更加详细完整一些，Command 指令轻便一点，直接一点**：

1. Command 指令： commands/refactor-clean.md
2. 子 Agent 的设计： agents/refactor-cleaner.md

目前 Command 指令更加清晰直接一些，我们接下来就分析整理一下这个命令书写的流程吧

```
# 重构清理

通过测试验证，安全识别并移除死代码：

1. 运行死代码分析工具：
    - knip：查找未使用的导出与文件
    - depcheck：查找未使用依赖
    - ts-prune：查找未使用的 TypeScript 导出
2. 在 .reports/dead-code-analysis.md 中生成完整报告
3. 按严重程度分类发现结果：
    - SAFE：测试文件、未使用的工具
    - CAUTION：API 路由、组件
    - DANGER：配置文件、主入口
4. 只建议安全删除项
5. 每次删除前：
    - 运行完整测试套件
    - 确认测试通过
    - 应用变更
    - 再次运行测试
    - 如失败则回滚
6. 展示已清理项目的汇总

在运行测试之前，绝不删除代码！

```

1. 先使用工具找出冗余代码：`knip`、`depcheck`、`ts-prune` 这三种工具
2. 把结果写入到文件中，同时对结果进行风险分类
3. 按照“完全流程”进行删除冗余带你：安全流程是在删除代码的前后都要运行测试，也就是(测试 - 删除 - 测试）

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-34-9) 3.4、代码地图 - 可信的上下文**

[\ codeMap3187×1385 105 KB](https://linux.do/uploads/default/original/4X/5/8/3/583e50a096bae5ab0bdd7913e3c9ebc49aca91bc.webp)

![codeMap](https://linux.do/uploads/default/optimized/4X/5/8/3/583e50a096bae5ab0bdd7913e3c9ebc49aca91bc_2_690x299.webp)

有代码地图的存在，其就可以作为 AI 或者开发者了解代码库的入口，AI 可以通过代码地图，使用较少的 Token 就可以了解项目的全局面貌。

代码地图里面的文档不要多，尽量保持精简。

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

该模式下使用到的文件 Command 和 Skill 为：

- agents/doc-updater.md
- commands/update-codemaps.md
- commands/update-docs.md

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-35-10) 3.5、一点总结**

那我们总结一下吧，关于提高项目可维护性的四种方式：

- 在适当干预下，两 **种验证方法（基于检查点评估和持续评估）足以避免大部分技术债务**。让 Claude 完成任务后通过运行技能和 PostToolUse 钩子进行验证，有助于实现这一点。
- **持续更新代码地图也有帮助，因为它记录了变更日志以及代码地图随时间的演变过程**，这提供了除代码仓库本身之外的可靠信息来源。
- **通过严格的规则，Claude 将避免创建杂乱的随机.md 文件，避免为相似代码生成重复文件**，也不会留下大量废弃代码。可以考虑rules的方式，全局的创建文档的规则

## [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-11) 四、子智能体的使用方式：循环验证调用+编排**

使用子智能体的方式会导致整个会话的上下文产生“中断”

子代理的存在是为了通过返回摘要而非全部信息来节省上下文。然而，编排器拥有子代理所缺乏的语义上下文。子代理只知道字面查询，不了解请求背后的目的或推理过程。摘要常常遗漏关键细节

> 来自@ PerceptualPeak 的类比：“你的老板派你去开会并要求你提供摘要。你回来后向他汇报了情况。十有八九，他会有后续问题。你的摘要不会包含他需要的所有信息，因为你没有他那种隐含的上下文。”
> 

所以目前更好使用子代理的方式有两种 **：循环验证调用模式和顺序执行的编排器**

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-41-12) 4.1、循环验证调用**

[\ 循环调用1908×1083 42.9 KB](https://linux.do/uploads/default/original/4X/6/3/f/63fd9daf0d5db5aef839126ec28f0ea400412462.webp)

![循环调用](https://linux.do/uploads/default/optimized/4X/6/3/f/63fd9daf0d5db5aef839126ec28f0ea400412462_2_690x391.webp)

循环验证调用的流程：

1. 主智能体评估子智能体的结果
2. 当结果不合格的话，主智能体根据评估结果提出新的检索任务
3. 子智能体按照新的检索任务继续检索
4. 循环最多 3 轮

**在这种模式下，主智能体派发给子智能体的任务要“具体的问题+更广泛的目标”，让整体的检索面积更大，能检索到更多的结果**

---

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

该模式下使用到的文件 Command 和 Skill 为：

- **skills/iterative-retrieval**

### [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-42-13) 4.2、编排智能体**

[\ 编排智能体1184×479 29.5 KB](https://linux.do/uploads/default/original/4X/a/c/5/ac53be4917fcb28c7501dbf48704907b49091d27.webp)

![编排智能体](https://linux.do/uploads/default/optimized/4X/a/c/5/ac53be4917fcb28c7501dbf48704907b49091d27_2_690x279.webp)

编排智能体的原则是：

1. 每个代理接收一个明确的输入、并生成一个明确的输出
2. 输出成为下一阶段的输入
3. 切勿跳过任何阶段 - 每个阶段都蕴含价值
4. 在智能体之间使用/clear命令以保持上下文的新鲜度
5. 将中间输出存储在文件中（而非仅存在内存中）

除了固定的四种功能编排好的子智能体调用流程，还可以自定义子智能体的调用顺序

```bash
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"

```

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

该模式下使用到的文件 Command 和 Skill 为：

- commands/orchestrate.md
- agents/architect.md
- agents/code-reviewer.md
- agents/planner.md
- agents/security-reviewer.md
- agents/tdd-guide.md

## [**标题链接](https://linux.do/t/topic/1549626#p-13323208-h-14) 五、文章配置详解**

```
everything-claude-code/
|-- .claude-plugin/         # 插件和市场清单
|   |-- plugin.json         # 插件元数据和组件路径
|   |-- marketplace.json    # /plugin marketplace add 的市场目录
|
|-- agents/           # 子智能体文件夹
|   |-- planner.md           		 # 功能实现规划
|   |-- architect.md         		 # 系统设计决策
|   |-- tdd-guide.md         		 # 测试驱动开发
|   |-- code-reviewer.md     		 # 质量和安全审查
|   |-- security-reviewer.md     # 漏洞分析
|   |-- build-error-resolver.md  # 构建错误解决
|   |-- e2e-runner.md        		 # Playwright E2E 测试
|   |-- refactor-cleaner.md  		 # 死代码清理
|   |-- doc-updater.md       	 	 # 文档同步
|   |-- go-reviewer.md       		 # Go 代码审查（新增）
|   |-- go-build-resolver.md 		 # Go 构建错误解决（新增）
|
|-- skills/           # 技能
|   |-- coding-standards/           # 编程语言最佳实践
|   |-- backend-patterns/           # API、数据库、缓存模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从会话中自动提取模式（长篇指南）
|   |-- continuous-learning-v2/     # 基于直觉的学习，带置信度评分
|   |-- iterative-retrieval/        # 子智能体的渐进式上下文优化
|   |-- strategic-compact/          # 手动压缩建议（长篇指南）
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- eval-harness/               # 验证循环评估（长篇指南）
|   |-- verification-loop/          # 持续验证（长篇指南）
|   |-- golang-patterns/            # Go 语言习惯用法和最佳实践（新增）
|   |-- golang-testing/             # Go 测试模式、TDD、基准测试（新增）
|
|-- commands/         # 命令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现规划
|   |-- e2e.md              # /e2e - E2E 测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 修复构建错误
|   |-- refactor-clean.md   # /refactor-clean - 死代码移除
|   |-- learn.md            # /learn - 会话中提取模式（长篇指南）
|   |-- checkpoint.md       # /checkpoint - 保存验证状态（长篇指南）
|   |-- verify.md           # /verify - 运行验证循环（长篇指南）
|   |-- setup-pm.md         # /setup-pm - 配置包管理器
|   |-- go-review.md        # /go-review - Go 代码审查（新增）
|   |-- go-test.md          # /go-test - Go TDD 工作流（新增）
|   |-- go-build.md         # /go-build - 修复 Go 构建错误（新增）
|   |-- skill-create.md     # /skill-create - 从 git 历史生成技能（新增）
|   |-- instinct-status.md  # /instinct-status - 查看学习到的直觉（新增）
|   |-- instinct-import.md  # /instinct-import - 导入直觉（新增）
|   |-- instinct-export.md  # /instinct-export - 导出直觉（新增）
|   |-- evolve.md           # /evolve - 将直觉聚类为技能（新增）
|
|-- rules/            # 始终遵循的指南（复制到 ~/.claude/rules/）
|   |-- security.md         # 强制性安全检查
|   |-- coding-style.md     # 不可变性、文件组织
|   |-- testing.md          # TDD、80% 覆盖率要求
|   |-- git-workflow.md     # 提交格式、PR 流程
|   |-- agents.md           # 何时委托给子智能体
|   |-- performance.md      # 模型选择、上下文管理
|
|-- hooks/            # 基于触发器的自动化
|   |-- hooks.json                # 所有钩子配置（PreToolUse、PostToolUse、Stop 等）
|   |-- memory-persistence/       # 会话生命周期钩子（长篇指南）
|   |-- strategic-compact/        # 压缩建议（长篇指南）
|
|-- scripts/          # 跨平台 Node.js 脚本（新增）
|   |-- lib/                     # 共享工具
|   |   |-- utils.js             # 跨平台文件/路径/系统工具
|   |   |-- package-manager.js   # 包管理器检测和选择
|   |-- hooks/                   # 钩子实现
|   |   |-- session-start.js     # 会话开始时加载上下文
|   |   |-- session-end.js       # 会话结束时保存状态
|   |   |-- pre-compact.js       # 压缩前状态保存
|   |   |-- suggest-compact.js   # 战略性压缩建议
|   |   |-- evaluate-session.js  # 从会话中提取模式
|   |-- setup-package-manager.js # 交互式包管理器设置
|
|-- tests/            # 测试套件（新增）
|   |-- lib/                     # 库测试
|   |-- hooks/                   # 钩子测试
|   |-- run-all.js               # 运行所有测试
|
|-- contexts/         # 动态系统提示注入上下文（长篇指南）
|   |-- dev.md              # 开发模式上下文
|   |-- review.md           # 代码审查模式上下文
|   |-- research.md         # 研究/探索模式上下文
|
|-- examples/         # 示例配置和会话
|   |-- CLAUDE.md           # 项目级配置示例
|   |-- user-CLAUDE.md      # 用户级配置示例
|
|-- mcp-configs/      # MCP 服务器配置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- marketplace.json  # 自托管市场配置（用于 /plugin marketplace add）

```

![:palm_tree:](https://linux.do/images/emoji/twemoji/palm_tree.png?v=15)

按照核心功能拆分使用的话：

1. 跨会话共享内存
    - scripts/hooks/session-start.js：会话开始钩子
    - scripts/hooks/pre-compact.js：会话压缩钩子
    - scripts/hooks/session-end.js：会话完成钩子
2. 持续学习并更新记忆：
    - commands/learn.md
    - skills/continuous-learning
3. 提高项目的可维护性：
    - commands/checkpoint.md
    - commands/verify.md
    - skills/verification-loop
    - commands/refactor-clean.md
    - agents/refactor-cleaner.md
    - agents/doc-updater.md
    - commands/update-codemaps.md
    - commands/update-docs.md
4. 子智能体使用方式：
    - skills/iterative-retrieval
    - commands/orchestrate.md
    - agents/architect.md
    - agents/code-reviewer.md
    - agents/planner.md
    - agents/security-reviewer.md
    - agents/tdd-guide.md