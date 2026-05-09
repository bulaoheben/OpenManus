# `app/sandbox/core/manager.py` — Docker 沙箱管理器

## 文件位置
`app/sandbox/core/manager.py`

## 核心作用
管理多个 `DockerSandbox` 实例的生命周期，提供并发控制、空闲自动清理和资源限制功能。

## 类结构

### `SandboxManager`

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `max_sandboxes` | `int` | `100` | 最大沙箱数量 |
| `idle_timeout` | `int` | `3600` | 空闲超时（秒） |
| `cleanup_interval` | `int` | `300` | 清理检查间隔（秒） |
| `_client` | `docker.Client` | `docker.from_env()` | Docker 客户端 |
| `_sandboxes` | `Dict[str, DockerSandbox]` | `{}` | 活跃沙箱映射 |
| `_last_used` | `Dict[str, float]` | `{}` | 最后使用时间记录 |
| `_locks` | `Dict[str, asyncio.Lock]` | `{}` | 沙箱级别并发锁 |
| `_global_lock` | `asyncio.Lock` | 新建 | 全局操作锁 |
| `_active_operations` | `Set[str]` | `set()` | 活跃操作跟踪 |
| `_cleanup_task` | `Optional[asyncio.Task]` | `None` | 自动清理任务 |
| `_is_shutting_down` | `bool` | `False` | 关闭标记 |

### 关键方法

**`ensure_image(image)` — 确保镜像可用**
检查 Docker 镜像是否存在，不存在则自动拉取。返回 `bool`。

**`sandbox_operation(sandbox_id)` — 操作上下文管理器**
`@asynccontextmanager`，提供沙箱级别并发控制：
1. 获取沙箱对应的 `asyncio.Lock`
2. 检查沙箱是否存在
3. 记录活跃操作和最后使用时间
4. `yield` 沙箱实例
5. 操作完成后移除活跃操作标记

**`create_sandbox(config, volume_bindings)` — 创建沙箱**
```python
async def create_sandbox(...) -> str  # 返回 sandbox_id
```
1. 全局锁保护，检查上限
2. 确保镜像可用
3. 创建 `DockerSandbox` 实例并调用 `create()`
4. 注册到内部映射
5. 失败时自动清理

**`get_sandbox(sandbox_id)` — 获取沙箱**
通过 `sandbox_operation` 上下文获取沙箱实例。

**`start_cleanup_task()` — 启动自动清理**
创建后台 `asyncio.Task`，每隔 `cleanup_interval` 调用 `_cleanup_idle_sandboxes()`。

**`_cleanup_idle_sandboxes()` — 清理空闲沙箱**
检查所有沙箱的最后使用时间，超过 `idle_timeout` 且无活跃操作的沙箱列入清理列表并删除。

**`cleanup()` — 全局清理**
1. 设置 `_is_shutting_down = True`
2. 取消清理任务
3. 并发删除所有沙箱（30 秒超时）
4. 清空所有内部映射

**`_safe_delete_sandbox(sandbox_id)` — 安全删除沙箱**
等待活跃操作完成（最多 5 秒），然后调用 `sandbox.cleanup()` 并移除记录。

**`delete_sandbox(sandbox_id)` — 删除指定沙箱**

**`get_stats()` — 获取统计信息**
返回 `total_sandboxes`、`active_operations`、`max_sandboxes` 等状态。

### 上下文管理器支持
实现了 `__aenter__` 和 `__aexit__`，支持 `async with SandboxManager() as manager:` 用法。

## 并发设计
- **全局锁** (`_global_lock`)：保护创建/删除等全局操作
- **沙箱级别锁** (`_locks[sandbox_id]`)：保护单个沙箱的操作
- **活跃操作集** (`_active_operations`)：跟踪正在进行的操作，防止清理冲突

## 调用关系
- 使用 `docker` 库的 API（`docker.from_env()`、`images`、`containers`）
- 使用 `app/sandbox/core/sandbox.py` 的 `DockerSandbox`
- 使用 `app/config.py` 的 `SandboxSettings`
