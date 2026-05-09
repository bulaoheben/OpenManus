# `app/sandbox/client.py` — 沙箱客户端

## 文件位置
`app/sandbox/client.py`

## 核心作用
定义沙箱客户端的抽象接口（`BaseSandboxClient`）和本地 Docker 实现（`LocalSandboxClient`），提供统一的沙箱操作契约。

## 协议类

### `SandboxFileOperations(Protocol)`
文件操作协议，定义 4 个异步方法签名：
- `copy_from(container_path, local_path)` — 从容器复制到本地
- `copy_to(local_path, container_path)` — 从本地复制到容器
- `read_file(path)` — 读取容器文件
- `write_file(path, content)` — 写入容器文件

## 抽象基类

### `BaseSandboxClient(ABC)`
定义 7 个抽象方法：

| 方法 | 说明 |
|------|------|
| `create(config, volume_bindings)` | 创建沙箱 |
| `run_command(command, timeout)` | 执行命令 |
| `copy_from(container_path, local_path)` | 从容器复制文件 |
| `copy_to(local_path, container_path)` | 复制文件到容器 |
| `read_file(path)` | 读取文件 |
| `write_file(path, content)` | 写入文件 |
| `cleanup()` | 清理资源 |

## 实现类

### `LocalSandboxClient(BaseSandboxClient)`
本地 Docker 沙箱客户端实现。

| 属性 | 类型 | 说明 |
|------|------|------|
| `sandbox` | `Optional[DockerSandbox]` | Docker 沙箱实例 |

所有方法均委托给 `DockerSandbox` 实例，并在操作前检查 `sandbox` 是否已初始化。

### `create_sandbox_client()` — 工厂函数
返回 `LocalSandboxClient()` 实例。

### `SANDBOX_CLIENT` — 模块单例
```python
SANDBOX_CLIENT = create_sandbox_client()
```
模块级沙箱客户端单例，供全局使用。

## 调用关系
- `BaseSandboxClient` 定义接口契约
- `LocalSandboxClient` 使用 `app/sandbox/core/sandbox.py` 的 `DockerSandbox`
- 被 `app/sandbox/__init__.py` 导出
