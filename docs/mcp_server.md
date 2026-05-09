# `app/mcp/` — MCP 服务模块

## 文件位置
`app/mcp/`

## 文件列表

| 文件 | 大小 | 说明 |
|------|------|------|
| `__init__.py` | 0 字节 | 空文件，标记为 Python 包 |
| `server.py` | 6.7 KB | MCP 服务器实现 |

---

## `__init__.py`
空文件，仅用于将 `app/mcp/` 标记为 Python 包。

---

## `server.py` — MCP 服务器

### 核心作用
将 OpenManus 的工具暴露为 MCP（Model Context Protocol）服务器，使外部 MCP 客户端（如 Claude Desktop、VS Code 等）可以通过标准协议调用 OpenManus 的工具。

### 类结构

#### MCPServer

| 属性 | 类型 | 说明 |
|------|------|------|
| `server` | `FastMCP` | FastMCP 服务器实例 |
| `tools` | `Dict[str, BaseTool]` | 注册的工具字典 |

**初始化时注册的 4 个标准工具：**

| 名称 | 类 | 说明 |
|------|-----|------|
| `"bash"` | `Bash` | Shell 命令执行 |
| `"browser"` | `BrowserUseTool` | 浏览器自动化 |
| `"editor"` | `StrReplaceEditor` | 文件编辑 |
| `"terminate"` | `Terminate` | 终止信号 |

### 关键方法

**register_tool(tool, method_name=None)：**
将 `BaseTool` 实例注册为 FastMCP 工具：
1. 调用 `tool.to_param()` 获取 OpenAI function calling schema
2. 创建异步包装函数 `tool_method(**kwargs)`，执行工具并序列化结果
3. 设置 `__name__`、`__doc__`、`__signature__` 元数据
4. 保存 `_parameter_schema`
5. 通过 `self.server.tool()(tool_method)` 注册到 FastMCP

**结果序列化逻辑：**
```python
if hasattr(result, "model_dump"):    # ToolResult → JSON
    return json.dumps(result.model_dump())
elif isinstance(result, dict):        # 普通字典 → JSON
    return json.dumps(result)
return result                          # 其他（字符串等）→ 直接返回
```

**_build_docstring(tool_function)：**
从 tool schema 构建格式化的文档字符串（含参数列表和 required/optional 标记）。

**_build_signature(tool_function)：**
从 tool schema 构建 Python `Signature` 对象：
- JSON Schema 类型映射到 Python 类型（string→str, integer→int, number→float 等）
- required 参数无默认值，optional 参数默认 `None`
- 所有参数为 `KEYWORD_ONLY`

**register_all_tools()：**
批量注册 `self.tools` 中所有工具。

**run(transport="stdio")：**
启动 MCP 服务器：
1. 注册所有工具
2. 注册清理函数（通过 `atexit`，确保程序退出时清理浏览器资源）
3. 启动 FastMCP 服务器（当前仅支持 `"stdio"` 传输模式）

**cleanup()：**
清理资源，目前仅清理浏览器工具（`BrowserUseTool.cleanup()`）。

### 主入口
```python
if __name__ == "__main__":
    args = parse_args()
    server = MCPServer()
    server.run(transport=args.transport)
```
支持命令行参数 `--transport`（当前仅 `stdio`，默认 `stdio`）。

### 调用关系

```
外部 MCP Client (Claude Desktop 等)
    │  stdio 协议
    ▼
MCPServer (FastMCP)
    │
    ├─ Bash              (本地 Shell 命令)
    ├─ BrowserUseTool    (浏览器自动化)
    ├─ StrReplaceEditor  (文件编辑)
    └─ Terminate         (终止信号)
```

**与 `app/tool/mcp.py` 的关系：**
- `app/tool/mcp.py` 是 **MCP 客户端**实现，连接外部 MCP 服务器获取工具
- `app/mcp/server.py` 是 **MCP 服务器**实现，将自己的工具暴露给外部客户端
- 两者互为镜像，使用同一协议实现双向工具互通

### 设计说明
- 基于 `mcp` 第三方库的 `FastMCP` 实现
- 使用 `FastMCP` 的装饰器 API（`self.server.tool()`）动态注册工具
- 通过动态构建 `Signature` 和 `_parameter_schema`，实现参数校验
- 仅支持 stdio 传输模式（HTTP 模式暂未启用）
