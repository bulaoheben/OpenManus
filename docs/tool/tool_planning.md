# `app/tool/planning.py` — 计划管理工具

[toc]

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



## 个人理解：

PlanningTool本身只是个数据维护工具，不参与"生成计划"这件事。关键在于 LLM Function
  Calling 机制把两者串联起来了。

  整个链路是这样的：

  你的输入 "搜索并下载视频"
      │
      ▼
  PlanningFlow._create_initial_plan()                    #
  flow/planning.py:136
      │  self.llm.ask_tool(tools=[self.planning_tool.to_param()])
      │  # 把 PlanningTool 的描述 + 参数结构（create/steps/title 等）发给
  LLM
      ▼
  LLM 看到 tools 参数里有 planning 工具的 schema，它"思考"后决定：
    → "我需要创建一个计划，调用 planning 工具的 create 命令"
    → 返回的不是文本，而是 tool_call：
       {
         name: "planning",
         arguments: '{"command":"create","title":"搜索并下载视频","steps":["
  [SEARCH]搜索资源","[DOWNLOAD]下载文件"]}'
       }
      │
      ▼
  PlanningFlow 解析 tool_call                              #
  flow/planning.py:179-198
      │  json.loads(arguments) → args
      │  args["plan_id"] = self.active_plan_id
      │  self.planning_tool.execute(**args)
      ▼
  PlanningTool._create_plan()                              #
  tool/planning.py:120
      │  self.plans[plan_id] = {
      │    "steps": ["[SEARCH]搜索资源", "[DOWNLOAD]下载文件"],
      │    "step_statuses": ["not_started", "not_started"],
      │    "step_notes": ["", ""]
      │  }
      │  self._current_plan_id = plan_id
      ▼
  Plan 创建完成，存在 PlanningTool.plans 字典里

  核心区别：LLM 并没有直接操作内存，它只是输出了一个 JSON格式的工具调用参数。PlanningFlow 收到这个参数后，调用
  PlanningTool.execute(command="create", steps=[...]) 来真正执行创建。LLM是"出脑力"（生成步骤内容），PlanningTool 是"出体力"（把步骤存到内存）。

  这就是你说的 LLM 生成的 Plan 通过工具调用转变为自己真正的 Plan 的完整过程——LLM 不知道也不关心数据怎么存，它只负责通过 function calling 的schema 描述来指挥工具做事。
