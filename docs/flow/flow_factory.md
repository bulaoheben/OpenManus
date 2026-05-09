# `app/flow/flow_factory.py` — Flow 工厂

## 文件位置
`app/flow/flow_factory.py`

## 核心作用
根据指定的 Flow 类型创建对应的 Flow 实例，封装流创建逻辑。

## 类结构

### FlowType(str, Enum)
当前仅支持一种 Flow 类型：

| 枚举值 | 字符串 | 说明 |
|--------|--------|------|
| `PLANNING` | `"planning"` | 计划式执行流 |

### FlowFactory
提供静态工厂方法 `create_flow()`：

```python
@staticmethod
def create_flow(flow_type, agents, **kwargs) -> BaseFlow
```

**参数：**
| 参数 | 类型 | 说明 |
|------|------|------|
| `flow_type` | `FlowType` | 要创建的 Flow 类型 |
| `agents` | `BaseAgent / List[BaseAgent] / Dict[str, BaseAgent]` | Agent 或 Agent 列表 |

**流程：**
1. 在 `flows` 字典中查找对应的 Flow 类
2. 未找到则抛出 `ValueError`
3. 找到则创建实例并返回

## 扩展方式
新增 Flow 类型只需：
1. 在 `FlowType` 中添加新枚举值
2. 实现继承 `BaseFlow` 的新 Flow 类
3. 在 `flows` 字典中添加映射

## 调用关系
- 在 `main.py` 中调用 `FlowFactory.create_flow()` 创建 Flow
- 当前仅支持 `PLANNING` 类型，映射到 `PlanningFlow`
