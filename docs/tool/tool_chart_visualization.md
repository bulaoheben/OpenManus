# `app/tool/chart_visualization/` — 图表可视化工具集

## 核心作用
实现数据可视化和图表生成能力，包含三个组件：Python 执行器、图表准备、数据可视化。

---

## `__init__.py` — 导出
导出三个类：`DataVisualization`、`VisualizationPrepare`、`NormalPythonExecute`

---

## `python_execute.py` — 标准化 Python 执行

### NormalPythonExecute(PythonExecute)
继承 `PythonExecute`，增加 `code_type` 参数：

| 参数 | 类型 | 枚举值 | 说明 |
|------|------|--------|------|
| `code_type` | `str` | `process`/`report`/`others` | 代码类型 |

- `execute()` 调用父类实现，固定 `timeout=5`
- 描述中提示 LLM 生成丰富的数据分析报告、保存到工作区

---

## `chart_prepare.py` — 图表准备

### VisualizationPrepare(NormalPythonExecute)

| 字段 | 值 |
|------|-----|
| `name` | `"visualization_preparation"` |
| `description` | 为 `data_visualization` 工具生成元数据 |

参数：

| 参数 | 类型 | 枚举值 | 说明 |
|------|------|--------|------|
| `code_type` | `str` | `visualization`/`insight` | 代码类型 |

**可视化类型（visualization）：** 加载数据、清理/转换、保存 CSV、生成图表描述、输出 JSON 信息（格式：`{"csvFilePath": string, "chartTitle": string}[]`）

**洞察类型（insight）：** 从已生成的图表中选择洞察，输出 JSON（格式：`{"chartPath": string, "insights_id": number[]}[]`）

---

## `data_visualization.py` — 数据可视化

### DataVisualization(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"data_visualization"` |
| `description` | 图表生成和洞察添加 |

参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `json_path` | `str` | (required) | JSON 信息文件路径 |
| `output_type` | `str` | `"html"` | 渲染格式（html=交互式 / png） |
| `tool_type` | `str` | `"visualization"` | 类型（visualization/insight） |
| `language` | `str` | `"en"` | 语言（zh/en） |

### data_visualization(json_info, output_type, language)
1. 读取 CSV 文件（`pandas.read_csv`）
2. 转换为 JSON 格式
3. 并行调用 `invoke_vmind()`（每个 CSV 对应一个图表）

### add_insights(json_info, output_type)
1. 读取指定图表的洞察信息
2. 并行调用 `invoke_vmind()` 添加洞察

### invoke_vmind(file_name, output_type, task_type, ...)
核心的 Node.js 图表生成引擎调用：
1. 构建 `vmind_params`（含 LLM 配置、数据集、图表描述等）
2. 启动子进程：`npx ts-node src/chartVisualize.ts`
3. 通过 stdin 传入 JSON 参数
4. 从 stdout 获取图表生成结果
5. 返回 JSON 结果（含 chart_path 或 error）

### 设计模式
- **并行处理**：`asyncio.gather()` 并发生成多个图表
- **子进程调用**：使用 `ts-node` 执行 TypeScript 图表生成引擎（VMind）
- **错误隔离**：单个图表失败不影响其他图表

## 工具调用流程
```
VisualizationPrepare (准备数据 CSV + JSON)
    → DataVisualization (生成图表)
        → VisualizationPrepare (准备洞察 JSON)
            → DataVisualization (添加洞察到图表)
```
