# `app/agent/manus.py` — Manus 通用 Agent

## 文件位置
`app/agent/manus.py`

## 核心作用
系统的默认通用 Agent，组合了最常用的工具集（浏览器、Python 执行、文件编辑、人工咨询），并集成了 MCP 服务器支持。

## 类结构

### Manus(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"Manus"` | |
| `max_observe` | 10000 | 观察结果最大长度 |
| `max_steps` | 20 | 默认最大步数 |
| `available_tools` | `PythonExecute, BrowserUseTool, StrReplaceEditor, AskHuman, Terminate` | 通用工具集 |
| `mcp_clients` | `MCPClients()` | MCP 客户端 |
| `connected_servers` | `dict` | 已连接的 MCP 服务器 |

### create() — 工厂方法
```python
@classmethod
async def create(cls, **kwargs) -> "Manus":
    instance = cls(**kwargs)
    await instance.initialize_mcp_servers()
    instance._initialized = True
    return instance
```
异步创建并初始化 MCP 连接。

### initialize_mcp_servers()
遍历 `config.mcp_config.servers` 配置，连接到所有 MCP 服务器（支持 SSE 和 stdio 两种传输协议）。

### connect_mcp_server()
连接到单个 MCP 服务器，将服务器提供的工具添加到 `available_tools`。

### disconnect_mcp_server()
断开 MCP 服务器连接，清理工具集合（移除 MCPClientTool 实例）。

### think() — 思考阶段（重写）
在标准 `think()` 基础上增加了浏览器状态感知：
1. 检查最近 3 条消息是否使用了 `BrowserUseTool`
2. 如果是，则通过 `BrowserContextHelper` 获取当前浏览器状态
3. 将浏览器状态信息注入 `next_step_prompt`
4. 临时替换 prompt → 调用 `super().think()` → 恢复 prompt

### cleanup()
清理浏览器和 MCP 连接资源。

## 调用关系
- `main.py` 中通过 `Manus.create()` 初始化并运行
- 使用 `BrowserContextHelper` 管理浏览器状态信息
- MCP 服务器配置来自 `config.mcp_config`
