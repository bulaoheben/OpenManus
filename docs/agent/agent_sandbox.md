# `app/agent/sandbox_agent.py` — 沙盒 Agent

## 文件位置
`app/agent/sandbox_agent.py`

## 核心作用
在 Daytona 沙盒环境中运行的 Agent 版本。支持沙盒浏览器、文件操作、Shell 命令和视觉工具，所有操作在隔离的容器环境中执行。

## 类结构

### SandboxManus(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"SandboxManus"` | |
| `max_observe` | 10000 | |
| `max_steps` | 20 | |
| `available_tools` | `AskHuman, Terminate`（初始） | 沙盒工具在初始化时动态添加 |
| `mcp_clients` | `MCPClients()` | MCP 客户端 |
| `sandbox_link` | `dict` | 沙盒链接（VNC/Website） |

### create() — 工厂方法
```python
@classmethod
async def create(cls, **kwargs) -> "SandboxManus":
    instance = cls(**kwargs)
    await instance.initialize_mcp_servers()
    await instance.initialize_sandbox_tools()
    instance._initialized = True
    return instance
```

### initialize_sandbox_tools()
1. 调用 `create_sandbox(password)` 创建新的 Daytona 沙盒
2. 获取 VNC（端口 6080）和 Website（端口 8080）预览链接
3. 创建沙盒工具实例并添加到 `available_tools`：
   - `SandboxBrowserTool` — 浏览器自动化
   - `SandboxFilesTool` — 文件操作
   - `SandboxShellTool` — Shell 命令
   - `SandboxVisionTool` — 图片读取
4. 设置 `SandboxToolsBase._urls_printed = True`

### connect_mcp_server() / disconnect_mcp_server()
与 `Manus` 相同的 MCP 服务器管理逻辑。

### think()
与 `Manus` 类似，但检测 `SandboxBrowserTool` 而非 `BrowserUseTool` 是否在使用中。

### delete_sandbox(sandbox_id)
删除指定沙盒并清理 `sandbox_link`。

### cleanup()
清理浏览器、断开 MCP、删除沙盒。

## 与 Manus 的对比

| 特性 | Manus | SandboxManus |
|------|-------|-------------|
| 运行环境 | 本地 | Daytona 沙盒 |
| 浏览器工具 | `BrowserUseTool`（本地 Playwright） | `SandboxBrowserTool`（沙盒内 API） |
| 文件操作 | `StrReplaceEditor`（本地文件系统） | `SandboxFilesTool`（沙盒 `/workspace`） |
| Shell 命令 | `Bash`（本地 subprocess） | `SandboxShellTool`（沙盒 tmux） |
| 视觉能力 | 无专用工具 | `SandboxVisionTool` |
| 资源清理 | 关闭浏览器 | 删除整个沙盒容器 |
