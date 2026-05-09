# `app/tool/search/` — 搜索引擎实现（Google / Baidu / Bing / DuckDuckGo）

## 核心作用
四个搜索引擎的具体实现，每个文件对应一个引擎，统一继承 `WebSearchEngine`，返回 `List[SearchItem]`。

---

## Google — `google_search.py`

### GoogleSearchEngine
依赖库：`googlesearch`（`from googlesearch import search`）

```python
raw_results = search(query, num_results=num_results, advanced=True)
```

- 使用 `googlesearch` 库（非官方 API，基于网页抓取）
- `advanced=True` 返回含 title/url/description 的对象
- 兼容纯字符串返回（仅 URL）

---

## Baidu — `baidu_search.py`

### BaiduSearchEngine
依赖库：`baidusearch`（`from baidusearch.baidusearch import search`）

```python
raw_results = search(query, num_results=num_results)
```

- 使用 `baidusearch` 库
- 处理三种返回格式：字符串、字典（title/url/abstract）、对象属性
- 综合兼容不同版本的 `baidusearch` 库

---

## Bing — `bing_search.py`

### BingSearchEngine
依赖库：`requests` + `BeautifulSoup`（lxml）

**特点：**
- 自行实现 HTML 解析，无需第三方搜索库
- 使用 `requests.Session` 维护请求头（含 Referer、User-Agent 轮换）
- 分页处理：解析 `ol#b_results` → `li.b_algo`，提取 h2 标题、URL、p 摘要
- 自动翻页直到达到 `num_results` 数量
- 摘要截断：300 字符上限
- 11 个 User-Agent 轮换（Chrome、Googlebot、Chromium 等）

### 核心方法
- `_search_sync(query, num_results)`：主搜索流程，循环翻页
- `_parse_html(url)`：解析 Bing 搜索结果 HTML，返回 `(results, next_url)`

---

## DuckDuckGo — `duckduckgo_search.py`

### DuckDuckGoSearchEngine
依赖库：`duckduckgo-search`（`from duckduckgo_search import DDGS`）

```python
raw_results = DDGS().text(query, max_results=num_results)
```

- 使用 `DDGS().text()` 方法搜索
- 处理三种返回格式：字符串、字典（title/href/body）、对象属性
- 无需 API key，无频率限制

---

## 对比总结

| 引擎 | 依赖库 | 实现方式 | 是否需要 API Key | 稳定性 |
|------|--------|---------|------------------|--------|
| Google | `googlesearch` | 第三方库封装 | 否 | 中（可能被限流） |
| Baidu | `baidusearch` | 第三方库封装 | 否 | 中 |
| Bing | requests+bs4 | 自行解析 HTML | 否 | 较高 |
| DuckDuckGo | `duckduckgo-search` | 第三方库封装 | 否 | 较高 |
