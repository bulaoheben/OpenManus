# `app/agent/data_analysis.py` — 数据分析 Agent

## 文件位置
`app/agent/data_analysis.py`

## 核心作用
专注数据分析和图表可视化任务的 Agent。提供 Python 执行、图表准备和数据可视化等数据分析专用工具。

## 类结构

### DataAnalysis(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"Data_Analysis"` | |
| `description` | 利用 Python 和数据可视化工具解决多种数据分析任务 | |
| `max_observe` | 15000 | 观察结果截断长度（比其他 Agent 大） |
| `max_steps` | 20 | |
| `available_tools` | `NormalPythonExecute, VisualizationPrepare, DataVisualization, Terminate` | 4 个数据分析专用工具 |

**提示词来源：**
- `system_prompt`：来自 `app.prompt.visualization.SYSTEM_PROMPT`，格式化为 `directory=config.workspace_root`
- `next_step_prompt`：来自 `app.prompt.visualization.NEXT_STEP_PROMPT`

## 设计说明
- 不重写 `think()` 或 `act()`，完全使用 `ToolCallAgent` 的默认行为
- `max_observe = 15000` 比默认值大，适应数据分析结果长度
- 推荐工作流：
  1. `NormalPythonExecute` — 数据处理和分析
  2. `VisualizationPrepare` — 准备图表数据
  3. `DataVisualization` — 生成可视化图表
