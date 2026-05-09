# `app/tool/ask_human.py` — 人工咨询工具

[toc]

## 文件位置

`app/tool/ask_human.py`

## 核心作用
允许 Agent 在遇到无法自行解决的问题时，向人类用户发送询问并获取输入。

## 类结构

### AskHuman(BaseTool)

| 字段 | 值 |
|------|-----|
| `name` | `"ask_human"` |
| `description` | 遇到困难时向人类求助 |

参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `inquire` | `str` | 要向人类提出的问题 |

### execute(inquire)
```python
return input(f"Bot: {inquire}\n\nYou: ").strip()
```

直接调用 Python 内置的 `input()` 函数，在终端输出 Agent 的问题并等待用户输入。

## 设计说明
- 使用同步 `input()` 调用（非异步），可能阻塞事件循环
- 工具描述中建议 Agent 仅在"无法继续"时使用
- 在非交互式环境下（如自动化脚本）会永久阻塞



# 补充：

## BaseTool 是什么？

**BaseTool = 所有 AI 工具的 “父类 / 模板 / 规范”**

你可以把它理解成：

- 所有工具（执行命令、搜索、画图、提问…）**必须按这个模板来写**
- 它规定了：**必须有 name、description、parameters、execute ()**
- AI 系统才能**统一识别、统一调用**所有工具

它不是你写的，是**框架提供的基类**，来自：

```python
from app.tool import BaseTool
```

### 3. parameters（参数描述）

- 告诉 AI **调用这个工具需要传什么参数**

- 必须是 **JSON Schema 格式**

- parameters 里面存的是什么？（超级重要）

  - **parameters = 告诉 AI：调用这个工具需要什么参数、参数是什么意思、哪些必填**

    它是固定的 **JSON Schema 格式**，AI 模型（GPT/Claude）能直接看懂。

    ```
    parameters: str = {
        "type": "object",
        "properties": {
            "inquire": {
                "type": "string",
                "description": "The question you want to ask human.",
            }
        },
        "required": ["inquire"],  # 这个参数必填
    }
    ```

    意思是：

    - 调用 `ask_human` 工具
    - **必须传一个叫 inquire 的字符串参数**
    - 它的意思是：要问人类的问题
    - 不填就报错

### 4. execute 方法（执行逻辑）

- 工具真正的功能代码
- 必须是 **async 异步函数**
- 输入参数要和 parameters 里定义的对应

```
async def execute(self, inquire: str) -> str:
```

------

- execute 方法做了什么？

```
async def execute(self, inquire: str) -> str:
    return input(f"""Bot: {inquire}\n\nYou: """).strip()
```

功能：

- 接收 AI 提出的问题 `inquire`
- 在终端打印：`Bot: xxx`
- 等待你输入回答
- 把你的回答返回给 AI

这就是 **AI 与人类交互的逻辑**。

------



# 七、最精简总结（必看）

## 1. BaseTool 是什么？

**所有 AI 工具的基类，统一规范。**

## 2. 必须强制定义的 4 个东西

- name
- description
- parameters
- async execute()

## 3. parameters 存什么？

**工具的参数格式：参数名、类型、描述、是否必填**（AI 能看懂的格式）

## 4.ask_human.py中的BaseTool是在哪里定义的？./app/tool下没有BaseTool文件啊，还是说只要是该目录下定义的 class，无论哪个文件里面都可以直接引用？

在 Python 项目中，一个包下面的 __init__.py 文件有什么用？它一般是怎么写的？

我用**最通俗、最实用、新手一看就懂**的方式给你讲清楚：**`__init__.py` 是 Python 包的 “身份证” + “入口文件”**。

------

一、一句话核心作用

**只要文件夹里有 `__init__.py`，Python 就把它当成一个「包（Package）」**，可以被其他文件 `import`。

没有它 → 只是普通文件夹有它 → 变成 Python 包

------

二、`__init__.py` 到底有什么用？（4 大核心功能）

1. 标记 “这是一个包”

这是最原始的作用。Python 看到这个文件，就知道：**哦，这是一个可以导入的模块包**。

2. 控制 `import *` 导入哪些内容

```python
__all__ = ["BaseTool", "AskHuman"]
```

表示：`from app.tool import *`只会导入 `BaseTool` 和 `AskHuman`。

3. **暴露内部类 / 函数，让外部导入更简单**

这是你看到的 **OpenManus / OpenClaw 里最常用的写法**！

```plaintext
app/
  tool/
    __init__.py
    base.py       # 里面有 BaseTool
    ask_human.py  # 里面有 AskHuman
```

在 `__init__.py` 里写：

```python
from .base import BaseTool
from .ask_human import AskHuman
```

外部就可以直接写：

```python
from app.tool import BaseTool, AskHuman
```

**不用写 from app.tool.base import BaseTool**

这就是你代码里：

```python
from app.tool import BaseTool
```

**能直接导入的原因！**

4. 初始化包的配置、版本、日志

```python
__version__ = "1.0.0"
```

------

三、它一般怎么写？（3 种最常见写法）

写法 1：空文件（最简单）

```python
# 什么都不写
```

只要标记这是包。

------

写法 2：导出类 / 函数（最常用！你看到的框架都这么写）

```python
# app/tool/__init__.py

from .base import BaseTool
from .ask_human import AskHuman

__all__ = ["BaseTool", "AskHuman"]
```

作用：**让外部导入更简洁**。

------

写法 3：加版本 / 初始化

```python
__version__ = "0.1.0"

from .base import BaseTool
```

------

四、为什么你代码里能写：

```python
from app.tool import BaseTool
```

就是因为**`app/tool/__init__.py` 里面做了这件事**：

```python
from .base import BaseTool
```

这是 Python 项目**标准规范**，所有框架（Django、Flask、OpenManus、OpenClaw）都这么干。

------

五、超级总结（新手必背）

1. **`__init__.py` = 包的身份证**
2. 有它 → 能 `import`
3. 里面写 `from .xxx import 类` → 让外部导入更简洁
4. `__all__` = 控制 `import *` 导入内容
5. **你看到的 BaseTool 能直接导入，全靠 \**init\**.py 暴露**
