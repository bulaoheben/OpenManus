# `app/agent/browser.py` — 浏览器 Agent 与辅助工具

## 文件位置
`app/agent/browser.py`

## 核心作用
提供专门用于浏览器自动化的 `BrowserAgent`，以及辅助类 `BrowserContextHelper`（被多个 Agent 复用）。`BrowserContextHelper` 负责获取浏览器状态、截图并将信息注入到 Agent 的提示词中。

## 类结构

### BrowserContextHelper
浏览器状态辅助类，不继承任何基类：

| 属性 | 类型 | 说明 |
|------|------|------|
| `agent` | `BaseAgent` | 所属的 Agent 实例 |
| `_current_base64_image` | `Optional[str]` | 当前浏览器截图 |

#### get_browser_state()
1. 从 Agent 的 `available_tools` 中查找 `BrowserUseTool`（或 `SandboxBrowserTool`）
2. 调用 `browser_tool.get_current_state()` 获取状态
3. 返回 JSON 解析后的状态字典（含 URL、标题、标签页、可点击元素等）

#### format_next_step_prompt()
1. 调用 `get_browser_state()` 获取当前状态
2. 格式化 URL、标签页数、滚动信息
3. 如果有截图，添加到 Agent 记忆
4. 返回格式化后的提示词模板

#### cleanup_browser()
调用 `BrowserUseTool.cleanup()` 清理浏览器资源。

### BrowserAgent(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"browser"` | |
| `max_observe` | 10000 | |
| `max_steps` | 20 | |
| `available_tools` | `BrowserUseTool, Terminate` | 仅浏览器 + 终止 |
| `tool_choices` | `ToolChoice.AUTO` | 允许自由文本回复 |

#### think()
每一步都通过 `BrowserContextHelper.format_next_step_prompt()` 注入当前浏览器状态信息，然后调用父类的 `think()`。

#### cleanup()
清理浏览器资源。

## 设计说明
- `BrowserContextHelper` 被 `Manus`、`SandboxManus` 和 `BrowserAgent` 复用
- `BrowserAgent` 的工具集非常精简（仅浏览器 + 终止），专注于网页浏览任务
- `tool_choices = AUTO` 允许 LLM 在不需要浏览器操作时直接回复文本
