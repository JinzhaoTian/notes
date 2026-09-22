目前大模型 API 接入协议主要有以下**四大标准**，它们在设计哲学和适用场景上各有侧重：

| 标准                                | 核心端点                   | 设计哲学                | 适用场景                      |
| --------------------------------- | ---------------------- | ------------------- | ------------------------- |
| **OpenAI Chat Completions**       | `/v1/chat/completions` | 无状态、对话式，客户端维护历史     | 通用对话、快速集成、多厂商兼容           |
| **OpenAI Responses**              | `/v1/responses`        | 有状态、Agent式，服务端管理上下文 | 构建Agent、内置工具、长任务          |
| **Anthropic Messages**            | `/v1/messages`         | 无状态，系统提示独立传递        | Claude生态、Claude Code、精细控制 |
| **Google Gemini GenerateContent** | `:generateContent`     | 原生多模态，支持思维链控制       | Gemini原生能力、企业级部署          |
## OpenAI Chat Completions API

```
POST https://api.openai.com/v1/chat/completions
```
### 认证

认证方式：`Authorization: Bearer $OPENAI_API_KEY`，请求头需包含 `Content-Type: application/json`。

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### 请求格式

请求体的核心结构是一个 `messages` 数组，每个元素代表对话中的一条消息。

```json
{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Tell me a joke."}
  ],
  "max_tokens": 100,
  "temperature": 0.7
}
```

**必填参数**：
- `model`（string）：模型 ID，如 `gpt-4o`、`o3`
- `messages`（array）：对话历史列表，按时间顺序排列

**消息角色（role）**：是 Chat Completions 的核心概念，支持以下类型：
- `system`：系统级指令，控制模型全局行为
- `user`：用户输入
- `assistant`：模型回复（用于多轮对话时回传历史）
- `developer`：在 o1 及更新模型中替代 `system` 角色，提供开发者级指令
- `tool`：工具调用结果（用于函数调用场景）

**常用可选参数**：
- `max_tokens`：生成的最大 token 数
- `temperature`（0–2）：采样温度，越低越确定性
- `top_p`：核采样
- `frequency_penalty` / `presence_penalty`（-2.0–2.0）：频率/存在惩罚，控制重复度
- `stream`（boolean）：是否流式返回
- `tools`：自定义函数列表（取代已弃用的 `functions`）

### 响应格式

返回 `ChatCompletion` 对象，核心字段为：
```json
{
  "id": "chatcmpl-xxx",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "..."},
      "finish_reason": "stop"
    }
  ],
  "usage": {"prompt_tokens": 10, "completion_tokens": 20, "total_tokens": 30}
}
```

文本内容位于 `choices[0].message.content`。


## OpenAI Responses API

```
POST https://api.openai.com/v1/responses
```

### 认证

与 Chat Completions 相同：`Authorization: Bearer $OPENAI_API_KEY`。

```bash
curl https://api.openai.com/v1/responses \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5",
    "input": "What is the capital of France?"
  }'
```

### 请求格式

Responses API 的核心设计变化在于用 `input` 替代 `messages`，并引入 `instructions` 独立承载系统提示。

```json
{
  "model": "gpt-5",
  "instructions": "You are a helpful assistant.",
  "input": "What is the capital of France?",
  "max_output_tokens": 500,
  "store": true
}
```

**核心参数**：
- `input`（string 或 array）：当前输入，可以是纯文本字符串，也可以是结构化的 input items 数组
- `instructions`（string）：系统/开发者指令，独立于对话历史
- `max_output_tokens`：输出 token 上限（含推理 token）
- `store`（boolean）：是否在服务端存储对话历史
- `previous_response_id`：引用之前的响应 ID 以延续对话（有状态模式）
- `conversation`：将响应关联到某个会话，输入/输出自动追加
- `include`：指定需要额外包含的输出数据，如 `reasoning.encrypted_content`、`web_search_call.action.sources` 等


**内置工具**：Responses API 支持直接在请求中启用平台内置工具，无需客户端实现执行循环：
- `web_search`：联网搜索
- `file_search`：文件检索
- `code_interpreter`：代码执行
- `computer_use`：计算机操作
- 远程 MCP 工具

### 响应格式

```json
{
  "id": "resp_xxx",
  "output": [
    {
      "type": "message",
      "content": [{"type": "output_text", "text": "Paris"}]
    }
  ],
  "output_text": "Paris",
  "usage": {...}
}
```

文本提取路径为 `output_text` 或 `output[0].content[0].text`。

### WebSocket 模式

Responses API 还支持 `wss://api.openai.com/v1/responses` 的持久连接模式，每轮仅发送 `response.create` 事件和 `previous_response_id`，适用于长时运行、大量工具调用的工作流


## Anthropic Messages API

```
POST https://api.anthropic.com/v1/messages
```

### 认证

认证方式：`x-api-key: $ANTHROPIC_API_KEY`，同时必须设置 `anthropic-version` 请求头（当前稳定版本为 `2023-06-01`）。可选使用 `anthropic-beta` 头开启 beta 功能。

### 请求格式

Messages API 采用无状态设计，系统提示通过独立的顶层 `system` 参数传递，`messages` 数组中**不存在** `system` 角色。

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Hello, Claude"}
  ]
}
```

**必填参数**：
- `model`（string）：模型 ID，如 `claude-opus-4-7`、`claude-sonnet-4-6`
- `max_tokens`（integer）：**必填**，单次生成的最大 token 数。这是与 OpenAI 格式最显著的区别之一，Anthropic 强制要求显式指定输出上限
- `messages`（array）：对话消息列表，角色仅支持 `user` 和 `assistant`，必须交替出现。单次请求最多支持 100,000 条消息

**`content` 字段**支持两种形式：
- 纯字符串
- 内容块数组，用于多模态输入，如图像块、文档块等

**可选参数**：
- `temperature`、`top_p`、`top_k`：采样控制（部分新模型如 Claude 4.7+ 已不支持这些参数）
- `stop_sequences`：停止序列
- `tools`：工具定义
- `stream`：流式返回

### 响应格式

```json
{
  "id": "msg_xxx",
  "role": "assistant",
  "content": [
    {"type": "text", "text": "Hello!"}
  ],
  "stop_reason": "end_turn",
  "usage": {"input_tokens": 10, "output_tokens": 5}
}
```

文本位于 `content[0].text`。`stop_reason` 可能为 `end_turn`、`max_tokens`、`tool_use` 等。


## Google Gemini GenerateContent API

```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent
```


## 厂商支持

| 厂商                | 核心端点                                                                                                | 协议风格            | 与 OpenAI Chat Completions 的关系 |
| ----------------- | --------------------------------------------------------------------------------------------------- | --------------- | ----------------------------- |
| **OpenAI**        | `/v1/chat/completions`（维护）  <br>`/v1/responses`（推荐）                                                 | 对话式 → Agent 式   | 原生                            |
| **Anthropic**     | `/v1/messages`                                                                                      | 独立的 Messages 格式 | 完全不同（schema 不兼容）              |
| **Google Gemini** | `:generateContent`（原生）  <br>`/v1beta/openai/chat/completions`（兼容层）                                  | 原生 + OpenAI 兼容  | 提供兼容端点，改 3 行代码即可迁移            |
| **DeepSeek**      | `https://api.deepseek.com/chat/completions`  <br>`https://api.deepseek.com/anthropic`（Anthropic 兼容） | 双兼容             | 同时兼容 OpenAI 和 Anthropic       |
| **Mistral**       | `https://api.mistral.ai/v1/chat/completions`                                                        | OpenAI 兼容       | 请求格式“与 OpenAI 标准非常相似”         |
| **Cohere**        | `POST /v1/chat`                                                                                     | 原生 Chat v2      | 提供 OpenAI 兼容客户端接入方式           |
| **Meta Llama**    | `https://api.llama.com/v1`                                                                          | 原生 API          | 非 OpenAI 兼容                   |
| **AI21**          | `https://api.ai21.com/studio/v1/...`                                                                | 原生 Studio API   | 非 OpenAI 兼容                   |
