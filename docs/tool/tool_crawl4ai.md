# `app/tool/crawl4ai.py` — Crawl4AI 网页爬虫工具

## 文件位置
`app/tool/crawl4ai.py`

## 核心作用
集成 Crawl4AI 库提供高性能网页爬取，支持 JavaScript 渲染、Markdown 内容提取，适用于 LLM 和 AI Agent 的数据收集。

## 类结构

### Crawl4aiTool(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"crawl4ai"` |
| `description` | Crawl4AI 驱动的网页爬虫，提取清洁的 AI 可用内容 |

参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `urls` | `Union[str, List[str]]` | (required) | 要爬取的 URL 列表 |
| `timeout` | `int` | 30 | 每个 URL 的超时（秒），范围 5-120 |
| `bypass_cache` | `bool` | false | 是否绕过缓存 |
| `word_count_threshold` | `int` | 10 | 内容块的最小词数 |

### execute() 执行流程
1. **URL 验证**：使用 `urlparse` 检查 scheme 和 netloc，仅允许 http/https
2. **配置 Crawl4AI**：`BrowserConfig`（headless chromium）+ `CrawlerRunConfig`（缓存、iFrame、弹窗移除等）
3. **异步爬取**：逐个 URL 调用 `AsyncWebCrawler.arun()`，统计词数、链接数、图片数
4. **结果汇总**：JSON 格式，包含成功/失败状态、状态码、标题、Markdown 内容、元数据

### 特点
- 支持 JS 密集网站（`java_script_enabled=True`）
- 自动处理 HTTPS 错误
- 排除 script/style 标签
- 进度日志（成功/失败标记）

### _is_valid_url(url)
URL 格式验证，确保 scheme 为 http/https 且包含 netloc。

## 调用关系
- 通过 `__init__.py` 导出，Agent 可使用
- 依赖 `crawl4ai` 第三方库（需预装）
- 与 `BrowserUseTool` 互补：前者用于交互式浏览，后者用于批量内容提取
