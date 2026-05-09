# `app/tool/terminate.py` — 终止交互工具

## 文件位置
`app/tool/terminate.py`

## 核心作用
提供 Agent 结束交互的标准方式。当任务完成或无法继续时，Agent 调用此工具通知系统结束执行。

## 类结构

### Terminate(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"terminate"` |
| `description` | 提示 LLM 在任务完成或无法继续时调用 |

参数 schema：

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `enum[str]` | `"success"` 或 `"failure"` |

### execute(status)

返回字符串 `"The interaction has been completed with status: {status}"`。

## 设计说明
- Agent 系统收到此工具的返回后，会判断任务结束
- 在 `app/agent/base.py` 的主循环中，`Terminate` 调用的返回会触发 `FINISHED` 状态
- 参数 `status` 允许 LLM 区分成功完成任务和无法继续的情况
