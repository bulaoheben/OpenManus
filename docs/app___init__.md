# `app/__init__.py` — 应用包入口

## 文件位置
`app/__init__.py`

## 核心作用
在包加载时检查 Python 版本兼容性，确保运行环境在 3.11~3.13 之间。

## 逻辑说明
```python
if sys.version_info < (3, 11) or sys.version_info > (3, 13):
    print("Warning: Unsupported Python version ...")
```
- 版本低于 3.11 或高于 3.13 时打印警告
- 不会阻断程序执行，仅作提醒
