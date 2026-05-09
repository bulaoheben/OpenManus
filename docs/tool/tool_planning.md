# `app/tool/planning.py` — 计划管理工具

## 文件位置
`app/tool/planning.py`

## 核心作用
允许 Agent 创建和管理复杂任务的执行计划。支持 7 种命令：创建、更新、列出、查看、设置活动、标记步骤、删除计划。

## 类结构

### PlanningTool(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"planning"` |
| `description` | 创建和管理计划的工具 |

| 属性 | 类型 | 说明 |
|------|------|------|
| `plans` | `dict` | 按 plan_id 存储所有计划 |
| `_current_plan_id` | `Optional[str]` | 当前活动计划 ID |

参数（7 个参数 + 1 个 required=command）：

| command | plan_id | title | steps | step_index | step_status | step_notes |
|---------|---------|-------|-------|------------|-------------|------------|
| create | required | required | required | - | - | - |
| update | required | optional | optional | - | - | - |
| list | - | - | - | - | - | - |
| get | optional | - | - | - | - | - |
| set_active | required | - | - | - | - | - |
| mark_step | optional | - | - | required | required | optional |
| delete | required | - | - | - | - | - |

### 命令详解

**create：** 创建新计划，所有步骤初始化为 `not_started`，自动设为活动计划
**update：** 更新标题或步骤，保留旧步骤的状态和备注（位置匹配）
**list：** 列出所有计划，显示进度（X/Y completed）和活动标记
**get：** 查看计划详情，默认返回活动计划
**set_active：** 设置活动计划
**mark_step：** 更新步骤状态（not_started/in_progress/completed/blocked）和备注
**delete：** 删除计划，如为活动计划则清空 `_current_plan_id`

### _format_plan()
格式化计划输出：标题、进度百分比、状态统计（completed/in_progress/blocked/not_started）、步骤列表（[✓]/[→]/[!]/[ ] 符号）。

## 设计模式
- **命令模式**：7 种命令统一通过 `execute(command=...)` 入口
- 计划数据以字典形式存储在 `plans` 属性中（非持久化，进程内存内）
- 步骤状态追踪支持进度统计（用于 LLM 感知任务完成度）
