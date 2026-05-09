# `app/sandbox/__init__.py` — Docker 沙箱模块入口

## 文件位置
`app/sandbox/__init__.py`

## 核心作用
Docker 沙箱模块的统一导出入口，封装了安全的容器化执行环境，提供资源限制和隔离能力。

## 导出内容

| 导出名 | 来源 | 说明 |
|--------|------|------|
| `BaseSandboxClient` | `client.py` | 沙箱客户端抽象基类 |
| `LocalSandboxClient` | `client.py` | 本地沙箱客户端实现 |
| `create_sandbox_client` | `client.py` | 沙箱客户端工厂函数 |
| `SandboxError` | `core/exceptions.py` | 沙箱异常基类 |
| `SandboxResourceError` | `core/exceptions.py` | 资源相关异常 |
| `SandboxTimeoutError` | `core/exceptions.py` | 超时异常 |
| `SandboxManager` | `core/manager.py` | 沙箱管理器 |
| `DockerSandbox` | `core/sandbox.py` | Docker 沙箱环境实现 |

## `__all__`
```python
__all__ = [
    "DockerSandbox", "SandboxManager", "BaseSandboxClient",
    "LocalSandboxClient", "create_sandbox_client",
    "SandboxError", "SandboxTimeoutError", "SandboxResourceError",
]
```

## 设计说明
- `app/sandbox/` 基于 **Docker** 实现本地容器化沙箱
- `app/daytona/` 基于 **Daytona** 实现远程云沙箱
- 两者是并列的两种沙箱实现方案，分别服务于不同的部署场景
