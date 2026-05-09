# 任务执行失败分析报告 — 视频下载任务

## 目录

[toc]



---

## 1. 概述

**测试任务**：下载爱情公寓第二季第一集到电脑桌面

**Agent**：Manus（`app/agent/manus.py`）

**结果**：任务失败，Agent 在 20 步循环中未能成功下载文件

**失败根因**：4 个问题共同导致 — (1) Google 搜索 ConnectTimeout 导致 web_search 动作失败；(2) extract_content 在百度搜索结果页无法提取有用信息；(3) Bilibili 视频下载需要 premium 会员或 cookie 认证；(4) Agent 达到 max_steps=20 上限时下载尚未完成。

---

## 2. 执行环境

| 项目 | 值 |
|------|-----|
| 模型 | deepseek-chat (api.deepseek.com) |
| max_tokens | 8192 |
| temperature | 0.0 |
| max_steps | 20 (Manus) |
| 浏览器 | 非 headless 模式 (Chrome) |
| 操作系统 | Windows 11 |
| Python | 3.12+ |
| httpx | 0.28.1 |

---

## 3. 任务执行轨迹

Agent 共执行 20 步（达到最大步数限制后停止），以下是完整轨迹：

### 阶段 1：浏览器搜索失败（Step 1-4）

| Step | 动作 | 工具参数 | 结果 |
|------|------|----------|------|
| 1 | web_search | `{"action":"web_search","query":"爱情公寓 第二季 第一集 下载"}` | Google 搜索 ConnectTimeout，web_search 返回 `RetryError[ConnectTimeout]` |
| 2 | go_to_url | `{"action":"go_to_url","url":"https://www.bilibili.com/bangumi/play/ss12021/"}` | 导航成功但页面非目标内容 |
| 3 | go_to_url | `{"action":"go_to_url","url":"https://www.bilibili.com/bangumi/media/md28228333/"}` | 导航到 Bilibili 番剧页 |
| 4 | go_to_url | `{"action":"go_to_url","url":"https://www.baidu.com/s?wd=爱情公寓第二季第一集下载"}` | 导航到百度搜索结果页 |

**结果**：Step 1 WebSearch 调用 Google 搜索失败（ConnectTimeout），此后 Agent 手动导航到 Bilibili 和百度。

### 阶段 2：浏览器提取内容失败（Step 5-7）

| Step | 动作 | 结果 |
|------|------|------|
| 5 | extract_content | LLM 返回"页面内容被截断，主要显示 HTML/CSS/JS 代码而非搜索结果" |
| 6 | scroll_down | 向下滚动 500 像素 |
| 7 | extract_content | LLM 仅返回"百度搜索结果页面中所有可见的搜索结果链接和标题"，无实际链接提取 |

**涉及的 Bug**：extract_content 将完整页面 HTML（含 CSS/JS）传给 LLM，LLM 无法从中提取结构化搜索结果。

### 阶段 3：Python 搜索尝试（Step 8-14）

| Step | 动作 | 结果 |
|------|------|------|
| 8 | python_execute | 打印"尝试搜索可用资源..." |
| 9 | python_execute | 检查依赖库（requests/bs4/yt-dlp 已安装） |
| 10 | python_execute | 用 requests+bs4 爬取百度搜索结果页，但只提取到导航链接（首页/新闻/地图等），无实际搜索结果 |
| 11 | python_execute | 用 yt-dlp 解析 Bilibili 剧集链接 → `ERROR: Unable to extract info` |
| 12 | python_execute | 用 yt-dlp 搜索 `ytsearch10:爱情公寓第二季第一集` → 超时 20 秒 |
| 13 | python_execute | 调用 Bilibili API 搜索 → 412 Precondition Failed（反爬限制） |
| 14 | python_execute | 修复 `import json` 后重试 Bilibili API → 仍然 412 |

**结果**：Python 搜索方式全部失败 — 百度搜索页结构复杂难解析、yt-dlp 对 Bilibili 支持有限、Bilibili API 有反爬机制。

### 阶段 4：浏览器搜索 Bilibili 并找到视频（Step 15-18）

| Step | 动作 | 结果 |
|------|------|------|
| 15 | go_to_url | 导航到 `BV1GW411u7iD` → 页面不存在 |
| 16 | go_to_url | 导航到 `search.bilibili.com/all?keyword=爱情公寓第二季第一集` |
| 17 | extract_content | **成功提取** Bilibili 搜索结果，找到"爱情公寓第二季01"视频（BV1YH4y1H7Z9） |
| 18 | go_to_url | 导航到 `https://www.bilibili.com/video/BV1YH4y1H7Z9/` |

**结果**：extract_content 对 Bilibili 搜索结果页效果良好，成功从 DOM 树提取了视频列表。

### 阶段 5：yt-dlp 尝试下载但失败（Step 19-20）

| Step | 动作 | 结果 |
|------|------|------|
| 19 | python_execute | `yt-dlp --dump-json` → **成功**获取视频元数据（标题、上传者、点赞等） |
| 20 | python_execute | `yt-dlp -o ... --format best[ext=mp4]/best` → **失败**。Bilibili 需要 premium 会员才能下载高清格式，`Requested format is not available` |

**最终结果**：Agent 达到 20 步上限，任务未成功。下载失败的关键是 Bilibili 对视频下载有 premium 认证要求。

---

## 4. 步骤-文件映射表

每条 Agent 动作的执行路径对应到源代码位置：

### 4.1 执行链路

```text
main.py
  └─ agent.run(prompt)                          # app/agent/base.py:116
       └─ step() × 20                           # app/agent/base.py:148
            └─ ReActAgent.step()                # app/agent/react.py
                 ├─ think()                     # app/agent/toolcall.py:39
                 │    └─ llm.ask_tool()         # app/llm.py:644
                 │         ├─ 构建请求参数        # app/llm.py:674-712
                 │         ├─ 调用 API           # app/llm.py:714
                 │         └─ 解析响应            # app/llm.py:716-725
                 └─ act()                       # app/agent/toolcall.py:131
                      └─ execute_tool()          # app/agent/toolcall.py:166
                           └─ available_tools.execute(name, tool_input)
                                                   # app/tool/tool_collection.py
```

### 4.2 工具分发

```text
ToolCollection.execute()
  ├─ name="web_search"       → BrowserUseTool  → _handle_web_search()        # app/tool/browser_use_tool.py:251
  ├─ name="go_to_url"        → BrowserUseTool  → _handle_go_to_url()           # app/tool/browser_use_tool.py:230
  ├─ name="extract_content"  → BrowserUseTool  → _handle_extract_content()     # app/tool/browser_use_tool.py:375
  ├─ name="scroll_down"      → BrowserUseTool  → _handle_scroll_down()         # app/tool/browser_use_tool.py:338
  └─ name="python_execute"   → PythonExecute   → execute()                     # app/tool/python_execute.py:39
```

### 4.3 关键文件职责

| 文件 | 行范围 | 职责 |
|------|--------|------|
| `app/agent/base.py` | 116-154 | 主运行循环，step 计数，状态管理 |
| `app/agent/toolcall.py` | 39-80 | think() — LLM 调用与工具选择 |
| `app/agent/toolcall.py` | 131-155 | act() — 工具执行与结果处理 |
| `app/agent/toolcall.py` | 166-196 | execute_tool() — 单个工具分发 |
| `app/llm.py` | 644-730 | ask_tool() — LLM API 调用含重试 |
| `app/llm.py` | 361-420 | ask() — 普通对话 |
| `app/tool/browser_use_tool.py` | 251-268 | web_search — 搜索+导航到首个结果 |
| `app/tool/browser_use_tool.py` | 375-452 | extract_content — LLM 提取页面内容 |
| `app/tool/python_execute.py` | 39-75 | python_execute — 多进程执行+超时控制 |

---

## 5. 根因分析

### 根因 #1：Google 搜索 ConnectTimeout

**问题**：`web_search` 动作在 Step 1 调用 Google 搜索引擎时发生 `ConnectTimeout`，导致搜索直接失败。这是本次运行与上次运行的主要差异 — 上次 Google 可用但返回百度链接，这次 Google 完全不可用。

**涉及文件**：
- `app/tool/browser_use_tool.py:251-268` — web_search 动作
- `app/tool/web_search.py:290-327` — 搜索引擎故障转移

**影响**：Agent 在 Step 1 就丢失了搜索能力，之后只能靠 LLM 猜测 URL 来手动导航。

### 根因 #2：extract_content 在百度搜索结果页失效

**问题**：`extract_content` 动作将完整页面 HTML 转换为 Markdown 后发给 LLM 提取内容。但百度搜索结果页包含大量 CSS/JS 代码，`markdownify.markdownify()` 转换后仍包含大量无用代码，导致 LLM 无法从中提取到实际的搜索结果链接。

**关键证据**（Step 5 输出）：
```
The page content is truncated and mostly shows HTML/CSS/JavaScript code 
rather than the actual search results... the actual search result links 
are not visible in the provided content.
```

**涉及文件**：`app/tool/browser_use_tool.py:375-452`

**对比**：extract_content 对 Bilibili 搜索结果页效果良好（Step 17 成功提取视频列表），但对百度搜索页基本无效。

### 根因 #3：Bilibili 视频下载需要 premium 认证

**问题**：Agent 在 Step 19 成功通过 `yt-dlp --dump-json` 获取了 Bilibili 视频元数据，但在 Step 20 实际下载时失败。Bilibili 限制非 premium 用户下载高清视频，需要 `--cookies-from-browser` 或 premium 会员认证。

**关键证据**（Step 20 输出）：
```
[BiliBili] Format(s) 4K 超高清, 1080P 60帧, 1080P 高清, 720P 准高清 are missing; 
you have to become a premium member to download them. 
Use --cookies-from-browser or --cookies for the authentication.
ERROR: Requested format is not available.
```

**涉及的预装工具**：`yt-dlp`（系统命令行工具，非项目代码）

**影响**：即使 Agent 成功找到了视频资源，也因认证问题无法下载。

### 根因 #4：Agent 达到 max_steps=20 上限

**问题**：`BaseAgent` 在执行 20 步后自动停止。本次运行中 Agent 在第 19 步才获取到视频元数据，第 20 步尝试下载时失败，没有更多步骤来：
1. 调整 yt-dlp 参数尝试降级格式
2. 尝试其他下载源
3. 处理认证问题

**涉及文件**：`app/agent/base.py:136-154`

```python
while self.current_step < self.max_steps and self.state != AgentState.FINISHED:
    self.current_step += 1
    # ... 执行 step ...
    # 达到 max_steps 后自动停止
```

**影响**：20 步对于需要浏览器搜索 + Python 执行的复杂任务捉襟见肘，尤其是在搜索引擎失败、多步浪费的情况下。

---

## 6. 已修复 Bug 清单

### Bug 1：Pydantic V2 `underscore_attrs_are_private` 已移除

- **文件**：`app/tool/base.py:101`
- **错误**：`ValueError: `underscore_attrs_are_private` is not supported by V2`
- **修复**：移除 `underscore_attrs_are_private = False`，保留 `arbitrary_types_allowed = True`

### Bug 2：httpx surrogate 字符导致 UnicodeEncodeError

- **文件**：`app/llm.py`（ask / ask_tool / ask_with_images 方法）
- **错误**：`'utf-8' codec can't encode characters in position 499-500: surrogates not allowed`
- **根因**：httpx 0.28.1 的 `json_dumps()` 使用 `ensure_ascii=False`，当 LLM 返回的数据中包含 surrogate 字符时，httpx 在传输时调用 `.encode("utf-8")` 失败
- **修复**：新增 `_clean_surrogates()` 函数，在构建请求参数前递归清理字符串中的 surrogate 字符

### Bug 3：extract_content JSON 解析错误

- **文件**：`app/tool/browser_use_tool.py:_extract_first_json`
- **错误**：`json.decoder.JSONDecodeError: Extra data: line 1 column 2510 (char 2509)`
- **根因**：LLM 返回的 JSON 对象后有额外字符（如 Markdown 代码块标记 ```），`json.loads()` 要求完整干净的 JSON
- **修复**：用括号平衡算法替代 `json.loads()`，提取第一个完整的 `{...}` JSON 对象

### Bug 4：python_execute 超时太短

- **文件**：`app/tool/python_execute.py:40`
- **问题**：默认 timeout=5 秒，对于大多数 Python 任务（如安装包、网络请求）太短
- **修复**：默认 timeout 改为 60 秒

### Bug 5：Windows 终端中文日志乱码

- **文件**：`app/logger.py:25-26`
- **问题**：Windows 终端编码为 GBK，loguru 输出 UTF-8 导致中文显示为乱码
- **修复**：在添加 loguru handler 前调用 `sys.stderr.reconfigure(encoding="utf-8")`

---

## 7. 待改进问题

### 7.1 web_search 搜索引擎故障转移优化

`web_search` 动作在 Google 失败时只是抛出异常。应考虑：
- 在浏览器 web_search 动作层面实现引擎级联重试，而不是只依赖 WebSearch 工具内部的故障转移
- 添加连接超时参数配置，避免长时间等待
- 在搜索引擎全失败时，使用浏览器直接导航到搜索引擎页面（如 `www.baidu.com/s?wd=...`）

**涉及文件**：`app/tool/browser_use_tool.py:251-268`

### 7.2 extract_content 增强

extract_content 应考虑：
- 对百度等复杂搜索页面，在提取前预处理 HTML（移除 CSS/JS）
- 在提取失败时降级为获取页面原始文本
- 对不同类型的网站（搜索页、视频页、文章页）使用不同的提取策略
- 增加提取结果的验证机制，确保 LLM 实际提取到了有用的内容

**涉及文件**：`app/tool/browser_use_tool.py:375-452`

### 7.3 yt-dlp 认证支持

Bilibili 视频下载需要 premium 认证。应支持：
- 自动检测 yt-dlp 下载失败的原因（认证、格式不可用等）
- 指导 Agent 使用 `--cookies-from-browser` 参数
- 在提示词中添加处理认证相关错误的策略

**涉及工具**：系统预装的 `yt-dlp`

### 7.4 Agent 步骤管理优化

当前 `max_steps=20` 的限制过于刚性。应考虑：
- 动态调整 max_steps：当 Agent 正在执行关键任务（如下载）时，允许延长步骤数
- 添加步骤预算管理：让 Agent 意识到还剩多少步，优先执行高价值操作
- 在 Agent 提示词中加入剩余步数信息

**涉及文件**：`app/agent/base.py:136-154`

### 7.5 python_execute 架构重构

当前 `multiprocessing.Process` + `Manager()` 的架构：
- 不支持持久化后台任务
- Manager 的 with 块退出会连带 kill 子进程
- 应考虑使用 `subprocess.Popen` 或 asyncio subprocess 替代

**涉及文件**：`app/tool/python_execute.py`

### 7.6 extract_content JSON 解析优化

当前使用括号平衡算法替代 `json.loads()` 是一种临时方案。更健壮的做法是：
- 使用 `json.JSONDecoder.raw_decode()` 标准方法
- 或在系统提示词中要求 LLM 返回纯 JSON（不加 Markdown 代码块）

**涉及文件**：`app/tool/browser_use_tool.py`

---

## 8. 本次运行统计数据（2026-05-09）

| 指标 | 值 |
|------|-----|
| 总步数 | 20（达上限） |
| 总耗时 | ~4.5 分钟 |
| 总输入 Token | 126,100 |
| 总输出 Token | 3,454 |
| 总计 Token | 129,554 |
| 平均输入/步 | 6,305 |
| 平均输出/步 | 173 |
| 执行工具次数 | 20 次（browser_use=10, python_execute=10） |

### Token 消耗趋势

步数越往后，输入 Token 越大（因为消息历史不断增长）：
- Step 1：1,977 tokens（输入）
- Step 10：5,530 tokens
- Step 15：8,002 tokens
- Step 20：10,655 tokens

### 各工具调用统计

| 工具 | 调用次数 | 成功 | 失败 |
|------|---------|------|------|
| browser_use → web_search | 1 | 0 | 1（ConnectTimeout） |
| browser_use → go_to_url | 6 | 6 | 0 |
| browser_use → extract_content | 3 | 2（Bilibili 有效, 百度无效） | 1（百度无结果） |
| browser_use → scroll_down | 1 | 1 | 0 |
| python_execute | 9 | 8 | 1（缺少 json import） |
