# `app/logger.py` — 日志系统

## 文件位置
`app/logger.py`

## 核心作用
配置基于 `loguru` 的日志系统，同时输出到控制台和文件，并导出全局 `logger` 实例。

## 函数

### `define_log_level(print_level, logfile_level, name)`
配置日志输出：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `print_level` | `"INFO"` | 控制台日志级别 |
| `logfile_level` | `"DEBUG"` | 文件日志级别 |
| `name` | `None` | 日志文件名前缀 |

执行流程：
1. 移除默认 handler
2. 配置 stderr 为 UTF-8 编码（确保中文正确显示）
3. 添加控制台 handler（`print_level`）
4. 添加文件 handler：`logs/{name}_{时间戳}.log`（`logfile_level`）

### 全局实例
```python
logger = define_log_level()
```
模块导出级别可直接 `from app.logger import logger` 使用。

## 调用关系
- 基于 `loguru` 库
- 被 `app/agent/`、`app/tool/`、`app/flow/`、`app/daytona/`、`app/sandbox/` 等模块引用
- 注意：`app/utils/logger.py` 是另一个使用 `structlog` 的日志模块，两者并存但风格不同
