# `app/prompt/browser.py` — 浏览器 Agent 提示词

## 文件位置
`app/prompt/browser.py`

## 核心作用
为浏览器自动化 Agent 提供详细的系统提示词，定义浏览器交互的完整规则。这是所有提示词中最长、最详细的（72 行）。

## 内容

### SYSTEM_PROMPT（71 行）
完整的浏览器自动化规则集，分为 9 个部分：

**1. 输入格式：**
- Task、Previous steps、Current URL、Open Tabs、Interactive Elements
- 元素格式：`[index]<type>text</type>`，只有带数字索引 [] 的元素是可交互的

**2. 响应格式（JSON）：**
强制要求 LLM 以固定 JSON 格式响应：
```json
{
  "current_state": {
    "evaluation_previous_goal": "Success|Failed|Unknown",
    "memory": "已完成和待完成的计数",
    "next_goal": "下一步立即行动"
  },
  "action": [{"action_name": {参数}}]
}
```

**3. 动作执行：**
- 可指定多个动作序列执行
- 页面变化后序列中断，获取新状态
- 高效链式操作（如填写表单）

**4. 元素交互：**
- 仅使用可交互元素的索引
- 标记为 `[]Non-interactive text` 的是不可交互的

**5. 导航与错误处理：**
- 无合适元素时使用其他功能
- 卡住时尝试替代方案（后退、新搜索、新标签）
- 处理弹窗/Cookie
- 使用滚动查找元素
- 遇到验证码尝试解决或换方法

**6. 任务完成：**
- 最后动作使用 `done`
- 到达最后一步时即使未完成也使用 `done` 并提供所有信息
- 对重复性任务在 memory 中计数

**7. 视觉上下文：**
- 使用图片理解页面布局
- 边界框对应元素索引

**8. 长任务：**
- 在 memory 中跟踪状态和子结果

**9. 内容提取：**
- 查找信息时在具体页面调用 `extract_content`

### NEXT_STEP_PROMPT（22 行）
格式化模板，包含占位符：
- `{url_placeholder}` — 当前 URL 和页面标题
- `{tabs_placeholder}` — 可用标签页
- `{content_above_placeholder}` / `{content_below_placeholder}` — 视口外内容
- `{results_placeholder}` — 动作结果或错误

包含常见的 browser-use 动作速查：
- `go_to_url`、`click_element`、`input_text`、`extract_content`、`scroll_down/up`

## 使用关系
- `SYSTEM_PROMPT` 被 `app/agent/browser.py` 的 `BrowserAgent` 使用
- `NEXT_STEP_PROMPT` 被 `BrowserContextHelper.format_next_step_prompt()` 动态格式化后注入 Agent 的 `next_step_prompt`
- 提示词中的动作名与 `BrowserUseTool`（`app/tool/browser_use_tool.py`）支持的 15 种动作对齐
- 该提示词借鉴了 `browser-use` 库的设计，要求 LLM 输出严格的 JSON 格式
