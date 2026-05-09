# `app/sandbox/core/exceptions.py` — 沙箱异常定义

## 文件位置
`app/sandbox/core/exceptions.py`

## 核心作用
定义沙箱系统的自定义异常类，实现结构化的错误处理。

## 异常层次

```
Exception
  └─ SandboxError              # 沙箱异常基类
       ├─ SandboxTimeoutError   # 操作超时
       └─ SandboxResourceError  # 资源相关错误
```

### `SandboxError(Exception)`
所有沙箱相关异常的基类，继承自 `Exception`。

### `SandboxTimeoutError(SandboxError)`
沙箱操作超时时抛出（如命令执行超过 `timeout` 限制）。

### `SandboxResourceError(SandboxError)`
资源相关错误时抛出（如资源限制、分配失败等）。

## 调用关系
- 被 `app/sandbox/core/terminal.py` 的 `DockerSession.execute()` 使用
- 被 `app/sandbox/__init__.py` 导出
- 被外部模块通过 `from app.sandbox import SandboxError` 引用
