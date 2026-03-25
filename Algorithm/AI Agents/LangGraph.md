LangGraph 是一个用于构建有状态、多步骤、带循环的 AI Agent 工作流的底层编排框架，底层逻辑遵循 [ReAct](ReAct.md) 范式。

## 核心概念

LangGraph 主要围绕三个核心概念构建，并借此实现了一些关键的高级功能：
1. **State（状态）**：一个在所有节点间共享的**公共数据结构**，存储着对话历史、当前数据等所有必要信息。每个节点都能读取和更新它，且你可以通过 Reducer 函数来精确控制状态的更新方式，比如是覆盖还是追加。
2. **Node（节点）**：工作流中的**基本处理单元**。一个节点可以是一个调用大模型的步骤、一个执行特定功能的函数、或者是一个工具调用。你负责在节点里编写具体逻辑。
3. **Edge（边）**：定义了节点间的**执行顺序和逻辑流向**。你可以使用普通的边来固定执行顺序，也可以用**条件边**来实现分支判断，让AI根据当前状态决定下一步走向哪个节点。

## 核心能力

基于这种图结构，LangGraph 提供了一些普通链式结构难以实现的能力：
1. **持久化执行与记忆**：在执行过程中可以自动保存状态。即使程序崩溃或任务需要暂停，也能从断点处**精确恢复**。这为长时间运行的任务提供了基础，并实现了跨会话的长期记忆。
2. **人机协作（Human-in-the-loop）**：可以在工作流的任意节点设置"断点"，暂停执行。这时你可以**人工介入**，检查和修改当前状态，确认无误后再让智能体继续执行，极大地增强了可控性和安全性。
3. **循环与复杂逻辑**：通过图结构，你可以轻松实现思考-行动-观察这样的循环，这是构建高级智能体（如 ReAct 模式）的核心。
4. **生产就绪**：支持流式输出，并且可以与 LangGraph 平台结合，实现一键部署、任务队列、自动扩缩容等生产级能力。


## 使用示例

### [Python](../../Language/Python/AI%20Agents/LangGraph.md)

```python
# 1. 安装：pip install -U langgraph

from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

# 2. 定义状态结构
class MyState(TypedDict):
    text: str

# 3. 定义一个节点函数，它接收状态并返回更新
def my_node(state: MyState) -> dict:
    # 在原有文本后添加 "hello"
    return {"text": state["text"] + " hello"}

# 4. 构建图
graph_builder = StateGraph(MyState)      # 初始化状态图
graph_builder.add_node("node_a", my_node) # 添加节点
graph_builder.add_edge(START, "node_a")   # 从开始节点指向 node_a
graph_builder.add_edge("node_a", END)     # 从 node_a 指向结束

# 5. 编译并运行
graph = graph_builder.compile()
result = graph.invoke({"text": "world"})

print(result)  # 输出: {'text': 'world hello'}
```

### [TypeScript](../../Language/Node/AI%20Agents/LangGraph.md)

