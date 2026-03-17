Harness Engineering（Harness：控制、测试和约束）是指围绕 AI Agent（特别是 Coding Agent）设计和构建约束机制、反馈回路、工作流控制和持续改进循环的系统工程实践，其解决的核心问题是：当 AI Agent 拥有了强大的代码生成能力后，如何确保其输出的可靠性、一致性和长期可维护性。

Harness Engineering 是 Prompt Engineering 和 Context Engineering 的自然延伸，三者构成嵌套关系：
- Context Engineering 管的是"给 Agent 看什么"
- Harness Engineering 管的是"系统怎么防崩、怎么量化、怎么修".

## 为什么需要 Harness Engineering

### 模型能力不是瓶颈

五个独立团队得出了相同结论：基础设施才是瓶颈，而非智能水平。这一判断得到了量化验证：
- Can.ac 实验：仅改变 Harness 的工具格式（编辑接口），就在 16 个模型上显著提升了编码基准分数。效果最显著的 Grok Code Fast 1 从 6.7% 跃升至 68.3%——没有修改任何模型权重.
- LangChain实验：仅通过 Harness 改进，在 Terminal Bench 2.0 上从第 30 名跃升至第 5 名，同一模型提升了 13.7 分。

这些结果表明：在纠结模型选择之前，先审视 Harness 设计能获得更高的投资回报率。OpenAI 团队说得很直接：**真正卡你的不是 Agent 写代码的能力，而是围绕它的结构、工具和反馈机制跟不上**。

### Agent 的典型失败模式

Anthropic 在做长时间运行 Agent 的过程中，总结了 Agent 常见的翻车姿势：
- **失败模式 1：试图一步到位（One-shotting）。** Agent 倾向于一次做完所有事情，结果在实现进行到一半时上下文窗口就耗尽了。下一个会话启动时看到的是半成品、没有文档的代码，只能花大量时间猜测之前发生了什么并试图恢复工作状态。
- **失败模式 2：过早宣布胜利。** 在项目后期，当部分功能已经完成后，Agent 会环顾四周，看到已有进展就直接宣布任务完成——即使还有大量功能未实现。
- **失败模式 3：过早标记功能完成。** 在没有明确提示的情况下，Agent 写完代码就标记为“完成”，却没有做端到端测试。单元测试或 curl 命令通过了不代表功能真正可用。
- **失败模式 4：环境启动困难。** 每次新会话启动时，Agent 需要花费大量 token 弄清楚如何运行应用、如何启动开发服务器，而不是把时间花在实际开发上。

### 上下文窗口利用率的甜蜜区间

Dex Horthy 有个很实用的经验观察：**上下文填得越满，LLM 输出质量越差**。以 168K token 的上下文窗口为例，大约用到 40% 就开始走下坡路了：
- **Smart Zone（前约 40%）**：聚焦、准确的推理。Agent 拥有相关、精炼的信息。
- **Dumb Zone（超过约 40%）**：幻觉、循环、格式错误的工具调用、低质量代码。更多 token 反而损害性能。




## 四大支柱

综合 OpenAI、Anthropic、Carlini（C 编译器项目）、Huntley、Horthy 等五个独立团队的实践，有四种模式反复出现并形成收敛，这就是 Harness Engineering 的四大支柱。

### 上下文架构（Context Architecture）

**核心原则**：Agent 应当恰好获得当前任务所需的上下文。

每个团队都独立发现，将所有指令塞进一个文件无法扩展，解决方案是**分层上下文与渐进式披露**：
- OpenAI 使用 `AGENTS.md` 作为动态反馈循环文件，每当 Agent 遇到失败时更新。
- Anthropic 使用大量 `README` 和每次会话频繁更新的进度文件。
- Horthy 倡导"频繁有意识压缩"（Frequent Intentional Compaction）。
* Vasilopoulos（2026 论文）将上下文形式化为三层：热记忆（Hot Memory）、领域专家（Domain Experts）、冷记忆知识库（Cold-Memory Knowledge）。

### Agent 专业化（Agent Specialization）

**核心原则**：专注于特定领域、拥有受限工具的 Agent 优于拥有全部权限的通用 Agent。

* Carlini（Anthropic C 编译器项目）将 Agent 专业化为编译器核心、去重、性能优化和文档四类角色。
* Vasilopoulos 部署了 19 个领域特定 Agent。
* Huntley 使用子 Agent 来保持主 Agent 上下文的清洁。

专业化不仅是组织性的，其本身就是上下文管理策略。每个专家因为携带更少的无关信息，所以运行在"Smart Zone"内。

### 持久化记忆（Persistent Memory）

**核心原则**：进度持久化在文件系统上，而非上下文窗口中。每次新 Agent 会话从零开始，通过文件系统产物重建上下文。

Anthropic 解决这一问题的方案堪称经典：
1. **初始化 Agent**：首次会话使用专门的 prompt，要求模型建立初始环境—— `init.sh` 脚本、`claude-progress.txt` 进度文件和初始 git 提交。
2. **Coding Agent**：后续每次会话要求模型在做出增量进展的同时，留下结构化更新。每个 Coding Agent 的典型会话启动流程如下：
	- 运行 `pwd` 查看工作目录
	- 读取 `git log` 和进度文件，了解最近的工作
	- 读取 feature list 文件，选择最高优先级的未完成功能
	- 启动开发服务器，运行基础端到端测试
	- 确认基本功能正常后，开始新功能开发

关键发现：**使用 JSON 格式追踪 feature 状态比 Markdown 更有效**，因为 Agent 不太可能不恰当地修改或覆盖结构化数据。

### 结构化执行（Structured Execution）

**核心原则**：将思考与执行分离。研究和规划在受控阶段进行，执行基于验证过的计划，验证通过自动化反馈（测试、Linter、CI）和人类审查完成。

* OpenAI 使用声明式 prompt 和反馈回路。轻量的计划用于小变更，复杂工作通过带有进度和决策日志的执行计划完成，并检入仓库。
* Huntley 将规划模式与构建模式分离。
* Horthy 的 Research-Plan-Implement 工作流围绕上下文管理精心设计。








## 参考

[Harness engineering: leveraging Codex in an agent-first world | OpenAI](https://openai.com/index/harness-engineering/)

作者：汉松  
链接：https://www.zhihu.com/question/590636216/answer/2012087634462257754  
来源：知乎  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。  
  

相比之下，我更看好 Every 提出的 [复利工程](https://link.zhihu.com/?target=https%3A//github.com/EveryInc/compound-engineering-plugin)（Compound Engineering）。它的核心不是“先写 spec”，而是让每次工程工作都能产生复利效应。具体来说，就是把计划、执行、review 的产出和踩坑经验都记录下来，下一轮就能更快、更稳。Every 的文章把这个循环定义为 Plan → Work → Review → Compound，并强调“80% 的价值在 plan 和 review 这两步”。

如何理解“复利”？传统的 Vibe Coding 追求短期收益：你输入 prompt，AI 生成代码，短平快的从零做一个应用，然后周而复始。而复利工程想做的是一个有记忆的系统。每个 PR 都在教 AI 学东西，每个错误都变成永久经验，每次代码审查都更新默认设置。简单来说，**Vibe Coding 让你今天更快，复利工程让你明天更快，而且一天比一天快。**

OpenAI 在实践中也提到，他们给 AI 准备了一个知识库，把文档结构化地存进去，方便 AI 理解和使用。但他们只给出了结果，没说具体怎么实现。

这恰好印证了一个核心需求：**要让 AI 能够持续迭代，就必须给它配一套知识记录系统**。而 Every 的“复利工程”，在我看来正是这个问题的完美解决方案。

上面这些讨论，都是我看完那篇文章后的一些想法。但想法归想法，具体怎么落地还得靠实践。下面我就分享一下，我在“复利工程”和“给 AI 可验证环境”这两件事上的一些经验。

正好我们最近正在重构从零搭建阿福的 Agent 新架构，有了一次从零开始的机会。所以我就在想，能不能在这个应用刚开始的时候，就完全让 AI 去介入，让它像 OpenAI 的实践一样，把所有的上下文和架构规范都落到代码仓库里面。所以我也需要去探索这么一种新的与 AI 协作的方式，也就是 Harness 工程。