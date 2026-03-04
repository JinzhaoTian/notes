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