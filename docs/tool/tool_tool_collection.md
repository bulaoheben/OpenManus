# `app/tool/tool_collection.py` — 工具集合管理

## 文件位置
`app/tool/tool_collection.py`

## 核心作用
管理和分发多个工具的集合类。Agent 将所有可用工具注册到 `ToolCollection` 中，通过名称查找并执行对应的工具。

## 类结构

### ToolCollection

| 属性 | 类型 | 说明 |
|------|------|------|
| `tools` | `tuple[BaseTool]` | 工具列表 |
| `tool_map` | `dict[str, BaseTool]` | 名称到工具的映射 |

| 方法 | 说明 |
|------|------|
| `to_params()` | 返回所有工具的 OpenAI function calling schema 列表 |
| `execute(name, tool_input)` | 按名称查找并执行工具，返回 ToolResult |
| `execute_all()` | 依次执行所有工具（通常仅用于测试） |
| `get_tool(name)` | 按名称获取工具实例 |
| `add_tool(tool)` | 添加单个工具（同名则跳过并警告） |
| `add_tools(*tools)` | 批量添加工具 |

## 执行流程
```python
result = await tool_collection.execute(name="bash", tool_input={"command": "ls -l"})
```
1. `tool_map.get(name)` 查找工具
2. 找不到返回 `ToolFailure`
3. 找到则调用 `tool(**tool_input)` → `tool.execute(**tool_input)`
4. `ToolError` 异常捕获为 `ToolFailure`

## 调用关系
- 在 `app/agent/toolcall.py:166` 的 `execute_tool()` 中调用
- Agent 初始化时通过 `add_tools()` 或构造函数注入注册所有工具
- 每个 Agent 实例通常持有一个 `ToolCollection`
