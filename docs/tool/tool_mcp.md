# `app/tool/mcp.py` — MCP 客户端工具集成

## 文件位置
`app/tool/mcp.py`

## 核心作用
实现 Model Context Protocol (MCP) 客户端，允许 Agent 通过 SSE 或 stdio 协议连接到远程 MCP 服务器，将远程工具注册到本地 `ToolCollection` 中使用。

## 类结构

### MCPClientTool(BaseTool)
远程工具的客户端代理：

| 属性 | 类型 | 说明 |
|------|------|------|
| `session` | `Optional[ClientSession]` | MCP 客户端会话 |
| `server_id` | `str` | 服务器标识 |
| `original_name` | `str` | 远程工具原始名称 |

**execute()：** 通过 `session.call_tool(original_name, kwargs)` 远程执行，提取 `TextContent` 返回。

### MCPClients(ToolCollection)
MCP 服务器连接和工具集合管理：

| 属性 | 类型 | 说明 |
|------|------|------|
| `sessions` | `dict[str, ClientSession]` | 服务器 ID 到会话的映射 |
| `exit_stacks` | `dict[str, AsyncExitStack]` | 资源清理栈 |

**连接方式：**

| 方法 | 传输协议 | 使用场景 |
|------|---------|---------|
| `connect_sse(server_url)` | SSE (HTTP) | 远程 MCP 服务器 |
| `connect_stdio(command, args)` | stdio (子进程) | 本地 MCP 服务器 |

**连接流程：**
1. 断开旧连接（如有）
2. 创建 `AsyncExitStack` 管理资源
3. 建立传输通道（SSE 或 stdio）
4. 初始化 MCP session
5. `list_tools()` 获取远程工具列表
6. 为每个工具创建 `MCPClientTool` 代理，注册到 `tool_map`

**_sanitize_tool_name()：** 清理工具名称（替换非法字符、去重下划线、截断 64 字符）。

**disconnect()：** 断开指定或所有服务器，清理会话和工具映射。

## 设计模式
- **代理模式**：`MCPClientTool` 是远程工具的本地代理
- **适配器模式**：将 MCP 协议的工具适配为 `BaseTool` 接口
- MCP 工具名称格式：`mcp_{server_id}_{original_name}`
- `AsyncExitStack` 确保资源正确释放

## 调用关系
- 继承 `ToolCollection`，可作为普通工具集合提供给 Agent
- 依赖 `mcp` 第三方库（Python MCP SDK）
- 使用 SSE 或 stdio 传输协议
