# `app/sandbox/core/sandbox.py` — Docker 沙箱环境

## 文件位置
`app/sandbox/core/sandbox.py`

## 核心作用
提供基于 Docker 容器的沙箱执行环境，包含资源限制、文件操作和命令执行能力。

## 类结构

### `DockerSandbox`

| 属性 | 类型 | 说明 |
|------|------|------|
| `config` | `SandboxSettings` | 沙箱配置 |
| `volume_bindings` | `Dict[str, str]` | 卷映射配置 |
| `client` | `docker.Client` | Docker 客户端 |
| `container` | `Optional[Container]` | Docker 容器实例 |
| `terminal` | `Optional[AsyncDockerizedTerminal]` | 异步终端接口 |

### 初始化流程

**`__init__(config, volume_bindings)`**
- `config` 为 `None` 时使用默认 `SandboxSettings()`
- 初始化 Docker 客户端

**`create()` — 创建并启动容器**
1. 配置 `host_config`（内存限制、CPU 限制、网络模式、卷绑定）
2. 生成唯一容器名 `sandbox_{8位hex}`
3. 使用 `tail -f /dev/null` 作为持久化命令
4. 设置 TTY、detach 模式
5. 启动容器
6. 初始化 `AsyncDockerizedTerminal`
7. 失败时调用 `cleanup()` 确保资源释放

### 资源限制
| 参数 | 来源 | 说明 |
|------|------|------|
| 内存限制 | `config.memory_limit` | Docker `mem_limit` |
| CPU 限制 | `config.cpu_limit` | `cpu_quota = 100000 * 限制值` |
| 网络模式 | `config.network_enabled` | 启用 → `bridge`，禁用 → `none` |
| 工作目录 | `config.work_dir` | 容器内工作目录 |

### 文件操作

**`read_file(path)` — 读取文件**
1. 路径安全检查（防止路径遍历）
2. `container.get_archive()` 获取 tar 流
3. `_read_from_tar()` 从 tar 中提取文件内容

**`write_file(path, content)` — 写入文件**
1. 路径安全检查
2. `run_command(f"mkdir -p {parent_dir}")` 创建父目录
3. `_create_tar_stream()` 创建 tar 流
4. `container.put_archive()` 上传到容器

**`copy_from(src_path, dst_path)` — 从容器复制到本地**
1. 确保目标父目录存在
2. `container.get_archive()` 获取 tar 流
3. 写入临时目录并解压
4. 支持单个文件和目录两种模式

**`copy_to(src_path, dst_path)` — 从本地复制到容器**
1. 检查源文件存在
2. 创建容器目标目录
3. 创建 tar 包（支持文件/目录）
4. `container.put_archive()` 上传
5. 通过 `test -e` 验证文件创建成功

### 辅助方法

**`_safe_resolve_path(path)` — 安全路径解析**
- 检测路径遍历攻击（`..` 检查）
- 相对路径拼接 `work_dir`，绝对路径直接使用

**`_prepare_volume_bindings()` — 准备卷绑定**
- 自动在临时目录创建工作目录映射
- 合并自定义卷绑定

**`_ensure_host_dir(path)` — 确保主机目录存在**
在系统临时目录下创建 `sandbox_{basename}_{随机4位hex}` 目录。

**`_create_tar_stream(name, content)` — 创建 tar 流**
静态方法。将文件内容打包成 `io.BytesIO` tar 流。

**`_read_from_tar(tar_stream)` — 从 tar 流读取**
静态方法。将 tar 流写入临时文件后解压读取。

### 生命周期

**`cleanup()` — 清理资源**
1. 关闭 terminal（`AsyncDockerizedTerminal.close()`）
2. 停止容器（5 秒超时）
3. 强制移除容器
4. 收集所有错误但不中断清理流程

### 上下文管理器
```python
async with DockerSandbox() as sandbox:
    await sandbox.run_command("echo hello")
```
`__aenter__` → `create()`，`__aexit__` → `cleanup()`。

## 调用关系
- 使用 `docker` 库（`docker.from_env()`、`Container`、`NotFound`）
- 使用 `app/sandbox/core/terminal.py` 的 `AsyncDockerizedTerminal`
- 使用 `app/sandbox/core/exceptions.py` 的 `SandboxTimeoutError`
- 使用 `app/config.py` 的 `SandboxSettings`
- 被 `app/sandbox/core/manager.py` 的 `SandboxManager` 管理
- 被 `app/sandbox/client.py` 的 `LocalSandboxClient` 调用
