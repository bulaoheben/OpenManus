# `app/schema.py` — 数据模型定义

## 文件位置
`app/schema.py`

## 核心作用
定义项目中的核心数据模型，包括消息角色、工具调用、Agent 状态和对话内存管理。

## 枚举类型

### `Role(str, Enum)`
消息角色：
| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `SYSTEM` | `"system"` | 系统提示 |
| `USER` | `"user"` | 用户消息 |
| `ASSISTANT` | `"assistant"` | AI 回复 |
| `TOOL` | `"tool"` | 工具返回结果 |

```python
ROLE_VALUES = ("system", "user", "assistant", "tool")
ROLE_TYPE = Literal["system", "user", "assistant", "tool"]
```

### `ToolChoice(str, Enum)`
工具选择策略：
| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `NONE` | `"none"` | 不使用工具 |
| `AUTO` | `"auto"` | 自动选择 |
| `REQUIRED` | `"required"` | 强制使用工具 |

```python
TOOL_CHOICE_VALUES = ("none", "auto", "required")
TOOL_CHOICE_TYPE = Literal["none", "auto", "required"]
```

### `AgentState(str, Enum)`
Agent 执行状态：
| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IDLE` | `"IDLE"` | 空闲 |
| `RUNNING` | `"RUNNING"` | 运行中 |
| `FINISHED` | `"FINISHED"` | 完成 |
| `ERROR` | `"ERROR"` | 错误 |

## 数据模型

### `Function(BaseModel)`
工具函数定义：
```python
name: str          # 函数名
arguments: str     # JSON 格式的参数
```

### `ToolCall(BaseModel)`
工具调用记录：
```python
id: str                    # 调用 ID
type: str = "function"     # 类型
function: Function         # 函数信息
```

### `Message(BaseModel)`
对话消息的通用表示：

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `role` | `ROLE_TYPE` | 必填 | 消息角色 |
| `content` | `Optional[str]` | `None` | 消息内容 |
| `tool_calls` | `Optional[List[ToolCall]]` | `None` | 工具调用列表 |
| `name` | `Optional[str]` | `None` | 工具名称 |
| `tool_call_id` | `Optional[str]` | `None` | 工具调用 ID |
| `base64_image` | `Optional[str]` | `None` | Base64 编码图片 |

**运算符重载：**
- `Message + Message` → `[Message, Message]`
- `Message + list` → `[Message] + list`
- `list + Message` → `list + [Message]`

**工厂方法：**

| 方法 | 说明 |
|------|------|
| `Message.user_message(content, base64_image)` | 创建用户消息 |
| `Message.system_message(content)` | 创建系统消息 |
| `Message.assistant_message(content)` | 创建 AI 消息 |
| `Message.tool_message(content, name, tool_call_id)` | 创建工具结果消息 |
| `Message.from_tool_calls(tool_calls, content)` | 从原始 tool calls 创建消息 |

**to_dict()：**
将 Message 转换为字典格式，仅包含非 `None` 字段。

### `Memory(BaseModel)`
对话记忆管理：

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `messages` | `List[Message]` | `[]` | 消息列表 |
| `max_messages` | `int` | `100` | 最大消息数 |

**方法：**
| 方法 | 说明 |
|------|------|
| `add_message(message)` | 添加单条消息，超出上限则截断 |
| `add_messages(messages)` | 添加多条消息 |
| `clear()` | 清空所有消息 |
| `get_recent_messages(n)` | 获取最近 n 条消息 |
| `to_dict_list()` | 转为字典列表 |

## 调用关系
- 基于 `pydantic.BaseModel` 进行数据验证
- 被 `app/llm.py` 的 `LLM` 类广泛使用
- 被 `app/agent/` 下所有 Agent 用于消息处理
- `Memory` 被 Agent 用于管理对话历史
- `AgentState` 被 `BaseAgent` 用于状态管理
- `ToolChoice` 被 `ToolCallAgent` 用于控制工具调用策略
