# `app/tool/create_chat_completion.py` — 结构化输出工具

## 文件位置
`app/tool/create_chat_completion.py`

## 核心作用
提供结构化输出的能力，支持将 LLM 的响应强制转换为指定类型（str、Pydantic Model、List、Dict、Union）。

## 类结构

### CreateChatCompletion(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"create_chat_completion"` |
| `description` | 创建带有指定输出格式的结构化补全 |

| 属性 | 说明 |
|------|------|
| `type_mapping` | Python 类型到 JSON schema 类型的映射 |
| `response_type` | 目标输出类型 |
| `required` | 必需的字段列表 |

### __init__(response_type)
接收 `response_type` 参数，据此构建对应的 `parameters` schema。

### _build_parameters()
根据 `response_type` 生成不同的 JSON schema：
- `str`：简单的字符串响应
- Pydantic `BaseModel` 子类：调用 `model_json_schema()` 生成
- `List[T]`：数组类型
- `Dict[K, V]`：对象类型
- `Union[...]`：anyOf 类型

### execute(**kwargs)
根据 `response_type` 进行类型转换：
- `str`：直接返回字符串
- `BaseModel`：用 kwargs 构造模型实例
- `list`/`dict`：直接返回（假设已为正确格式）
- 其他类型：尝试构造函数转换

## 设计说明
- 主要用于 Agent 需要 LLM 返回特定结构而非自由文本的场景
- 参数 schema 在 `__init__` 时动态生成，而非静态定义
- `_create_type_schema` 使用 `typing.get_origin()` 和 `typing.get_args()` 解析泛型类型
