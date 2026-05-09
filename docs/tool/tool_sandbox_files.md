# `app/tool/sandbox/sb_files_tool.py` — 沙盒文件管理工具

## 文件位置
`app/tool/sandbox/sb_files_tool.py`

## 核心作用
在 Daytona 沙盒环境中执行文件操作（创建、读取、更新、删除），所有操作相对于 `/workspace` 目录隔离执行。

## 类结构

### SandboxFilesTool(SandboxToolsBase)

| 字段 | 值 |
|------|-----|
| `name` | `"sandbox_files"` |
| `description` | 沙盒文件系统操作 |

参数（4 种 action）：

| action | 必填参数 | 说明 |
|--------|---------|------|
| `create_file` | `file_path`, `file_contents` | 创建新文件 |
| `str_replace` | `file_path`, `old_str`, `new_str` | 字符串替换 |
| `full_file_rewrite` | `file_path`, `file_contents` | 完整重写文件 |
| `delete_file` | `file_path` | 删除文件 |

### 方法详解

**_create_file()：**
1. `clean_path()` 规范化路径
2. 检查文件是否已存在（存在则拒绝）
3. 自动创建父目录
4. 上传文件 + 设置权限
5. **特殊逻辑**：自动检测 `index.html` 并返回 HTTP 服务器预览链接（8080 端口）

**_str_replace()：**
- 要求 old_str 唯一（同 `StrReplaceEditor` 的逻辑）
- 显示替换位置附近的 snippet

**_full_file_rewrite()：**
- 要求文件已存在
- 也包含 index.html 自动检测逻辑

**_delete_file()：**
- 删除操作，文件不存在时返回错误

### get_workspace_state()
遍历 `/workspace` 所有文件，读取内容并返回状态字典（带排除规则：通过 `should_exclude_file()` 过滤）。

### clean_path()
使用 `app.utils.files_utils.clean_path` 规范化路径。

## 调用关系
- 继承 `SandboxToolsBase`（`app.daytona.tool_base`）
- 通过 `sandbox.fs` API 执行实际文件操作
- 对比本地的 `StrReplaceEditor`：功能类似但运行在沙盒环境中
