# `app/agent/base.py` — Agent 抽象基类

## 文件位置
`app/agent/base.py`

## 核心作用
定义所有 Agent 的抽象基类 `BaseAgent`，提供状态管理、记忆管理、基于步骤的执行循环等基础设施。

## 类结构

### BaseAgent (BaseModel, ABC)

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | `str` | (required) | Agent 名称 |
| `description` | `Optional[str]` | None | Agent 描述 |
| `system_prompt` | `Optional[str]` | None | 系统级提示词 |
| `next_step_prompt` | `Optional[str]` | None | 下一步行动提示词 |
| `llm` | `LLM` | `LLM()` | LLM 实例 |
| `memory` | `Memory` | `Memory()` | 记忆存储 |
| `state` | `AgentState` | `IDLE` | 当前 Agent 状态 |
| `max_steps` | `int` | 10 | 最大执行步数 |
| `current_step` | `int` | 0 | 当前执行步数 |
| `duplicate_threshold` | `int` | 2 | 重复内容检测阈值 |

### 关键方法

**run(request=None)** — 主执行循环：
1. 检查状态是否为 `IDLE`，否则抛出 `RuntimeError`
2. 可选添加用户请求到记忆
3. 进入 `RUNNING` 状态，循环执行 `step()` 直到达到 `max_steps` 或状态变为 `FINISHED`
4. 每次迭代 `current_step += 1`
5. 检测卡住状态（`is_stuck()`）并处理
6. 超过 `max_steps` 后重置状态为 `IDLE`
7. 最后执行 `SANDBOX_CLIENT.cleanup()`

**step()** — 抽象方法，子类必须实现

**update_memory(role, content, base64_image)** — 添加消息到记忆：
- 支持 4 种角色：`user`、`system`、`assistant`、`tool`
- 使用 `message_map` 字典分发到 `Message` 的不同工厂方法

**is_stuck()** — 检测死循环：
- 倒序遍历最近消息，统计与最后一条消息内容相同的 `assistant` 消息数
- 超过 `duplicate_threshold`（默认 2）则判定为卡住

**handle_stuck_state()** — 卡住时添加提示词，引导 LLM 改变策略

**state_context(new_state)** — 异步上下文管理器，安全的状态转换：
- 进入时设置新状态，退出时恢复原状态
- 异常时转换为 `ERROR` 状态

### 配置说明
- `arbitrary_types_allowed = True`
- `extra = "allow"` 允许子类添加额外字段

## 继承层级
```
BaseAgent (抽象基类)
  └─ ReActAgent (think + act 循环)
       └─ ToolCallAgent (工具调用)
            ├─ Manus (通用 Agent)
            ├─ BrowserAgent (浏览器 Agent)
            ├─ MCPAgent (MCP Agent)
            ├─ SWEAgent (代码 Agent)
            ├─ SandboxManus (沙盒 Agent)
            └─ DataAnalysis (数据分析 Agent)
```
