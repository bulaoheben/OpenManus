# `app/utils/files_utils.py` — 文件工具函数

## 文件位置
`app/utils/files_utils.py`

## 核心作用
提供文件操作相关的工具函数，包括路径清理和文件排除判断。

## 排除常量

### `EXCLUDED_FILES` — 排除文件集合
```python
{".DS_Store", ".gitignore", "package-lock.json", "postcss.config.js", ...}
```
包含配置文件、锁文件等不应操作的文件。

### `EXCLUDED_DIRS` — 排除目录集合
```python
{"node_modules", ".next", "dist", "build", ".git"}
```
构建产物和依赖目录。

### `EXCLUDED_EXT` — 排除扩展名集合
```python
{".ico", ".svg", ".png", ".jpg", ".jpeg", ".gif", ".bmp", ".tiff", ".webp", ".db", ".sql"}
```
图片文件和数据库文件。

## 函数说明

### `should_exclude_file(rel_path)` → `bool`
判断文件是否应被排除：

| 检查项 | 逻辑 |
|--------|------|
| 文件名 | `os.path.basename` 是否在 `EXCLUDED_FILES` 中 |
| 目录路径 | `os.path.dirname` 路径中是否包含 `EXCLUDED_DIRS` 中的名称 |
| 文件扩展名 | `os.path.splitext` 后缀是否在 `EXCLUDED_EXT` 中 |

### `clean_path(path, workspace_path="/workspace")` → `str`
清理并规范化路径，使其相对于工作空间：
1. 去除开头 `/`
2. 去除 `workspace_path` 前缀
3. 去除 `workspace/` 前缀
4. 再次去除开头 `/`

```python
clean_path("/workspace/data/file.txt")  → "data/file.txt"
clean_path("workspace/data/file.txt")   → "data/file.txt"
clean_path("/data/file.txt")            → "data/file.txt"
```

## 调用关系
- 被 `app/daytona/tool_base.py` 的 `SandboxToolsBase.clean_path()` 调用
- 被各种文件操作工具用于路径规范化
