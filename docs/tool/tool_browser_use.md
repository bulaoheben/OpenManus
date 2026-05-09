# `app/tool/browser_use_tool.py` — 浏览器自动化工具

[toc]



## 文件位置

`app/tool/browser_use_tool.py`

## 核心作用
提供完整的浏览器自动化能力。基于 `browser-use` 库封装 Playwright，支持导航、点击、输入、滚动、内容提取、标签页管理等 15+ 种动作。维护浏览器会话跨多次调用。

## 类结构

### BrowserUseTool(BaseTool[Context])

| 字段 | 值 |
|------|-----|
| `name` | `"browser_use"` |
| `description` | 15 种浏览器动作的详细说明 |

| 属性 | 类型 | 说明 |
|------|------|------|
| `lock` | `asyncio.Lock` | 防止并发操作浏览器 |
| `browser` | `BrowserUseBrowser` | browser-use 库的浏览器实例 |
| `context` | `BrowserContext` | 浏览器上下文（标签页、Cookie） |
| `dom_service` | `DomService` | DOM 交互服务 |
| `web_search_tool` | `WebSearch` | 内部集成的搜索工具 |
| `llm` | `LLM` | 用于 extract_content 的 LLM 实例 |

### _ensure_browser_initialized()
延迟初始化浏览器，配置来自 `config.browser_config`，支持代理、无头模式、Chromium 实例路径、WSS/CDP 连接。

### execute(action, ...)
15 种动作分 5 类：

**导航类：** `go_to_url`、`go_back`、`refresh`、`web_search`
- `web_search` 组合了 `WebSearch.execute()` + 导航到首个结果

**元素交互类：** `click_element`、`input_text`、`get_dropdown_options`、`select_dropdown_option`
- 使用 browser-use 的 `get_dom_element_by_index(index)` 定位元素

**滚动类：** `scroll_down`、`scroll_up`、`scroll_to_text`
- 通过 `execute_javascript` 执行 `window.scrollBy()`

**内容提取：** `extract_content`
1. `markdownify.markdownify(page.content())` 将 HTML 转 Markdown
2. LLM 通过 tool calling（`tool_choice="required"`）提取结构化内容
3. 使用 `_extract_first_json()` 括号平衡算法解析 LLM 返回的 JSON
4. 支持 max_content_length 截断（默认 2000 字符）

**标签页管理：** `switch_tab`、`open_tab`、`close_tab`

**工具：** `wait`、`send_keys`、`scroll_to_text`

### get_current_state()
获取当前浏览器状态（URL、标题、标签页、可点击元素列表、滚动信息），附带 base64 截图。

### cleanup()
关闭浏览器上下文和浏览器实例，释放资源。

## 已知问题
- `extract_content` 对百度等复杂搜索页效果差（CSS/JS 干扰）
- `web_search` 在 Google 不可达时无浏览器级故障转移
- `_extract_first_json()` 括号平衡算法是对 `json.loads()` 失败的临时方案

## 调用关系
- Agent 通过 ToolCollection 调度
- 内部集成 `WebSearch` 处理 `web_search` 动作
- 使用 `LLM.ask_tool()` 处理 `extract_content`
- 配置来自 `app/config.py` 的 `browser_config`



## 补充：

### 1.app/tool/browser_use_tool.py中browser_use是在哪里定义的？

同样是第三方库，不在 OpenManus 项目内：

  C:\Users\yongy\miniconda3\envs\open_manus\Lib\site-packages\browser_use\__init__.py

  browser-use 是一个专门做 AI Agent 浏览器自动化的 Python 库（基于 Playwright），核心功能包括：

![image-20260509182109299](C:\Users\yongy\AppData\Roaming\Typora\typora-user-images\image-20260509182109299.png)

  项目中所有 browser_use.xxx 或 from browser_use import xxx 导入的都是这个第三方包，不是项目自身代码。
