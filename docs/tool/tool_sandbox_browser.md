# `app/tool/sandbox/sb_browser_tool.py` — 沙盒浏览器工具

## 文件位置
`app/tool/sandbox/sb_browser_tool.py`

## 核心作用
在 Daytona 沙盒环境中通过 REST API + curl 控制浏览器。与 `BrowserUseTool` 不同，此工具不直接使用 Playwright，而是通过沙盒内部的自动化服务操作浏览器。

## 类结构

### SandboxBrowserTool(SandboxToolsBase)

| 字段 | 值 |
|------|-----|
| `name` | `"sandbox_browser"` |
| `description` | 沙盒浏览器自动化工具 |

| 属性 | 说明 |
|------|------|
| `browser_message` | 存储最后一次浏览器状态消息 |

支持的 15 种动作：`navigate_to`、`go_back`、`wait`、`click_element`、`input_text`、`send_keys`、`switch_tab`、`close_tab`、`scroll_down`、`scroll_up`、`scroll_to_text`、`get_dropdown_options`、`select_dropdown_option`、`click_coordinates`、`drag_drop`

### _execute_browser_action(endpoint, params)
核心执行方法，通过 curl 命令调用沙盒内部自动化服务（`http://localhost:8003/api/automation/{endpoint}`）：
1. 构造 curl 命令（POST/GET JSON）
2. `sandbox.process.exec(curl_cmd, timeout=30)` 执行
3. 解析 JSON 响应
4. 验证截图（`_validate_base64_image()`）
5. 构造 `ThreadMessage` 存储浏览器状态

### _validate_base64_image()
多层验证：非空检查、base64 字符检查、长度模 4、解码验证、文件大小限制（10MB）、PIL 图像格式验证（JPEG/PNG/GIF/BMP/WEBP/TIFF）、尺寸限制（8192x8192）。

### get_current_state()
从 `browser_message` 获取上次浏览器状态（URL、标题、标签页、滚动信息），可选附带截图。

### create_with_sandbox(sandbox)
工厂方法。

## 与 BrowserUseTool 的对比

| 特性 | BrowserUseTool | SandboxBrowserTool |
|------|---------------|-------------------|
| 环境 | 本地 Playwright | 沙盒内自动化服务 |
| 通信方式 | 直接 Python API | curl + REST API |
| 浏览器引擎 | browser-use + Playwright | 沙盒内浏览器 |
| 内容提取 | LLM + markdownify | 依赖 API 返回 |
| 截图 | Playwright API | API 返回 base64 |
