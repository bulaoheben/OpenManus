# `app/sandbox/core/terminal.py` — Docker 异步终端

## 文件位置
`app/sandbox/core/terminal.py`

## 核心作用
为 Docker 容器提供异步交互式终端能力，支持命令执行、超时控制、输出解析和命令注入防护。

## 类结构

### `DockerSession`
底层的 Docker 交互式会话，通过 Docker API 创建 socket 连接与容器通信。

| 属性 | 类型 | 说明 |
|------|------|------|
| `api` | `APIClient` | Docker API 客户端 |
| `container_id` | `str` | 容器 ID |
| `exec_id` | `str` | 执行实例 ID |
| `socket` | `socket` | 与容器的 socket 连接 |

**`create(working_dir, env_vars)` — 创建交互式会话**
1. 通过 `exec_create()` 创建 bash 执行实例（`bash --norc --noprofile`）
2. 通过 `exec_start(socket=True)` 获取 socket 连接
3. 设置 socket 为非阻塞模式
4. 读取直到出现 `$ ` 提示符

**`close()` — 清理会话**
发送 `exit` 命令 → 关闭 socket → 检查 exec 实例状态。所有步骤都有异常防护，确保清理不中断。

**`_read_until_prompt()` — 读取直到提示符**
循环读取 socket 数据，直到缓冲区包含 `$ ` 提示符。

**`execute(command, timeout)` — 执行命令**
1. 调用 `_sanitize_command()` 进行命令注入检查
2. 发送 `{command}\necho $?\n` 到 socket
3. 异步读取输出，过滤掉回显的命令行和退出码
4. 支持超时控制
5. 返回清理后的输出文本

**`_sanitize_command(command)` — 命令注入防护**
检查并阻止危险命令：
- `rm -rf /`、`rm -rf /*`
- `mkfs`、`dd if=/dev/zero`
- `fork 炸弹`、`chmod -R 777 /`、`chown -R`

### `AsyncDockerizedTerminal`
高级异步终端接口，封装 `DockerSession`。

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `client` | `docker.Client` | `docker.from_env()` | Docker 客户端 |
| `container` | `Container` | 根据 ID 获取 | 容器实例 |
| `working_dir` | `str` | `/workspace` | 工作目录 |
| `env_vars` | `Dict[str, str]` | `{}` | 环境变量 |
| `default_timeout` | `int` | `60` | 默认超时（秒） |
| `session` | `DockerSession` | `None` | Docker 会话 |

**`init()` — 初始化终端**
1. 确保工作目录存在（`_ensure_workdir()`）
2. 创建并初始化 `DockerSession`

**`_ensure_workdir()` — 确保工作目录存在**
通过 `exec_run(f"mkdir -p {working_dir}")` 创建目录。

**`_exec_simple(cmd)` — 简单命令执行**
使用 `container.exec_run()` 执行一次性命令，返回 `(exit_code, output)`。

**`run_command(cmd, timeout)` — 运行命令**
委托给 `session.execute()`，使用默认或指定的超时时间。

**`close()` — 关闭终端**
关闭 `DockerSession`。

### 上下文管理器
```python
async with AsyncDockerizedTerminal(container) as terminal:
    output = await terminal.run_command("ls -la")
```
`__aenter__` → `init()`，`__aexit__` → `close()`。

## 执行流程示例
```
AsyncDockerizedTerminal.run_command("python script.py")
  │
  ├─ DockerSession._sanitize_command("python script.py")  # 安全检查
  ├─ 发送 "python script.py\necho $?\n" 到 socket
  ├─ 读取输出（过滤回显和退出码）
  │    ├─ 跳过第一行（命令回显）
  │    ├─ 收集实际输出行
  │    └─ 跳过 "echo $?" 和数字行
  └─ 返回清理后的输出文本
```

## 调用关系
- 使用 `docker` 库（`APIClient`、`Container`、`APIError`）
- 被 `app/sandbox/core/sandbox.py` 的 `DockerSandbox` 使用
- 异常被上层通过 `SandboxTimeoutError` 包装

## 安全设计
- 命令注入黑名单检查
- 非阻塞 socket 通信
- 输出自动过滤敏感信息（命令回显、退出码）
- 所有清理操作都有异常防护（`try/except pass`）
