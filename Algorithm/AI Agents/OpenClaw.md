OpenClaw 是一个开源的、可本地部署的AI智能体（AI Agent）平台，其核心价值在于，不只是一个能对话的聊天机器人，更是一个能真正理解指令并执行复杂任务的数字员工。


![](_imgs/Pasted%20image%2020260303160507.png)



## 数据流转

```mermaid
sequenceDiagram
    participant C as Channel
    participant A as Adapter
    participant O as Orchestrator
    participant P as Plugin
    participant M as Memory
    
    C->>A: Raw message
    A->>A: Protocol normalization
    A->>O: Normalized intent
    O->>M: Retrieve context
    O->>O: LLM planning
    O->>P: Execute primitive
    P->>M: Log execution
    P-->>O: Result
    O-->>A: Formatted response
    A-->>C: Channel-specific reply
```



## [Pi Agent](Pi%20Agent.md)

OpenClaw 的核心 Agent 框架是 [pi-mono](https://github.com/badlogic/pi-mono) ，其是一个极简、优雅的 AI Agent 框架，提供了 Agent 运行所必须的所有功能，OpenClaw 在 Pi 的基础上进行了扩展，加入了网关、多端支持、记忆等功能。


## 记忆系统

OpenClaw 选择 Markdown 文件作为记忆的“唯一真实来源”，所有记忆都清晰可见。同时，为了高效检索，它会利用向量数据库（如 Milvus）为这些文件建立索引。这种设计意味着你可以像编辑文档一样，直接打开、修改、甚至用 Git 管理 AI 的记忆，既透明又强大。



