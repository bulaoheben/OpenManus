# `app/prompt/toolcall.py` — ToolCallAgent 提示词

## 文件位置
`app/prompt/toolcall.py`

## 核心作用
为 `ToolCallAgent`（`app/agent/toolcall.py`）提供最基础的系统提示词和下一步提示词。

## 内容

### SYSTEM_PROMPT
```python
"You are an agent that can execute tool calls"
```
极简的系统提示词，仅声明 Agent 具有执行工具调用的能力。被 `ToolCallAgent` 及其所有子类继承使用。

### NEXT_STEP_PROMPT
```python
"If you want to stop interaction, use `terminate` tool/function call."
```
提示 Agent 在需要结束交互时使用 `terminate` 工具。

## 设计说明
- 只有 2 行，是所有提示词中最简洁的
- 被 `ToolCallAgent.think()` 在调用 LLM 时用作系统级和下一步提示词
- 子类（`Manus`、`SWEAgent`、`BrowserAgent` 等）通过覆盖 `system_prompt` 和 `next_step_prompt` 属性来替换更具体的提示词
