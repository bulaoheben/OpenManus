# `app/tool/file_operators.py` — 文件操作接口与实现

## 文件位置
`app/tool/file_operators.py`

## 核心作用
定义文件操作的抽象接口（`FileOperator` 协议），并提供本地和沙盒两种环境的实现。

## 类结构

### FileOperator (Protocol)
使用 Python `typing.Protocol` 定义的接口规范：

```python
@runtime_checkable
class FileOperator(Protocol):
    async def read_file(path) -> str
    async def write_file(path, content) -> None
    async def is_directory(path) -> bool
    async def exists(path) -> bool
    async def run_command(cmd, timeout=120.0) -> Tuple[int, str, str]
```

`@runtime_checkable` 允许使用 `isinstance()` 检查。

### LocalFileOperator(FileOperator)
本地文件系统实现：

| 方法 | 实现 |
|------|------|
| `read_file` | `Path(path).read_text(encoding="utf-8")` |
| `write_file` | `Path(path).write_text(content, encoding="utf-8")` |
| `is_directory` | `Path(path).is_dir()` |
| `exists` | `Path(path).exists()` |
| `run_command` | `asyncio.create_subprocess_shell` + `asyncio.wait_for` |

### SandboxFileOperator(FileOperator)
沙盒环境实现（通过 Daytona sandbox client）：

| 领域 | 说明 |
|------|------|
| `read_file` | `sandbox_client.read_file(str(path))` |
| `write_file` | `sandbox_client.write_file(str(path), content)` |
| `is_directory` | 运行 `test -d {path}` 检查 |
| `exists` | 运行 `test -e {path}` 检查 |
| `run_command` | `sandbox_client.run_command(cmd, timeout)` |

自动调用 `_ensure_sandbox_initialized()` 初始化沙盒连接。

## 设计模式
- **协议（Protocol）接口**：使用鸭子类型而非抽象继承，降低耦合
- **策略模式**：`StrReplaceEditor` 根据 `config.sandbox.use_sandbox` 选择实现
- **PathLike 类型别名**：`Union[str, Path]`

## 调用关系
- 被 `StrReplaceEditor`（`str_replace_editor.py`）使用
- `SandboxFileOperator` 依赖 `app.sandbox.client.SANDBOX_CLIENT`
- `LocalFileOperator` 仅依赖 pathlib 和 asyncio
