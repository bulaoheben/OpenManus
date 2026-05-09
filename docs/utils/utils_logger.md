# `app/utils/logger.py` — 日志配置

## 文件位置
`app/utils/logger.py`

## 核心作用
配置并导出全局日志记录器，基于 `structlog` 库实现结构化日志。

## 配置详情

### 环境模式检测
```python
ENV_MODE = os.getenv("ENV_MODE", "LOCAL")
```

### 渲染器选择
| 环境 | 渲染器 | 效果 |
|------|--------|------|
| `LOCAL`（默认） | `structlog.dev.ConsoleRenderer()` | 开发友好的彩色控制台输出 |
| 其他（如 `PRODUCTION`） | `structlog.processors.JSONRenderer()` | JSON 格式，适合日志采集 |

### 处理器链
按顺序执行的日志处理器：

| 处理器 | 作用 |
|--------|------|
| `add_log_level` | 添加日志级别 |
| `PositionalArgumentsFormatter()` | 格式化位置参数 |
| `dict_tracebacks` | 字典格式的异常回溯 |
| `CallsiteParameterAdder` | 添加调用位置信息（文件名、函数名、行号） |
| `TimeStamper(fmt="iso")` | ISO 格式时间戳 |
| `merge_contextvars` | 合并上下文变量 |
| `renderer` | 最终渲染（Console 或 JSON） |

### 导出
```python
logger: structlog.stdlib.BoundLogger = structlog.get_logger(level=logging.DEBUG)
```
全局可用的 `logger` 实例，日志级别为 `DEBUG`。

## 使用示例
```python
from app.utils.logger import logger

logger.info("Sandbox created", sandbox_id="abc123")
logger.error("Operation failed", exc_info=True)
```

## 调用关系
- 被几乎所有模块引用（`app/tool/`、`app/agent/`、`app/daytona/`、`app/sandbox/` 等）
- 基于第三方 `structlog` 库
