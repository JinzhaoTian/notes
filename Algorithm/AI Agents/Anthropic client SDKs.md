Anthropic Client SDKs 是 Anthropic 官方提供的编程工具包，旨在帮助开发者通过代码轻松、安全地调用 Claude API，而无需手动处理复杂的 HTTP 请求和响应解析。

| SDK                     | 主要语言/平台                 | 特点                                                                                                                       |
| ----------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **`@anthropic-ai/sdk`** | TypeScript / JavaScript | 官方 TypeScript 库，支持 Node.js，提供完整的类型定义和流式响应。                                                                               |
| **`anthropic`**         | Python                  | 官方 [Python](../../Language/Python/AI%20Agents/Anthropic%20Python%20SDK.md) 库，同时提供同步和异步客户端，与 Pydantic v1/v2 兼容，并集成了自动重试逻辑 |
## 核心功能

1. **完整的 API 覆盖**：全面支持 Anthropic 最新的 API 功能，如 **Messages API**（多轮对话）、**工具调用**（让 Claude 使用外部工具）和**流式响应**（实现打字机效果）。
2. **便捷的配置与使用**：只需提供 API 密钥即可初始化客户端，所有复杂的请求头、身份验证细节都被封装在内部。例如，发送一条消息非常简单：
```python
from anthropic import Anthropic

client = Anthropic()

response = client.messages.create(
	model="claude-3-5-sonnet-latest",
	max_tokens=1024,
	messages=[{"role": "user", "content": "Hello!"}]
)
```

3. **健壮的错误处理**：内置了针对网络问题、速率限制、服务端错误（如 `5xx` 错误）的**自动重试机制**（默认重试2次），并采用指数退避策略，提高了应用的稳定性。
4. **优秀的开发体验**：
    - **类型安全**：TypeScript 和 Python SDK 都提供了详尽的类型定义，在编码阶段就能获得智能提示和参数校验，减少错误。
    - **流式支持**：通过 `async for` 循环可以轻松处理流式响应，实现实时输出。
    - **多平台支持**：除了官方库，社区也为其他语言提供了客户端。

