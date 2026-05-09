# `app/tool/sandbox/sb_shell_tool.py` — 沙盒 Shell 命令执行工具

## 文件位置
`app/tool/sandbox/sb_shell_tool.py`

## 核心作用
在 Daytona 沙盒中通过 tmux 会话执行 Shell 命令。默认非阻塞执行，适合长期运行的任务（如启动服务器、构建进程）。

## 类结构

### SandboxShellTool(SandboxToolsBase)

| 字段 | 值 |
|------|-----|
| `name` | `"sandbox_shell"` |
| `description` | 沙盒 Shell 命令执行，基于 tmux 非阻塞 |

参数（4 种 action）：

| action | 必填参数 | 说明 |
|--------|---------|------|
| `execute_command` | `command` | 执行命令（支持阻塞/非阻塞） |
| `check_command_output` | `session_name` | 检查命令输出 |
| `terminate_command` | `session_name` | 终止命令 |
| `list_commands` | 无 | 列出所有活跃 tmux 会话 |

### _execute_command(command, folder, session_name, blocking, timeout)

**非阻塞模式（默认）：**
1. 创建/复用 tmux 会话
2. 发送命令到 tmux
3. 立即返回 session_name，Agent 可后续调用 `check_command_output`

**阻塞模式：**
1. 发送命令到 tmux
2. 每 2 秒轮询检查完成状态（检测 $ # > Done 等提示符）
3. 超时（默认 60s）后捕获最终输出
4. 自动清理 tmux 会话

**_execute_raw_command(command)：**
执行辅助命令（tmux 控制指令），使用 `sandbox.process.execute_session_command()` 通过 Dayona session API 执行。

**tmux 工作流：**
```
tmux new-session -d -s {name}        # 创建会话
tmux send-keys -t {name} "cmd" Enter # 发送命令
tmux capture-pane -t {name} -p       # 捕获输出
tmux kill-session -t {name}          # 删除会话
tmux has-session -t {name}           # 检查会话存在
tmux list-sessions                   # 列出会话
tmux kill-server                     # 清理所有
```

### _cleanup_session()
清理时删除 tmux 会话和 `_sessions` 记录。

## 与 Bash 工具的对比

| 特性 | Bash (bash.py) | SandboxShellTool |
|------|---------------|------------------|
| 环境 | 本地 asyncio subprocess | 沙盒内 tmux session |
| 持久化 | 单会话，跨调用保持 | 多会话命名，非阻塞 |
| 超时 | 120 秒 | 可配置，默认 60 |
| 长任务 | 需后台运行 + 文件重定向 | 天然支持（非阻塞） |
| 中断处理 | ctrl+c 命令 | terminate_command action |
