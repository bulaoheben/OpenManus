# `app/llm.py` — LLM 集成

[toc]

## 文件位置

`app/llm.py`

## 核心作用
提供统一的 LLM 调用接口，支持 OpenAI 兼容 API、Azure OpenAI 和 AWS Bedrock，包含 Token 计数、重试机制、多模态支持和工具调用功能。

## 常量

```python
REASONING_MODELS = ["o1", "o3-mini"]           # 推理模型列表（不使用 temperature 参数）
MULTIMODAL_MODELS = [                           # 多模态模型列表
    "gpt-4-vision-preview", "gpt-4o", "gpt-4o-mini",
    "claude-3-opus-20240229", "claude-3-sonnet-20240229", "claude-3-haiku-20240307",
]
```

#### 一、先看懂配置里的两个分组

1）REASONING_MODELS = ["o1", "o3-mini"]

**专门用于深度思考、数学、代码、逻辑推理**

特点：**纯文本、不支持看图、不使用 temperature**（输出更稳定、更严谨）

2）MULTIMODAL_MODELS = [...]

**多模态模型，能看图片 + 文本一起理解**

特点：**支持识图、截图分析、图表、OCR、图文问答**

#### 二、你项目里为什么这么分？

- **REASONING_MODELS**：做**思考、决策、规划**
- **MULTIMODAL_MODELS**：做**看图、理解界面、分析截图**

你的框架是：**用推理模型当大脑，用多模态模型处理视觉输入**

这是 AI Agent（如 OpenManus、OpenClaw）的标准架构。



## 类结构

### `TokenCounter`
计算消息和图像的 token 消耗。

**常量：**
| 常量 | 值 | 说明 |
|------|-----|------|
| `BASE_MESSAGE_TOKENS` | `4` | 每条消息基础 token |
| `FORMAT_TOKENS` | `2` | 格式 token |
| `LOW_DETAIL_IMAGE_TOKENS` | `85` | 低细节图片 token |
| `HIGH_DETAIL_TILE_TOKENS` | `170` | 高细节每 tile token |
| `MAX_SIZE` | `2048` | 图片缩放最大尺寸 |
| `HIGH_DETAIL_TARGET_SHORT_SIDE` | `768` | 高细节目标短边 |
| `TILE_SIZE` | `512` | 每个 tile 尺寸 |

**关键方法：**
- `count_text(text)` — 使用 tiktoken 计算文本 token
- `count_image(image_item)` — 根据 detail 等级和尺寸计算图片 token
  - `low`：固定 85 tokens
  - `high`/`medium`：缩放到 2048x2048 → 短边缩放到 768px → 按 512px tile 计算（170 tokens/tile）+ 85 tokens
- `count_content(content)` — 计算消息内容 token（支持文本和图片混合）
- `count_tool_calls(tool_calls)` — 计算工具调用 token
- `count_message_tokens(messages)` — 计算整个消息列表总 token 数

### `LLM`
核心 LLM 接口类，使用单例模式管理不同配置的实例。

**实例管理：**
```python
LLM._instances: Dict[str, "LLM"] = {}  # 按 config_name 缓存
```
`__new__` 方法确保同一 `config_name` 只创建一个实例。

**初始化（`__init__`）：**
1. 从 `config.llm` 获取指定 `config_name` 的配置
2. 初始化 tiktoken tokenizer
3. 根据 `api_type` 选择客户端：
   - `"azure"` → `AsyncAzureOpenAI`
   - `"aws"` → `BedrockClient`（来自 `app/bedrock.py`）
   - 其他 → `AsyncOpenAI`
4. 初始化 `TokenCounter`

**属性：**

| 属性 | 类型 | 说明 |
|------|------|------|
| `model` | `str` | 模型名称 |
| `max_tokens` | `int` | 最大输出 token |
| `temperature` | `float` | 采样温度 |
| `total_input_tokens` | `int` | 累计输入 token |
| `total_completion_tokens` | `int` | 累计输出 token |
| `max_input_tokens` | `Optional[int]` | 输入 token 上限 |
| `client` | `AsyncOpenAI / BedrockClient` | API 客户端 |
| `token_counter` | `TokenCounter` | Token 计数器 |

**核心方法：**

#### `format_messages(messages, supports_images)`
格式化消息用于 LLM 请求：
1. 将 `Message` 对象转为字典
2. 处理 `base64_image` 字段（支持图片的模型 → 转为 `image_url` 内容块；不支持图片的模型 → 移除字段）
3. 验证 role 有效性

#### `ask(messages, system_msgs, stream, temperature)` → `str`
发送文本请求给 LLM，返回响应文本：
1. 检查模型是否支持多模态
2. 格式化消息
3. 计算并检查 token 限制
4. 清理 surrogate 字符（防止 UnicodeEncodeError）
5. 流式或非流式调用
6. 更新 token 计数

这段代码是一个**异步调用 LLM（大模型）的核心方法 `ask()`**

作用：**给模型发消息 → 模型返回回答 → 支持流式输出 / 非流式 → 处理 Tokens → 处理错误**

**参数通俗解释**

- `messages`：你要问 AI 的对话记录
- `system_msgs`：系统提示（比如 “你是一个编程助手”）
- `stream`：True = 一边生成一边输出（打字机）
- `temperature`：越高回答越天马行空，越低越严谨



#### `ask_with_images(messages, images, system_msgs, stream, temperature)` → `str`
发送图文请求给 LLM：
1. 验证模型支持多模态
2. 确保最后一条消息是 user 角色
3. 将图片附加到最后一条消息的 content 中
4. 其余与 `ask()` 类似

#### `ask_tool(messages, system_msgs, timeout, tools, tool_choice, temperature)` → `ChatCompletionMessage`
发送工具调用请求给 LLM：
1. 验证 `tool_choice` 合法性
2. 格式化消息
3. 计算 token（含工具描述）
4. 清理 surrogate 字符
5. 始终使用非流式模式
6. 返回包含 tool_calls 的响应消息

**重试机制：**
所有 `ask*` 方法使用 `@retry`（tenacity）：
```python
@retry(
    wait=wait_random_exponential(min=1, max=60),  # 指数退避 1~60 秒
    stop=stop_after_attempt(6),                    # 最多重试 6 次
    retry=retry_if_exception_type((OpenAIError, Exception, ValueError)),  # 不重试 TokenLimitExceeded
)
```

**工具函数：**
`_clean_surrogates(obj)` — 递归清理字典/列表/字符串中的 surrogate 字符，防止 `httpx` 的 `UnicodeEncodeError`。

## 调用关系
- 使用 `openai` 库（`AsyncOpenAI`、`AsyncAzureOpenAI`）
- 使用 `tiktoken` 进行 token 计算
- 使用 `tenacity` 实现重试
- 使用 `app/bedrock.py` 的 `BedrockClient`（AWS Bedrock 支持）
- 使用 `app/config.py` 的 `config` 和 `LLMSettings`
- 使用 `app/exceptions.py` 的 `TokenLimitExceeded`
- 使用 `app/schema.py` 的 `Message`、`ToolChoice` 等
- 被 `app/agent/toolcall.py` 的 `ToolCallAgent` 调用
- 被 `app/flow/planning.py` 的 `PlanningFlow` 调用
