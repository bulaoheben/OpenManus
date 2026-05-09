# `app/daytona/tool_base.py` — Daytona 沙箱工具基类

## 文件位置
`app/daytona/tool_base.py`

## 核心作用
定义 `SandboxToolsBase` 基类，为所有基于 Daytona 沙箱的工具提供统一的沙箱访问管理。同时包含 `ThreadMessage` 数据类。

## 辅助数据类

### `ThreadMessage(dataclass)`
表示要添加到线程的消息：

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `str` | 必填 | 消息类型 |
| `content` | `Dict[str, Any]` | 必填 | 消息内容 |
| `is_llm_message` | `bool` | `False` | 是否为 LLM 生成的消息 |
| `metadata` | `Optional[Dict]` | `None` | 附加元数据 |
| `timestamp` | `float` | `datetime.now()` | 时间戳 |

提供 `to_dict()` 方法转换为字典用于 API 调用。

## 类结构

### `SandboxToolsBase(BaseTool)`
所有沙箱工具的基类，提供项目级别的沙箱访问。

| 属性 | 类型 | 说明 |
|------|------|------|
| `project_id` | `Optional[str]` | 项目 ID |
| `_sandbox` | `Optional[Sandbox]` | 沙箱实例（私有） |
| `_sandbox_id` | `Optional[str]` | 沙箱 ID（私有） |
| `_sandbox_pass` | `Optional[str]` | 沙箱密码（私有） |
| `workspace_path` | `str` | 工作空间路径，默认 `"/workspace"` |
| `_sessions` | `dict[str, str]` | 会话字典（私有） |
| `_urls_printed` | `ClassVar[bool]` | 类级标记，控制 VNC/Website URL 是否已打印 |

### 关键方法

**`_ensure_sandbox()` — 确保沙箱就绪**
1. `_sandbox` 为 `None` 时：调用 `create_sandbox()` 创建新沙箱，首次创建时打印 VNC URL（6080）和 Website URL（8080）
2. `_sandbox` 已存在但状态为 `ARCHIVED`/`STOPPED`：调用 `daytona.start()` 重启并执行 `start_supervisord_session()`

**`sandbox` property — 获取沙箱实例**
确保 `_sandbox` 已初始化，否则抛出 `RuntimeError`。

**`sandbox_id` property — 获取沙箱 ID**
确保 `_sandbox_id` 已初始化，否则抛出 `RuntimeError`。

**`clean_path(path)` — 路径规范化**
调用 `files_utils.clean_path()` 将路径规范化到 `/workspace` 相对路径。

### Pydantic 配置
```python
class Config:
    arbitrary_types_allowed = True
    underscore_attrs_are_private = True
```

## 调用关系
- 继承 `app/tool/base.py` 的 `BaseTool`
- 使用 `app/daytona/sandbox.py` 的 `create_sandbox()` 和 `start_supervisord_session()`
- 使用 `app/utils/files_utils.py` 的 `clean_path()`
- 基于第三方 `daytona` SDK 的 `Daytona`、`Sandbox`、`SandboxState`
- 子类包括 `SandboxBrowserTool`、`SandboxFileOperator`、`SandboxShellTool`、`SandboxVisionTool` 等
