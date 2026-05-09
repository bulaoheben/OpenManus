# `app/prompt/mcp.py` — MCP Agent 提示词

## 文件位置
`app/prompt/mcp.py`

## 核心作用
为 `MCPAgent`（`app/agent/mcp.py`）提供与 MCP 服务器交互相关的系统提示词、下一步提示词以及错误处理提示词。

## 内容

### SYSTEM_PROMPT（21 行）
指导 Agent 如何与 MCP 服务器交互：
- 访问 MCP 服务器提供的动态工具
- 使用前先检查可用工具
- 提供正确格式的参数
- 优雅处理错误
- 多步骤任务逐一调用工具
- 清晰解释推理过程和行动

### NEXT_STEP_PROMPT（6 行）
提示 Agent 逐步思考问题，识别当前阶段最有帮助的 MCP 工具。

### TOOL_ERROR_PROMPT（8 行）
工具调用出错时的修复提示：
- 检查参数是否正确
- 参数格式是否无效
- 工具是否已不可用
- 操作是否不支持

使用 `{tool_name}` 占位符替换出错工具的名称。

### MULTIMEDIA_RESPONSE_PROMPT（4 行）
当工具返回多媒体内容（图片、音频）时，提示 Agent 已接收并处理了该内容，要求使用这些信息继续任务或向用户提供见解。

使用 `{tool_name}` 占位符替换返回多媒体的工具名称。

## 使用关系
- 全部被 `app/agent/mcp.py` 的 `MCPAgent` 使用
- `TOOL_ERROR_PROMPT` 在工具执行出错时作为系统消息加入记忆
- `MULTIMEDIA_RESPONSE_PROMPT` 在 `MCPAgent._handle_special_tool()` 中当工具返回 `base64_image` 时加入记忆
- 提示词强调 MCP 工具的**动态性**——工具可能随时增加或消失
