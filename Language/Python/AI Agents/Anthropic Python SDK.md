Python `anthropic` 库是 Anthropic 公司官方出品的 Python SDK，主要用于通过编程方式访问 Claude 大模型。

## 核心功能

这个库封装了 Anthropic 的 REST API，让你可以用 Python 代码轻松构建复杂的 AI 应用。它的主要特性包括：

1. **简单易用**：提供了一个清晰、符合 Python 习惯的接口，只需几行代码就能发送请求并获取 Claude 的回复。
2. **类型安全**：内置了完整的类型定义 (TypedDicts 和 Pydantic 模型)，在你写代码时就能提供自动补全和类型检查，大大减少因参数拼写错误导致的问题。
3. **支持同步和异步**：同时提供了 `Anthropic` (同步) 和 `AsyncAnthropic` (异步) 两个客户端，你可以根据项目需求灵活选择，异步版本在构建高并发的 Web 应用时性能更好。
4. **流式响应**：支持以流的形式逐字接收 Claude 的回复，这在需要实时交互体验的场景（如聊天机器人）中非常有用。
5. **高级功能支持**：不仅支持基础的对话，还全面支持工具调用（Tool Use / Function Calling）、消息批处理、Token 计数等高级特性。

## 安装

```bash
pip install anthropic
```

