# `app/prompt/swe.py` — SWE Agent 提示词

## 文件位置
`app/prompt/swe.py`

## 核心作用
为 `SWEAgent`（`app/agent/swe.py`）提供系统提示词，定义自主编程任务的交互规范。

## 内容

### SYSTEM_PROMPT（22 行）
定义自主程序员的角色和工作规范：

**关键规则：**
1. **命令环境**：在命令行中使用特殊文件编辑器界面（每窗口显示 `{{WINDOW}}` 行）
2. **编辑器命令**：除 bash 外，还可使用特定文件编辑命令
3. **缩进要求**：编辑命令要求正确缩进，空格必须精确写出
4. **响应格式**：shell 提示格式为 `(Open file: <path>)\n(Current directory: <cwd>)\nbash-$`
5. **单工具调用**：每条消息只能包含 **一个** 工具调用，然后等待 shell 响应
6. **禁止交互式命令**：不支持交互式会话（如 python、vim）

**设计特点：**
- SWE-agent 风格，专为代码修改和命令行任务优化
- 强调单步执行、等待反馈的模式
- 所有输出会被保存供后续参考

## 使用关系
- 被 `app/agent/swe.py` 的 `SWEAgent` 使用
- `next_step_prompt` 为空字符串，完全依赖 `SYSTEM_PROMPT` 引导行为
- 内置 `{{WINDOW}}` 双花括号占位符（Jinja2 风格），但代码中未替换
- 不包含 `NEXT_STEP_PROMPT` 定义
