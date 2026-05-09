# `app/agent/__init__.py` — Agent 模块导出入口

## 文件位置
`app/agent/__init__.py`

## 核心作用
从各个子模块导入所有公开的 Agent 类，统一导出。

## 导出列表

| 导出名 | 来源文件 | 说明 |
|--------|---------|------|
| `BaseAgent` | `app/agent/base.py` | Agent 抽象基类 |
| `BrowserAgent` | `app/agent/browser.py` | 浏览器自动化 Agent |
| `ReActAgent` | `app/agent/react.py` | 思考-行动循环 Agent |
| `SWEAgent` | `app/agent/swe.py` | 代码执行 Agent |
| `ToolCallAgent` | `app/agent/toolcall.py` | 工具调用 Agent |
| `MCPAgent` | `app/agent/mcp.py` | MCP 服务器交互 Agent |

## 设计说明
- 未导出 `Manus`、`SandboxManus`、`DataAnalysis` 等具体 Agent 实现
- 这些未导出的 Agent 需通过完整路径导入（如 `from app.agent.manus import Manus`）
