# `app/bedrock.py` — AWS Bedrock 客户端

## 文件位置
`app/bedrock.py`

## 核心作用
封装 Amazon Bedrock API，提供与 OpenAI 兼容的接口格式，使 LLM 模块可以通过统一的方式调用 Bedrock 模型。

## 全局变量
```python
CURRENT_TOOLUSE_ID = None
```
跨函数调用跟踪当前 tool use ID。

## 辅助类

### `OpenAIResponse`
将嵌套字典/列表递归转换为属性访问方式的对象，提供 `model_dump()` 方法（追加 `created_at` 时间戳）。

## 客户端结构

```
BedrockClient
  └─ Chat
       └─ ChatCompletions
```

### `BedrockClient`
初始化 `boto3.client("bedrock-runtime")`，创建 `Chat` 实例。

### `Chat`
持有 `ChatCompletions` 实例的中间层。

### `ChatCompletions`
核心类，负责消息格式转换和 API 调用。

## 格式转换方法

### `_convert_openai_tools_to_bedrock_format(tools)`
将 OpenAI function calling 格式转换为 Bedrock tool 格式：

| OpenAI 字段 | Bedrock 字段 |
|-------------|-------------|
| `function.name` | `toolSpec.name` |
| `function.description` | `toolSpec.description` |
| `function.parameters.properties` | `toolSpec.inputSchema.json.properties` |
| `function.parameters.required` | `toolSpec.inputSchema.json.required` |

### `_convert_openai_messages_to_bedrock_format(messages)`
将 OpenAI 消息格式转换为 Bedrock 格式：

| OpenAI role | Bedrock 处理 |
|-------------|-------------|
| `system` | 提取为独立的 `system_prompt` |
| `user` | 直接映射，content 转为 `[{"text": ...}]` |
| `assistant` | 映射为 assistant，含 tool_calls 时加 `toolUse` 块 |
| `tool` | 映射为 `user` 角色的 `toolResult` 块 |

### `_convert_bedrock_response_to_openai_format(bedrock_response)`
将 Bedrock 响应转换为 OpenAI 兼容格式：
- 提取 text 内容
- 提取 toolUse 信息转为 `tool_calls` 数组
- 构建标准 OpenAI chat completion 响应结构

## API 调用方法

### `create(model, messages, max_tokens, temperature, stream, tools, tool_choice)`
入口方法：
1. 调用 `_convert_openai_tools_to_bedrock_format()` 转换工具
2. 根据 `stream` 参数选择流式或非流式调用

### `_invoke_bedrock(...)` — 非流式
调用 `client.converse()`，返回转换后的 OpenAI 格式响应。

### `_invoke_bedrock_stream(...)` — 流式
调用 `client.converse_stream()`，逐事件处理流：
- `messageStart`：记录角色
- `contentBlockDelta.text`：累积文本输出
- `contentBlockStart.toolUse`：开始工具调用
- `contentBlockDelta.toolUse`：累积工具调用输入
- `contentBlockStop`：完成内容块

## 调用关系
- 使用 `boto3` 库连接 AWS Bedrock
- 被 `app/llm.py` 的 `LLM` 类在 `api_type == "aws"` 时使用
- 作为 `AsyncOpenAI` 的替代客户端，提供相同的接口方法名
