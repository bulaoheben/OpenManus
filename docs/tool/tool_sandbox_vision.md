# `app/tool/sandbox/sb_vision_tool.py` — 沙盒视觉工具

## 文件位置
`app/tool/sandbox/sb_vision_tool.py`

## 核心作用
允许 Agent 读取沙盒内的图片文件，支持压缩和 base64 编码，供 LLM 视觉处理。

## 类结构

### SandboxVisionTool(SandboxToolsBase)

| 字段 | 值 |
|------|-----|
| `name` | `"sandbox_vision"` |
| `description` | 视觉工具，读取沙盒内的图片文件 |

唯一的 action：`see_image`

参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `file_path` | `str` | 图片在 `/workspace` 下的相对路径 |

### execute(action, file_path)
处理流程：
1. **路径清理**：`clean_path()` 规范化
2. **文件检查**：验证路径存在且为文件
3. **大小检查**：最大 10MB
4. **MIME 类型检测**：支持 `image/jpeg`、`image/png`、`image/gif`、`image/webp`
5. **图片压缩**（`compress_image()`）：
   - 处理 RGBA/LA/P 模式（填充白色背景）
   - 缩放至最大 1920x1080（保持宽高比）
   - 格式优化：GIF（optimize）、PNG（compress_level=6）、JPEG（quality=85）
6. **大小验证**：压缩后不超过 5MB
7. **返回结果**：`ToolResult` 带 `base64_image`

## 设计考虑
- 透明背景图填充白色（RGBA→RGB）避免视觉失真
- 两级大小限制（原图 10MB / 压缩后 5MB）
- 支持 PIL 支持的所有常见图片格式
