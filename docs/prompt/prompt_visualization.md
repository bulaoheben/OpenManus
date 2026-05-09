# `app/prompt/visualization.py` — 数据分析 Agent 提示词

## 文件位置
`app/prompt/visualization.py`

## 核心作用
为 `DataAnalysis` Agent（`app/agent/data_analysis.py`）提供系统提示词，定义数据分析和可视化任务的规范。

## 内容

### SYSTEM_PROMPT（5 行）
```python
"""
You are an AI agent designed to data analysis / visualization task.
# Note:
1. The workspace directory is: {directory}; Read / write file in workspace
2. Generate analysis conclusion report in the end
"""
```

**关键点：**
- 专注于数据分析和可视化任务
- `{directory}` 占位符在初始化时被 `config.workspace_root` 替换
- 要求在最后生成分析结论报告

### NEXT_STEP_PROMPT（10 行）
- 根据用户需求分解问题
- **每次只选一个最合适的工具**
- 每个工具执行后解释结果并提出下一步
- 观察到 Error 时审查并修复

## 使用关系
- 被 `app/agent/data_analysis.py` 的 `DataAnalysis` 使用
- `SYSTEM_PROMPT.format(directory=config.workspace_root)` 初始化
- 强调每次只用一个工具（`ONLY ONE`），避免并行操作
