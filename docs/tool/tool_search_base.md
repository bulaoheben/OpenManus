# `app/tool/search/__init__.py` & `base.py` — 搜索引擎接口

## 文件位置
`app/tool/search/__init__.py`、`app/tool/search/base.py`

## 核心作用
定义搜索引擎的抽象接口和数据结构，统一所有搜索引擎的实现规范。

---

## `__init__.py` — 导出入口
导出所有搜索引擎类：
- `WebSearchEngine`（抽象基类）
- `GoogleSearchEngine`
- `BaiduSearchEngine`
- `BingSearchEngine`
- `DuckDuckGoSearchEngine`

---

## `base.py` — 基础模型与抽象基类

### SearchItem (Pydantic BaseModel)
单个搜索结果的通用数据模型：

| 字段 | 类型 | 说明 |
|------|------|------|
| `title` | `str` | 结果标题 |
| `url` | `str` | 结果 URL |
| `description` | `Optional[str]` | 结果摘要/描述 |

`__str__` 格式：`"{title} - {url}"`

### WebSearchEngine (Pydantic BaseModel)
搜索引擎的抽象基类，定义接口：

```python
def perform_search(self, query, num_results=10, *args, **kwargs) -> List[SearchItem]
```

子类必须实现此方法，返回 `SearchItem` 列表。

## 设计模式
- **策略模式**：`WebSearchEngine` 定义策略接口，每个搜索引擎实现具体策略
- **组合模式**：`WebSearch` 持有多个引擎实例，按顺序尝试
- 新增搜索引擎只需继承 `WebSearchEngine` 并实现 `perform_search`
