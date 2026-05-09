# `app/tool/web_search.py` — 网络搜索工具

## 文件位置
`app/tool/web_search.py`

## 核心作用
提供多引擎、带故障转移的网络搜索能力。支持 Google、Baidu、DuckDuckGo、Bing 四个搜索引擎，自动按优先级顺序尝试，全部失败后可重试。

## 类结构

### SearchResult (Pydantic BaseModel)
单个搜索结果的结构化表示：

| 字段 | 类型 | 说明 |
|------|------|------|
| `position` | `int` | 结果排名 |
| `url` | `str` | 结果 URL |
| `title` | `str` | 结果标题 |
| `description` | `str` | 结果描述/摘要 |
| `source` | `str` | 搜索引擎名称 |
| `raw_content` | `Optional[str]` | 可选的页面原始内容 |

### SearchMetadata
搜索元数据：`total_results`、`language`、`country`。

### SearchResponse(ToolResult)
继承 `ToolResult` 的结构化搜索响应，包含 `query`、`results`、`metadata`。`populate_output` validator 将结果自动格式化为易读文本。

### WebContentFetcher
工具类，使用 `requests` + `BeautifulSoup` 从搜索结果页面提取正文内容：
- 移除 script/style/header/footer/nav 元素
- 限制 10000 字符
- 10 秒超时

### WebSearch(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"web_search"` |
| `description` | 实时信息搜索，自动故障转移 |

| 属性 | 说明 |
|------|------|
| `_search_engine` | 4 个搜索引擎实例的字典 |
| `content_fetcher` | `WebContentFetcher` 实例 |

参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `query` | `str` | (required) | 搜索关键词 |
| `num_results` | `int` | 5 | 返回结果数量 |
| `lang` | `str` | `"en"` | 语言代码 |
| `country` | `str` | `"us"` | 国家代码 |
| `fetch_content` | `bool` | `false` | 是否获取页面全文 |

### execute() 执行流程
1. 从 config 获取 retry_delay、max_retries、lang、country
2. `_try_all_engines()` 按配置顺序尝试（preferred → fallbacks → 其余引擎）
3. 成功后可选 `_fetch_content_for_results()` 获取页面全文
4. 全部引擎失败则等待 retry_delay 秒后重试，最多 max_retries 次
5. 始终返回 `SearchResponse`（成功或错误）

### _get_engine_order()
搜索顺序：配置的 preferred 引擎 → fallback 列表 → 剩余引擎。

### _perform_search_with_engine()
使用 `tenacity.retry` 装饰器（最多 3 次，指数退避），在 `run_in_executor` 中同步执行搜索引擎的 `perform_search`。

## 调用关系
- `BrowserUseTool` 的 `web_search` 动作内部使用 `WebSearch.execute()`
- Agent 也可直接调用 `WebSearch` 工具
- 依赖 `app/tool/search/` 下的 4 个搜索引擎实现
- 配置来自 `config.search_config`
