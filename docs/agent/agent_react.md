# `app/agent/react.py` — ReAct (思考-行动) 循环 Agent

## 文件位置
`app/agent/react.py`

## 核心作用
实现 ReAct（Reasoning + Acting）模式的 Agent 基类。将每一步拆分为 `think()`（决定做什么）和 `act()`（执行行动）两个阶段。

## 类结构

### ReActAgent(BaseAgent, ABC)

继承 `BaseAgent`，重新声明了部分属性以提供默认值：

| 属性 | 默认值 |
|------|--------|
| `llm` | `LLM()` |
| `memory` | `Memory()` |
| `state` | `AgentState.IDLE` |
| `max_steps` | 10 |
| `current_step` | 0 |

### step() — 单步执行
```python
async def step(self) -> str:
    should_act = await self.think()
    if not should_act:
        return "Thinking complete - no action needed"
    return await self.act()
```

**核心逻辑：**
1. 调用 `think()` 判断是否需要行动
2. 如果 `think()` 返回 `False`，跳过行动直接返回
3. 否则调用 `act()` 执行已决定的行动

### think() — 抽象方法
子类必须实现，处理当前状态并决定下一步行动，返回 `bool` 表示是否需要执行行动。

### act() — 抽象方法
子类必须实现，执行已决定的行动并返回结果字符串。

## 设计模式
- **模板方法模式**：`step()` 定义"思考→行动"固定流程，`think()` 和 `act()` 由子类实现
- `ReActAgent` 本身也是抽象类，不提供 `think()` 和 `act()` 的实现
- `ToolCallAgent` 是第一个提供了完整 `think()` + `act()` 实现的子类
