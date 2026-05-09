# `app/agent/swe.py` — SWE Agent (代码执行 Agent)

## 文件位置
`app/agent/swe.py`

## 核心作用
专注代码执行任务的 Agent。源于 SWE-agent 概念，提供 Bash 命令和文件编辑能力，适合自动化编程和系统操作任务。

## 类结构

### SWEAgent(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"swe"` | |
| `description` | 自主 AI 程序员，直接与计算机交互解决问题 | |
| `max_steps` | 20 | |
| `available_tools` | `Bash, StrReplaceEditor, Terminate` | 仅 3 个工具 |

| 属性 | 值 |
|------|-----|
| `system_prompt` | `SYSTEM_PROMPT`（来自 `app.prompt.swe`） |
| `next_step_prompt` | `""`（空字符串） |
| `special_tool_names` | `[Terminate().name]` |

## 设计说明
- 工具集极简（Bash + 文件编辑 + 终止），无浏览器或搜索工具
- `next_step_prompt` 为空，依赖 `system_prompt` 引导行为
- 适合纯代码任务：编写代码、执行脚本、编辑文件
- 不重写 `think()` 或 `act()`，完全使用 `ToolCallAgent` 的默认行为
