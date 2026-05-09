# `app/tool/str_replace_editor.py` — 文件编辑工具

## 文件位置
`app/tool/str_replace_editor.py`

## 核心作用
提供文件查看、创建、编辑的能力。支持 5 种命令（view、create、str_replace、insert、undo_edit），状态跨调用持久化。支持本地和沙盒两种文件系统后端。

## 类结构

### StrReplaceEditor(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"str_replace_editor"` |
| `description` | 文件编辑工具，状态持久化，含 cat -n 显示格式 |

| 属性 | 说明 |
|------|------|
| `_file_history` | 文件内容历史（用于 undo） |
| `_local_operator` | 本地文件操作器 |
| `_sandbox_operator` | 沙盒文件操作器 |

参数（7 个）：

| 参数 | 类型 | 说明 |
|------|------|------|
| `command` | enum | `view`/`create`/`str_replace`/`insert`/`undo_edit` |
| `path` | str | 文件或目录的绝对路径 |
| `file_text` | str | `create` 命令的创建内容 |
| `old_str` | str | `str_replace` 的旧字符串 |
| `new_str` | str | `str_replace` 的新字符串 / `insert` 的插入内容 |
| `insert_line` | int | `insert` 命令的插入行号 |
| `view_range` | [int, int] | `view` 命令的行范围 |

### execute() 执行流程
1. 根据 `config.sandbox.use_sandbox` 选择文件操作器（`_get_operator()`）
2. `validate_path()` 验证：路径必须为绝对路径、非 create 命令需路径存在、create 需路径不存在
3. 分发到对应命令处理函数

### 命令详解

**view：** 目录→ `find -maxdepth 2` 列出内容；文件→ `cat -n` 格式显示，支持行范围
**create：** 写入文件内容，保存到 history
**str_replace：** 精确匹配替换，要求 old_str 唯一（否则报错列出冲突行号），编辑后显示 snippet
**insert：** 在指定行后插入文本，显示编辑前后 snippet
**undo_edit：** 从 history 恢复上一次修改前的文件内容

### _make_output()
格式化输出：每行加行号前缀 `{line:6}\t{content}`，超 16000 字符截断。

## 调用关系
- 通过 `app/tool/file_operators.py` 中的 `FileOperator` 接口执行实际文件操作
- 根据 `config.sandbox.use_sandbox` 自动选择本地或沙盒模式
- `_file_history` 使用 `defaultdict(list)` 管理每个文件的编辑历史
