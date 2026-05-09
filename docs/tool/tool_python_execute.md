# `app/tool/python_execute.py` — Python 代码执行工具

## 文件位置
`app/tool/python_execute.py`

## 核心作用
在隔离的子进程中执行 Python 代码，带有超时控制和安全性限制。使用 `multiprocessing.Process` 实现在独立进程中运行。

## 类结构

### PythonExecute(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"python_execute"` |
| `description` | 提示 LLM 只有 print 输出可见 |

参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `code` | `str` | 要执行的 Python 代码 |

### execute(code, timeout=60)

执行流程：
1. 创建 `multiprocessing.Manager().dict()` 用于进程间通信
2. 准备 `safe_globals`（仅包含 `__builtins__`）限定执行环境
3. 启动子进程执行 `_run_code`
4. 等待 `timeout` 秒
5. 如果子进程仍存活则 `terminate()` 并返回超时错误

### _run_code(code, result_dict, safe_globals)
- 重定向 `sys.stdout` 到 `StringIO` 捕获所有 print 输出
- `exec(code, safe_globals)` 执行代码
- 异常时捕获并写入 result_dict

## 设计说明
- **多进程隔离**：使用 `multiprocessing.Process` 而非 `subprocess`，保证内存隔离
- **安全限制**：仅暴露 `__builtins__`，不包含任何已导入模块
- **超时控制**：`proc.join(timeout)` + `proc.terminate()` 双重保障
- **已知限制**：`Manager()` 的 `with` 块退出会 kill 子进程；不支持持久化后台任务；不保留状态跨调用
- 超时从 5 秒调整为 60 秒（修复记录）

## 调用关系
- Agent 通过 ToolCollection 调用
- `chart_visualization/python_execute.py` 中的 `NormalPythonExecute` 继承此类，增加 `code_type` 参数
- 输出为 `Dict`（含 observation 和 success），需手动转为 ToolResult 格式
