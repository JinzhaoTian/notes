LangChain 是一个用于构建基于大型语言模型（LLM）应用程序的框架，旨在简化将 LLM 与其他工具、数据源和业务逻辑集成的过程。它提供了一套模块化的组件和工具，帮助开发者快速搭建高效的 AI 应用，如问答系统、聊天机器人、自动化工作流等。


## 核心功能

1. **模块化设计**：LangChain 提供可组合的组件（如模型、提示模板、记忆、索引等），开发者可以灵活组装这些模块来构建复杂应用。
2. **支持多种语言模型**：兼容 OpenAI、Anthropic、Hugging Face 等主流 LLM，也支持本地部署的模型（如 Llama2、GPT-J）。
3. **数据增强（Retrieval-Augmented Generation, RAG）**：支持将外部数据（文档、数据库、API）与 LLM 结合，通过检索增强生成更准确的回答。
4. **记忆（Memory）**：可管理对话历史或上下文，使模型在多轮交互中保持连贯性（如聊天机器人）。
5. **代理（Agents）**：允许 LLM 动态调用工具（如搜索引擎、计算器、自定义函数），完成复杂任务。
6. **多语言支持**：主要支持 Python 和 JavaScript（TypeScript），覆盖前后端开发。


## 应用场景

- **问答系统**：基于文档或知识库的智能问答。
- **聊天机器人**：支持多轮对话和个性化回复。
- **文本生成**：自动生成报告、摘要、代码等。
- **数据分析和处理**：结合工具处理结构化/非结构化数据。
- **自动化工作流**：如邮件分类、客服工单处理等。


## 关键组件

1. **Models**：对接不同 LLM 或嵌入模型。
2. **Prompts**：管理提示模板，优化输入输出。
3. **Chains**：将多个组件串联成完整流程（如“检索→生成”）。
4. **Indexes**：处理外部数据（文档加载、向量存储、检索）。
5. **Memory**：存储对话或交互状态。
6. **Agents**：让 LLM 自主选择工具执行任务。


## 示例代码

### [Python](../../Language/Python/AI%20Agents/LangChain.md)

```python
from langchain_community.llms import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

# 1. 初始化模型
llm = OpenAI(model_name="gpt-3.5-turbo")

# 2. 创建提示模板
prompt = PromptTemplate(
    input_variables=["topic"],
    template="用简单的话解释一下{topic}的概念。"
)

# 3. 构建链并运行
chain = LLMChain(llm=llm, prompt=prompt)
response = chain.run(topic="量子计算")
print(response)
```

### [TypeScript](../../Language/Node/AI%20Agents/LangChain.md)

