# `app/tool/bash.py` — Bash 命令执行工具

## 文件位置
`app/tool/bash.py`

## 核心作用
在终端中执行 Bash 命令，支持交互式进程、长时间运行命令和超时控制。维护一个持久的 shell 会话，跨多次工具调用保持状态。

## 类结构

### _BashSession
Bash shell 会话管理，使用 asyncio subprocess 实现：

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `command` | `str` | `/bin/bash` | shell 类型 |
| `_output_delay` | `float` | `0.2` | 输出轮询间隔 |
| `_timeout` | `float` | `120.0` | 命令超时时间 |
| `_sentinel` | `str` | `"<<exit>>"` | 输出结束标记 |

| 方法 | 说明 |
|------|------|
| `start()` | 启动 asyncio subprocess shell |
| `stop()` | 终止 shell 进程 |
| `run(command)` | 执行命令，通过 sentinel 标记检测输出结束 |

**run 方法的工作机制：**
1. 将命令写入 stdin，追加 `echo '<<exit>>'`
2. 异步读取 stdout buffer，等待 sentinel 出现
3. 超时（120s）则标记 `_timed_out`，要求重启会话
4. 命令退出码为 -1 表示进程未完成，LLM 可发送空命令获取更多日志或 `ctrl+c` 中断

### Bash(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"bash"` |
| `description` | 详细说明长命令后台运行、交互式命令处理、超时重试策略 |

参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `command` | `str` | Bash 命令，可为空（获取日志）或 `ctrl+c`（中断） |

**execute 方法特性：**
- `restart=True`：重启会话
- 自动延迟初始化 session
- 支持交互式命令（exit code -1）

## 设计说明
- 使用 asyncio subprocess 而非 blocking subprocess，避免阻塞事件循环
- **注意**：在 Windows 上 `preexec_fn=os.setsid` 不受支持，当前仅兼容 Linux/macOS
- sentinel 模式确保能检测命令完成，而非固定超时
- 会话持久化使得 `cd` 等状态操作跨调用生效

## 调用关系
- Agent 通过 ToolCollection 调用
- 使用 `CLIResult`（而非普通 ToolResult）输出
- 异常时抛出 `ToolError`
