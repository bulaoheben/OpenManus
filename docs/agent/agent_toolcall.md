# `app/agent/toolcall.py` — 工具调用 Agent

[toc]

## 文件位置

`app/agent/toolcall.py`

## 核心作用
`ToolCallAgent` 是 ReAct 模式的核心实现，提供 LLM 工具调用的完整能力。LLM 在 `think()` 阶段决定调用哪些工具，在 `act()` 阶段执行这些工具并处理结果。

## 类结构

### ToolCallAgent(ReActAgent)

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | `str` | `"toolcall"` | |
| `system_prompt` | `str` | `SYSTEM_PROMPT` | 系统提示词 |
| `next_step_prompt` | `str` | `NEXT_STEP_PROMPT` | 下一步提示词 |
| `available_tools` | `ToolCollection` | `CreateChatCompletion(), Terminate()` | 可用工具集合 |
| `tool_choices` | `TOOL_CHOICE_TYPE` | `ToolChoice.AUTO` | 工具选择模式 |
| `special_tool_names` | `List[str]` | `[Terminate().name]` | 特殊工具名列表 |
| `tool_calls` | `List[ToolCall]` | `[]` | 当前步的工具调用列表 |
| `max_steps` | `int` | 30 | 最大步数 |
| `max_observe` | `Optional[Union[int, bool]]` | None | 观察结果截断长度 |

### think() — 决策阶段
1. 如果 `next_step_prompt` 存在，添加到消息列表
2. 调用 `llm.ask_tool()` 获取 LLM 响应（含工具调用）
3. 异常处理：捕获 `TokenLimitExceeded`，转为 `FINISHED` 状态
4. 提取 `tool_calls` 和 `content`
5. 根据 `tool_choices` 模式处理：
   - **NONE**：不允许使用工具，返回是否有文本内容
   - **AUTO**：自由模式，有工具调用或用内容则返回 `True`
   - **REQUIRED**：必须使用工具，无工具调用时仍返回 `True`（在 act 中处理）
6. 添加 assistant 消息到记忆
7. 返回 `bool` 指示是否需要执行 act

### act() — 执行阶段
1. 如果 `tool_choices == REQUIRED` 但无工具调用，抛出 `ValueError`
2. 遍历 `self.tool_calls`，每个调用 `execute_tool(command)`
3. 结果可选截断（`max_observe`）
4. 添加 tool 消息到记忆（含可能的 `base64_image`）
5. 返回所有结果拼接字符串

### execute_tool(command) — 单工具执行
1. 验证 ToolCall 格式
2. 在 `tool_map` 中查找工具
3. `json.loads()` 解析参数
4. `available_tools.execute(name=name, tool_input=args)` 执行
5. 调用 `_handle_special_tool()` 处理特殊工具
6. 提取 `base64_image`（如有）
7. 格式化观察结果

### _handle_special_tool(name, result)
调用 `_should_finish_execution()` 判断是否完成任务，是则设置 `state = AgentState.FINISHED`。

### cleanup()
遍历 `available_tools`，调用所有实现了 `cleanup()` 异步方法的工具的清理逻辑。

### run(request)
重写父类 `run()`，添加 `try/finally` 保证 `cleanup()` 始终执行。

## 工作流程
```
ToolCallAgent.step()
  ├─ think()
  │    ├─ 添加 next_step_prompt
  │    ├─ llm.ask_tool() → 获取工具调用决策
  │    └─ 返回是否需要 act
  └─ act()
       ├─ 遍历 tool_calls
       ├─ execute_tool() 逐个执行
       ├─ _handle_special_tool() 检查终止
       └─ 返回结果
```

## 工具选择模式
| 模式 | 行为 |
|------|------|
| `ToolChoice.AUTO` | LLM 自由选择是否使用工具 |
| `ToolChoice.NONE` | 禁止使用工具 |
| `ToolChoice.REQUIRED` | 强制使用工具 |
