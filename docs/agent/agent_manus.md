# `app/agent/manus.py` — Manus 通用 Agent

[toc]

## 文件位置

`app/agent/manus.py`

## 核心作用
系统的默认通用 Agent，组合了最常用的工具集（浏览器、Python 执行、文件编辑、人工咨询），并集成了 MCP 服务器支持。

## 类结构

### Manus(ToolCallAgent)

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `"Manus"` | |
| `max_observe` | 10000 | 观察结果最大长度 |
| `max_steps` | 20 | 默认最大步数 |
| `available_tools` | `PythonExecute, BrowserUseTool, StrReplaceEditor, AskHuman, Terminate` | 通用工具集 |
| `mcp_clients` | `MCPClients()` | MCP 客户端 |
| `connected_servers` | `dict` | 已连接的 MCP 服务器 |

### create() — 工厂方法
```python
@classmethod
async def create(cls, **kwargs) -> "Manus":
    instance = cls(**kwargs)
    await instance.initialize_mcp_servers()
    instance._initialized = True
    return instance
```
异步创建并初始化 MCP 连接。

(

**`@classmethod`**：这是**类方法**，不用创建对象就能调用

**`async def`**：这是**异步方法**，里面可以 await

**目的**：解决 **Python 构造函数 `__init__` 不能是异步** 的问题！

### 为什么要这么写？（最重要！）

#### 因为：

### **Python 的 `__init__` 构造方法 CANNOT 是异步的！**

你**不能**写：

```
# ❌ 错误！__init__ 不能加 async
async def __init__(self):
    await self.connect()
```

所以必须用 **`@classmethod + async`** 来做**真正完整的初始化**。

### 这个方法怎么用？（一看就懂）

**普通创建（不推荐，没初始化完）**

```
manus = Manus()
# 此时服务器还没启动，不能用！
```

**正确创建（用这个 create 方法）**

```
manus = await Manus.create()
# ✅ 服务器已启动，完全就绪，可以直接用！
```

### initialize_mcp_servers()
遍历 `config.mcp_config.servers` 配置，连接到所有 MCP 服务器（支持 SSE 和 stdio 两种传输协议）。

### connect_mcp_server()
连接到单个 MCP 服务器，将服务器提供的工具添加到 `available_tools`。

### disconnect_mcp_server()
断开 MCP 服务器连接，清理工具集合（移除 MCPClientTool 实例）。

### think() — 思考阶段（重写）
在标准 `think()` 基础上增加了浏览器状态感知：
1. 检查最近 3 条消息是否使用了 `BrowserUseTool`
2. 如果是，则通过 `BrowserContextHelper` 获取当前浏览器状态
3. 将浏览器状态信息注入 `next_step_prompt`
4. 临时替换 prompt → 调用 `super().think()` → 恢复 prompt

### cleanup()
清理浏览器和 MCP 连接资源。

## 调用关系
- `main.py` 中通过 `Manus.create()` 初始化并运行
- 使用 `BrowserContextHelper` 管理浏览器状态信息
- MCP 服务器配置来自 `config.mcp_config`



## python基础知识：

这个方法里的 CLS 是什么意思？“async def create(cls, **kwargs) -> "Manus":”为什么它要返回"Manus"这个字符串？ ？

#### 一、`cls` 是什么？

**`cls` = 当前这个类（Manus 类本身）**

------

**详细解释**

你写的是 **@classmethod（类方法）**类方法的第一个参数永远是 **`cls`**，代表：**调用这个方法的 类本身**

```
@classmethod
async def create(cls, **kwargs) -> "Manus":
```

如果你这样调用：

```
manus = await Manus.create()
```

那么方法里的：

- **`cls` = `Manus`**

------

**它和 `self` 的区别**

- `self` = **实例对象**
- `cls` = **类本身**

```
# 普通方法：操作对象
def func(self):

# 类方法：操作类
@classmethod
def func(cls):
```

------

#### 二、为什么 `cls` 在这里？

因为这行代码需要它：

```
instance = cls(**kwargs)
```

等于：

```
instance = Manus(**kwargs)
```

**用 `cls` 的好处：**

- 子类继承时，自动指向子类，不用改代码
- 更规范、更通用

------

#### 三、为什么返回 `"Manus"` 字符串？

**重点：**它不是返回字符串！它是注解：告诉编辑器 “我返回的是 Manus 类型”！

```
async def create(cls, **kwargs) -> "Manus":
```

`-> "Manus"` 的意思：**这个方法返回的类型是 Manus**

------

**为什么加引号**？

因为 **类还没定义完**！

```
class Manus:
    async def create(cls, **kwargs) -> "Manus":
```

**Python 执行到这里时，Manus 类还没结束定义**

所以不能直接写 `Manus`，必须加引号 `"Manus"`。

这叫：**字符串类型注解（Forward Reference）**

------

#### 四、最终翻译

```
async def create(cls, **kwargs) -> "Manus":
```

**人话翻译：**

> 这是一个异步类方法，
>
> 第一个参数 cls 代表 Manus 类，
>
> 接收任意关键字参数，
>
> **返回值是 Manus 类型的对象**

------

#### 五、超级总结（必背）

1. **`cls` = 当前类（Manus 类本身）**
2. **`cls()` = 创建当前类的实例**
3. **`-> "Manus"` = 标注返回类型是 Manus**
4. **加引号 = 因为类还没定义完，提前引用**

------

最简单记住：

- **self = 对象**
- **cls = 类**
- **-> "Manus" = 我返回 Manus 类型
