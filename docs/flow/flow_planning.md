# `app/flow/planning.py` — 计划式执行流

## 文件位置
`app/flow/planning.py`

## 核心作用
`PlanningFlow` 是项目中最核心的 Flow 实现，它协调多个 Agent 按计划分步骤执行任务。工作流程为：创建计划 → 分配步骤给 Agent → 执行 → 标记完成 → 循环直到全部完成。

## 辅助枚举

### PlanStepStatus(str, Enum)
步骤状态的枚举定义：

| 状态 | 值 | 符号 |
|------|-----|------|
| `NOT_STARTED` | `"not_started"` | `[ ]` |
| `IN_PROGRESS` | `"in_progress"` | `[→]` |
| `COMPLETED` | `"completed"` | `[✓]` |
| `BLOCKED` | `"blocked"` | `[!]` |

提供 3 个类方法：`get_all_statuses()`、`get_active_statuses()`（NOT_STARTED + IN_PROGRESS）、`get_status_marks()`。

## 类结构

### PlanningFlow(BaseFlow)

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `llm` | `LLM` | `LLM()` | 用于计划创建的 LLM |
| `planning_tool` | `PlanningTool` | `PlanningTool()` | 计划管理工具 |
| `executor_keys` | `List[str]` | `[]` | 执行 Agent 的 key 列表 |
| `active_plan_id` | `str` | `plan_{timestamp}` | 当前活动计划 ID |
| `current_step_index` | `Optional[int]` | `None` | 当前步骤索引 |

### __init__() — 初始化
在 `BaseFlow.__init__()` 前处理特殊参数：
- `executors` → 映射为 `executor_keys`
- `plan_id` → 映射为 `active_plan_id`
- 自动创建 `PlanningTool` 实例
- 未指定 `executor_keys` 时，默认使用所有 Agent key

### execute(input_text) — 主执行流程
```python
async def execute(self, input_text: str) -> str:
    1. _create_initial_plan(input_text)     # 创建计划
    2. while True:
        3. _get_current_step_info()          # 获取下一步
        4. if 无下一步: _finalize_plan(); break
        5. get_executor(step_type)           # 选择执行 Agent
        6. _execute_step(executor, step_info) # 执行步骤
        7. if AgentState.FINISHED: break
    8. return result
```

### _create_initial_plan(request) — 创建计划
1. 构建系统提示词，包含 Agent 描述信息
2. 如果有多 Agent，指示 LLM 在步骤中使用 `[agent_name]` 格式标记执行者
3. 调用 `self.llm.ask_tool()` 让 LLM 使用 `PlanningTool.create()` 创建计划
4. 如果 LLM 未调用工具，创建默认计划（"Analyze request → Execute task → Verify results"）

### get_executor(step_type) — 选择执行 Agent
根据步骤类型选择 Agent：
- 如果 `step_type` 与某 Agent key 匹配，使用该 Agent
- 否则使用 `executor_keys` 中第一个可用 Agent
- 最终回退到 `primary_agent`

步骤类型通过正则 `\[([A-Z_]+)\]` 从步骤文本中提取（如 `[SEARCH]`、`[CODE]`）。

### _execute_step(executor, step_info) — 执行步骤
1. 获取当前计划状态文本
2. 构建执行提示词（含计划状态和当前任务）
3. 调用 `executor.run(step_prompt)` 让 Agent 执行
4. 执行成功后调用 `_mark_step_completed()`

### _get_current_step_info() — 获取当前步骤
查找计划中第一个状态为 `not_started` 或 `in_progress` 的步骤：
1. 从 `planning_tool.plans` 中读取步骤列表和状态
2. 找到第一个活跃步骤
3. 将其标记为 `in_progress`
4. 返回 `(step_index, step_info)`

步骤信息包含 `text` 以及通过正则 `\[([A-Z_]+)\]` 提取的 `type`。

### _finalize_plan() — 结束计划
1. 获取最终计划状态
2. 使用 `self.llm.ask()` 生成执行总结
3. 如果 LLM 调用失败，回退到 `primary_agent.run()`
4. 兜底返回 `"Plan completed. Error generating summary."`

### _get_plan_text() / _generate_plan_text_from_storage()
获取格式化计划文本。首选通过 `planning_tool.execute(command="get")`，失败时直接从 `planning_tool.plans` 存储中生成。

## 执行流程示例
```
PlanningFlow.execute("搜索并下载视频")
  │
  ├─ _create_initial_plan()
  │    ├─ LLM 调用 PlanningTool.create()
  │    └─ 创建 steps: ["[SEARCH] 搜索资源", "[DOWNLOAD] 下载文件", "[VERIFY] 验证结果"]
  │
  ├─ 循环迭代 steps:
  │    ├─ Step 0: get_executor("search") → BrowserAgent
  │    │    └─ _execute_step(BrowserAgent, "搜索资源")
  │    │         ├─ BrowserAgent.run(step_prompt)
  │    │         └─ _mark_step_completed()
  │    │
  │    ├─ Step 1: get_executor("download") → Manus
  │    │    └─ _execute_step(Manus, "下载文件")
  │    │
  │    └─ Step 2: get_executor("verify") → Manus
  │         └─ _execute_step(Manus, "验证结果")
  │
  └─ _finalize_plan()
       └─ LLM 生成执行总结
```

## 调用关系
- 继承 `BaseFlow`（`app/flow/base.py`）
- 使用 `PlanningTool`（`app/tool/planning.py`）管理计划
- 使用 `LLM.ask_tool()` 生成初始计划
- 使用 `LLM.ask()` 生成最终总结
- 通过 `FlowFactory.create_flow(FlowType.PLANNING, agents)` 创建
- 支持多 Agent 协作，Agent 类型包括 `Manus`、`BrowserAgent`、`SWEAgent` 等
