# `app/tool/base.py` — 工具基类与结果模型

[toc]

## 文件位置

`app/tool/base.py`

## 核心作用
定义整个工具系统的基类和数据结构，是所有工具类的父类。提供以下基础设施：
- `ToolResult`：工具执行结果的数据模型
- `BaseTool`：所有工具的抽象基类
- `CLIResult` / `ToolFailure`：结果子类

---

## 类结构

### ToolResult (Pydantic BaseModel)
工具执行结果的统一封装，包含 4 个字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | `Any` | 成功时的输出内容 |
| `error` | `Optional[str]` | 失败时的错误信息 |
| `base64_image` | `Optional[str]` | 可选的 base64 编码图片（浏览器截图等） |
| `system` | `Optional[str]` | 系统级消息 |

实现了 `__bool__`（任一字段有值则为 True）、`__add__`（两个 ToolResult 合并，但 base64_image 不能合并）、`__str__`（优先显示 error，否则显示 output）、`replace()`（创建替换部分字段的新对象）。



|             写法             |           含义            |
| :--------------------------: | :-----------------------: |
|         `name: str`          | 必须是字符串，不能为 None |
|    `name: Optional[str]`     |   可以是字符串 或 None    |
| `name: Optional[str] = None` |   可选字符串，默认是空    |

`Field(default=None)` = 默认值为空，可选字段

### BaseTool (ABC + BaseModel)
所有工具的抽象基类，核心接口：

| 方法 | 说明 |
|------|------|
| `__call__(**kwargs)` | 将工具实例作为异步函数调用，委托给 `execute() `       (__call__ 是 Python 的魔法方法（内置方法）作用：让一个类的实例，像函数一样被调用。) |
| `execute(**kwargs)` | **抽象方法**，子类必须实现 |
| `to_param()` | 转换为 OpenAI function calling 格式的字典（`type` + `function` 含 name/description/parameters） |
| `success_response(data)` | 创建成功 ToolResult，自动将 dict 序列化为 JSON |
| `fail_response(msg)` | 创建失败 ToolResult |

Pydantic 配置：`arbitrary_types_allowed = True`（修复了旧版 `underscore_attrs_are_private` 已被移除的问题）。



### CLIResult(ToolResult) / ToolFailure(ToolResult)
- `CLIResult`：可渲染为命令行输出的结果
- `ToolFailure`：明确表示失败的结果

## 设计模式
- **模板方法模式**：`__call__` 固定流程，`execute` 由子类实现具体逻辑
- **统一结果格式**：所有工具返回值统一为 `ToolResult`，简化 Agent 结果处理
- **序列化兼容**：`to_param()` 输出兼容 OpenAI API 的 tool calling 格式

## 调用关系
- `app/tool/__init__.py` 导出 `BaseTool` 供外部使用
- 所有具体工具类（Bash、BrowserUseTool、WebSearch 等）继承 `BaseTool`
- `ToolCollection` 通过 `to_param()` 获取所有工具的 schema 列表，通过 `execute()` 分发调用
- Agent 的 `execute_tool()` 接收工具执行结果（ToolResult）并返回给 LLM
