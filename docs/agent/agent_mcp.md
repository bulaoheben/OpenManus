# `app/agent/mcp.py` — MCP Agent

## 文件位置
`app/agent/mcp.py`

## 核心作用
专门用于连接和操作 MCP（Model Context Protocol）服务器的 Agent。支持 SSE 和 stdio 两种传输协议，可动态刷新远程工具列表。

## 类结构

### MCPAgent(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"mcp_agent"` | |
| `max_steps` | 20 | |
| `connection_type` | `"stdio"` | 默认连接类型 |
| `available_tools` | `None` | 初始化时设置为 `mcp_clients` |
| `_refresh_tools_interval` | 5 | 每 N 步刷新一次工具列表 |

### initialize(connection_type, server_url, command, args)
初始化 MCP 连接：
1. 根据 `connection_type` 选择连接方式（SSE 或 stdio）
2. `mcp_clients.connect_sse()` 或 `mcp_clients.connect_stdio()`
3. 将 `available_tools` 设置为 `mcp_clients`（MCPClients 继承 ToolCollection）
4. `_refresh_tools()` 记录初始工具 schema
5. 添加系统消息告知 LLM 可用工具

### _refresh_tools()
定期刷新 MCP 服务器工具列表，检测新增、移除和变更的工具：
- 调用 `mcp_clients.list_tools()` 获取当前工具
- 对比 `tool_schemas` 中的历史记录
- 将变更通知添加到 Agent 记忆

### think()
在父类 `think()` 之前：
1. 检查 MCP 会话和工具是否可用
2. 每 `_refresh_tools_interval` 步刷新一次工具列表
3. 如果所有工具都移除，判定服务关闭

### _handle_special_tool(name, result)
处理特殊工具的额外逻辑：如果工具返回了 `base64_image`，添加多媒体响应提示词到 Agent 记忆。

### _should_finish_execution(name, **kwargs)
工具名称为 `"terminate"` 时结束执行。

### cleanup()
断开 MCP 连接。

### run(request)
确保 cleanup 在 run 结束后执行（包括异常情况）。

## 调用关系
- 不使用 `Manus` 的 MCP 配置加载方式，而是通过 `initialize()` 手动指定连接参数
- `available_tools` 直接设置为 `MCPClients` 实例
- 每次 `think()` 前检查工具可用性，适应动态变化的 MCP 服务
