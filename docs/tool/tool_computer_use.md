# `app/tool/computer_use_tool.py` — 桌面自动化工具

## 文件位置
`app/tool/computer_use_tool.py`

## 核心作用
通过 Daytona 沙盒的自动化服务 API 控制桌面环境。支持鼠标、键盘、截图等 11 种动作，维护鼠标位置状态。

## 类结构

### ComputerUseTool(SandboxToolsBase)

| 字段 | 值 |
|------|-----|
| `name` | `"computer_use"` |
| `description` | 桌面自动化工具（鼠标、键盘、截图） |

| 属性 | 说明 |
|------|------|
| `mouse_x` / `mouse_y` | 当前鼠标位置（跨调用维护） |
| `api_base_url` | 沙盒自动化服务 URL |
| `session` | `aiohttp.ClientSession` |

支持的 11 种动作：`move_to`、`click`、`scroll`、`typing`、`press`、`wait`、`mouse_down`、`mouse_up`、`drag_to`、`hotkey`、`screenshot`

**按键支持：** 87 个预定义按键（字母、数字、功能键、组合键如 ctrl+c、alt+tab）
**鼠标按钮：** left、right、middle

### execute(action, ...)
通过 `_api_request()` 调用沙盒 REST API（`{api_base_url}/api/automation/...`）：

| 动作 | API 端点 | 说明 |
|------|---------|------|
| move_to | `/mouse/move` | 移动鼠标到 (x, y) |
| click | `/mouse/click` | 点击（支持 clicks 次数、button 选择） |
| scroll | `/mouse/scroll` | 滚动（-10~10） |
| typing | `/keyboard/write` | 输入文本 |
| press | `/keyboard/press` | 按单个键 |
| mouse_down | `/mouse/down` | 按下鼠标按钮 |
| mouse_up | `/mouse/up` | 释放鼠标按钮 |
| drag_to | `/mouse/drag` | 拖拽到目标坐标 |
| hotkey | `/keyboard/hotkey` | 组合键 |
| screenshot | `/screenshot` | 截图并保存到本地文件 |

**screenshot 动作特性：** 返回 base64_image 并保存到 `screenshots/` 目录（带时间戳 + `latest_screenshot.png`）。

### _api_request(method, endpoint, data)
使用 `aiohttp.ClientSession` 发送 GET/POST 请求，异常时返回 `{"success": false, "error": ...}`。

### create_with_sandbox(sandbox)
工厂方法，通过沙箱创建工具实例。

## 调用关系
- 继承 `app.daytona.tool_base.SandboxToolsBase`
- 依赖 Daytona 沙盒的自动化 REST API（端口 8000）
- 使用 `aiohttp` 进行 HTTP 通信
