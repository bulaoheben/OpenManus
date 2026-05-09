# `app/flow/base.py` — Flow 抽象基类

## 文件位置
`app/flow/base.py`

## 核心作用
定义执行流（Flow）的抽象基类，支持多 Agent 编排。Flow 不同于 Agent——Agent 负责单步骤的思考-行动循环，Flow 负责协调多个 Agent 按流程执行任务。

## 类结构

### BaseFlow(BaseModel, ABC)

| 属性 | 类型 | 说明 |
|------|------|------|
| `agents` | `Dict[str, BaseAgent]` | Agent 字典，key 为名称，value 为 Agent 实例 |
| `tools` | `Optional[List]` | 可选的全局工具列表 |
| `primary_agent_key` | `Optional[str]` | 主 Agent 的 key |

### __init__(agents, **data)
灵活的构造函数，支持 3 种 Agent 输入格式：

| 输入类型 | 转换结果 | 使用场景 |
|---------|---------|---------|
| `BaseAgent`（单个） | `{"default": agent}` | 单 Agent 流程 |
| `List[BaseAgent]` | `{"agent_0": a0, "agent_1": a1, ...}` | 多 Agent，自动编号 |
| `Dict[str, BaseAgent]` | 原样使用 | 多 Agent，命名方式 |

如果未指定 `primary_agent_key`，自动使用第一个 Agent 作为主 Agent。

### 属性与方法

| 方法 | 说明 |
|------|------|
| `primary_agent` | 返回 `primary_agent_key` 对应的 Agent |
| `get_agent(key)` | 按 key 获取 Agent |
| `add_agent(key, agent)` | 添加 Agent |
| `execute(input_text)` | **抽象方法**，执行流程 |

## 设计模式
- **模板方法模式**：子类实现 `execute()` 定义具体的多 Agent 执行流程
- **工厂模式**：通过 `FlowFactory` 创建具体的 Flow 实例
- 支持单 Agent 和多 Agent 两种模式
- 使用 Pydantic 的 `arbitrary_types_allowed = True` 配置

## 调用关系
- 被 `app/flow/flow_factory.py` 的 `FlowFactory` 用于创建具体 Flow
- 被 `app/flow/planning.py` 的 `PlanningFlow` 继承实现
- 底层使用 `app/agent/base.py` 的 `BaseAgent` 执行任务
