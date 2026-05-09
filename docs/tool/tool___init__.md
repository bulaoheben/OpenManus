# `app/tool/__init__.py` — 工具模块导出入口

## 文件位置
`app/tool/__init__.py`

## 核心作用
从各个子模块导入所有公开的工具类，统一导出。提供项目中工具模块的集中导入入口。

## 导出列表

| 导出名 | 来源文件 | 说明 |
|--------|---------|------|
| `BaseTool` | `app/tool/base.py` | 工具抽象基类 |
| `Bash` | `app/tool/bash.py` | Bash 命令执行工具 |
| `BrowserUseTool` | `app/tool/browser_use_tool.py` | 浏览器自动化工具 |
| `Terminate` | `app/tool/terminate.py` | 终止交互工具 |
| `StrReplaceEditor` | `app/tool/str_replace_editor.py` | 文件编辑工具 |
| `WebSearch` | `app/tool/web_search.py` | 网络搜索工具 |
| `ToolCollection` | `app/tool/tool_collection.py` | 工具集合管理 |
| `CreateChatCompletion` | `app/tool/create_chat_completion.py` | 结构化输出工具 |
| `PlanningTool` | `app/tool/planning.py` | 计划管理工具 |
| `Crawl4aiTool` | `app/tool/crawl4ai.py` | 网页爬虫工具 |

## 设计说明
- 未导出 `AskHuman`、`PythonExecute`、`ComputerUseTool`、沙箱工具及图表工具
- 未导出的工具需通过完整路径导入（如 `from app.tool.python_execute import PythonExecute`）
- `__all__` 列表明确定义了公开接口
