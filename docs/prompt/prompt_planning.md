# `app/prompt/planning.py` — 规划 Agent 提示词

## 文件位置
`app/prompt/planning.py`

## 核心作用
为使用规划功能的 Agent 提供系统提示词，定义结构化计划的创建、执行和跟踪方法。

## 内容

### PLANNING_SYSTEM_PROMPT（17 行）
定义规划专家的角色和工作流程：

1. **分析请求**：理解任务范围
2. **创建计划**：使用 `planning` 工具制定清晰、可执行的计划
3. **执行步骤**：使用可用工具按计划执行
4. **跟踪进度**：必要时调整计划
5. **完成任务**：任务完成时立即使用 `finish` 结束

**工具说明：**
- `planning`：创建、更新、跟踪计划（命令：create、update、mark_step 等）
- `finish`：任务完成时结束

**原则：**
- 将任务分解为逻辑步骤，有明确结果
- 避免过多的细节或子步骤
- 考虑依赖关系和验证方法
- 目标达成后及时结束

### NEXT_STEP_PROMPT（8 行）
下一步行动的决策流程：
1. 计划是否足够，还是需要改进？
2. 能否立即执行下一步？
3. 任务是否完成？完成则立即使用 `finish`

## 使用关系
- 被使用 `PlanningTool` 的 Agent 引用
- 与 `app/tool/planning.py` 中的 `PlanningTool` 配套使用
- 提示词中的命令名与 `PlanningTool` 支持的 7 种命令对齐
- 注意提示词中写的是 `finish` 而非 `terminate`，暗示与该工具协同使用
