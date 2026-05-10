# `app/exceptions.py` — 异常定义

## 文件位置
`app/exceptions.py`

## 核心作用
定义 OpenManus 项目中的自定义异常类。

## 异常层次

```
Exception
  ├─ ToolError            # 工具执行错误
  └─ OpenManusError       # 所有 OpenManus 异常基类
       └─ TokenLimitExceeded  # Token 限制超限
```

### `ToolError(Exception)`
工具执行过程中遇到错误时抛出。包含 `message` 属性。

### `OpenManusError(Exception)`
所有 OpenManus 自定义异常的基类。

### `TokenLimitExceeded(OpenManusError)`
当 LLM 请求的 token 数超过 `max_input_tokens` 限制时抛出。在 `app/llm.py` 中被捕获并重新抛出（不重试）。

## 调用关系
- `ToolError` 被 `app/tool/base.py` 的 `BaseTool.run()` 使用
- `TokenLimitExceeded` 被 `app/llm.py` 的 `LLM.ask()`、`LLM.ask_tool()` 抛出，并在 `@retry` 装饰器的异常类型列表中排除（不重试）
