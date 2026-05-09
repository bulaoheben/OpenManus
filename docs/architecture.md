# OpenManus 项目架构文档

## 目录

1. [项目概述](#1-项目概述)
2. [项目结构](#2-项目结构)
3. [核心架构](#3-核心架构)
4. [Agent 体系](#4-agent-体系)
5. [工具系统](#5-工具系统)
6. [LLM 集成](#6-llm-集成)
7. [Flow 执行流](#7-flow-执行流)
8. [沙箱系统](#8-沙箱系统)
9. [配置系统](#9-配置系统)
10. [执行入口](#10-执行入口)
11. [运行方式](#11-运行方式)

---

## 1. 项目概述

OpenManus 是一个基于大语言模型（LLM）的多功能 AI Agent 框架，能够通过多种工具解决各类复杂任务。项目采用 Python 3.12+ 开发，支持多种 LLM 后端（OpenAI、Azure、AWS Bedrock、Ollama、DeepSeek 等），并集成了 MCP（Model Context Protocol）协议、浏览器自动化、代码执行、文件操作、网页搜索等功能。

**核心特性：**
- 多 Agent 架构，支持不同任务类型的专用 Agent
- 丰富的工具系统（代码执行、浏览器控制、文件编辑、网页搜索等）
- 支持 MCP 协议，可动态扩展工具
- 支持计划流（Planning Flow）多步骤任务执行
- 支持 Docker 沙箱和 Daytona 云沙箱两种隔离执行环境
- 支持 AWS Bedrock 推理
- 支持多搜索引擎（Google、Baidu、DuckDuckGo、Bing）

---

## 2. 项目结构

```
OpenManus/
├── main.py                        # 主入口（Manus Agent）
├── run_flow.py                    # Flow 执行入口（计划流模式）
├── run_mcp.py                     # MCP Agent 运行入口
├── run_mcp_server.py              # MCP 服务器运行入口
├── sandbox_main.py                # 沙箱模式入口（SandboxManus）
├── setup.py                       # 包安装配置
├── requirements.txt               # 依赖清单
├── config/
│   ├── config.toml                # 全局配置文件
│   ├── config.example.toml        # 配置示例
│   ├── mcp.json                   # MCP 服务器连接配置
│   └── config.example-*.toml      # 各 LLM 提供商配置示例
├── app/
│   ├── config.py                  # 配置加载（单例模式）
│   ├── schema.py                  # 核心数据模型
│   ├── llm.py                     # LLM 客户端封装
│   ├── bedrock.py                 # AWS Bedrock 适配器
│   ├── logger.py                  # 日志系统
│   ├── exceptions.py              # 异常定义
│   ├── agent/                     # Agent 层
│   │   ├── base.py                # BaseAgent 抽象基类
│   │   ├── react.py               # ReActAgent（Think-Act 循环）
│   │   ├── toolcall.py            # ToolCallAgent（工具调用）
│   │   ├── manus.py               # Manus Agent（主 Agent）
│   │   ├── browser.py             # Browser Agent + BrowserContextHelper
│   │   ├── swe.py                 # SWE Agent（编程任务）
│   │   ├── mcp.py                 # MCP Agent（MCP 工具）
│   │   ├── data_analysis.py       # Data Analysis Agent
│   │   └── sandbox_agent.py       # SandboxManus（沙箱版）
│   ├── tool/                      # 工具层
│   │   ├── base.py                # BaseTool + ToolResult
│   │   ├── tool_collection.py     # ToolCollection 工具集合
│   │   ├── terminate.py           # Terminate 终止工具
│   │   ├── bash.py                # Bash 执行
│   │   ├── python_execute.py      # Python 代码执行
│   │   ├── browser_use_tool.py    # 浏览器自动化
│   │   ├── str_replace_editor.py  # 文件编辑
│   │   ├── web_search.py          # 网页搜索
│   │   ├── file_operators.py      # 文件操作（本地/沙箱）
│   │   ├── planning.py            # 计划管理
│   │   ├── create_chat_completion.py # 结构化输出
│   │   ├── crawl4ai.py            # 网页爬虫
│   │   ├── ask_human.py           # 询问人类
│   │   ├── computer_use_tool.py   # 桌面自动化
│   │   ├── mcp.py                 # MCP 客户端
│   │   ├── search/                # 搜索引擎
│   │   │   ├── base.py            # 搜索引擎基类
│   │   │   ├── google_search.py   # Google
│   │   │   ├── baidu_search.py    # 百度
│   │   │   ├── duckduckgo_search.py # DuckDuckGo
│   │   │   └── bing_search.py     # Bing
│   │   ├── sandbox/               # 沙箱工具
│   │   │   ├── sb_browser_tool.py # 沙箱浏览器
│   │   │   ├── sb_files_tool.py   # 沙箱文件操作
│   │   │   ├── sb_shell_tool.py   # 沙箱命令执行
│   │   │   └── sb_vision_tool.py  # 沙箱视觉
│   │   └── chart_visualization/   # 图表可视化
│   ├── flow/                      # 执行流层
│   │   ├── base.py                # BaseFlow 抽象基类
│   │   ├── planning.py            # PlanningFlow 计划执行流
│   │   └── flow_factory.py        # FlowFactory 工厂
│   ├── prompt/                    # 提示词模板
│   │   ├── manus.py               # Manus Agent 提示词
│   │   ├── toolcall.py            # ToolCall 提示词
│   │   ├── browser.py             # Browser 提示词
│   │   ├── swe.py                 # SWE 提示词
│   │   ├── mcp.py                 # MCP 提示词
│   │   ├── planning.py            # Planning 提示词
│   │   └── visualization.py       # 数据可视化提示词
│   ├── sandbox/                   # Docker 沙箱
│   │   ├── client.py              # 沙箱客户端接口
│   │   └── core/
│   │       ├── sandbox.py         # DockerSandbox 实现
│   │       ├── manager.py         # SandboxManager 管理器
│   │       ├── terminal.py        # 异步终端
│   │       └── exceptions.py      # 沙箱异常
│   ├── daytona/                   # Daytona 云沙箱
│   │   ├── sandbox.py             # Daytona 沙箱管理
│   │   └── tool_base.py           # 沙箱工具基类
│   ├── mcp/                       # MCP 服务端
│   │   └── server.py              # MCP Server
│   └── utils/                     # 工具函数
│       ├── files_utils.py         # 文件路径处理
│       └── logger.py              # 日志工具
├── protocol/a2a/                  # A2A 协议支持
├── tests/                         # 测试
├── examples/                      # 使用示例
├── workspace/                     # 工作区目录
└── logs/                          # 日志目录
```

---

## 3. 核心架构

### 3.1 层次架构

```
┌──────────────────────────────────────────────┐
│              入口层 (Entry Points)             │
│  main.py  run_flow.py  run_mcp.py  sandbox.py │
├──────────────────────────────────────────────┤
│              Flow 执行流层                     │
│  BaseFlow → PlanningFlow                      │
├──────────────────────────────────────────────┤
│              Agent 层                          │
│  BaseAgent → ReActAgent → ToolCallAgent       │
│    ├── Manus (主 Agent)                       │
│    ├── BrowserAgent (浏览器)                   │
│    ├── SWEAgent (编程)                         │
│    ├── MCPAgent (MCP 协议)                    │
│    ├── DataAnalysis (数据分析)                  │
│    └── SandboxManus (沙箱)                    │
├──────────────────────────────────────────────┤
│              工具层 (Tools)                    │
│  BaseTool → ToolCollection                    │
│    ├── 代码执行 (PythonExecute, Bash)          │
│    ├── 浏览器 (BrowserUseTool)                 │
│    ├── 文件编辑 (StrReplaceEditor)             │
│    ├── 搜索 (WebSearch)                        │
│    ├── 终止 (Terminate)                        │
│    └── MCP / 沙箱工具                          │
├──────────────────────────────────────────────┤
│              LLM 层                            │
│  LLM (OpenAI / Azure / Bedrock / Ollama)       │
├──────────────────────────────────────────────┤
│           基础设施 (Config, Schema, Logger)     │
└──────────────────────────────────────────────┘
```

### 3.2 核心数据模型 (`app/schema.py`)

```python
class AgentState(str, Enum):
    IDLE      # 空闲状态
    RUNNING   # 运行中
    FINISHED  # 已完成
    ERROR     # 错误状态

class Role(str, Enum):
    SYSTEM      # 系统消息
    USER        # 用户消息
    ASSISTANT   # 助手消息
    TOOL        # 工具消息

class Message(BaseModel):
    role          # 角色
    content       # 内容
    tool_calls    # 工具调用
    name          # 名称
    tool_call_id  # 工具调用 ID
    base64_image  # 图片

class Memory(BaseModel):
    messages      # 消息列表（最大 100 条）
```

---

## 4. Agent 体系

### 4.1 继承层次

```
BaseAgent                    # 抽象基类：状态管理、记忆存储、执行循环
  └── ReActAgent             # Think-Act 循环模式
        └── ToolCallAgent    # 工具调用代理：think→act 驱动
              ├── Manus          # 通用主 Agent（默认入口）
              ├── BrowserAgent   # 浏览器自动化 Agent
              ├── SWEAgent       # 编程任务 Agent
              ├── MCPAgent       # MCP 协议 Agent
              ├── DataAnalysis   # 数据分析 Agent
              └── SandboxManus   # 沙箱版本 Manus
```

### 4.2 BaseAgent (`app/agent/base.py`)

所有 Agent 的抽象基类，提供：
- **状态管理**：通过 `AgentState`（IDLE/RUNNING/FINISHED/ERROR）和 `state_context` 上下文管理器实现安全状态转换
- **记忆系统**：`Memory` 对象存储消息历史，上限 100 条
- **执行循环**：`run()` 方法执行主循环，每次迭代调用 `step()`，最多 `max_steps` 步
- **卡死检测**：检测重复响应，自动注入策略变更提示
- **资源清理**：执行结束后调用 `SANDBOX_CLIENT.cleanup()`

关键方法：
```python
async def run(request: str) -> str    # 执行主循环
async def step() -> str               # 单步执行（抽象方法）
def update_memory(role, content)      # 更新记忆
def is_stuck() -> bool                # 卡死检测
def handle_stuck_state()              # 卡死处理
```

### 4.3 ReActAgent (`app/agent/react.py`)

实现经典的 **ReAct（Reasoning + Acting）** 模式：
```python
async def step() -> str:       # 每步先 think 后 act
    should_act = await think()
    if not should_act: return
    return await act()

async def think() -> bool      # 抽象：思考决策
async def act() -> str         # 抽象：执行动作
```

### 4.4 ToolCallAgent (`app/agent/toolcall.py`)

核心工具调用 Agent，继承 ReActAgent，实现了完整的工具调用流程：

- **`think()`**: 调用 LLM 的 `ask_tool()` 方法，解析返回的工具调用指令
- **`act()`**: 顺序执行所有工具调用，处理执行结果
- **`execute_tool()`**: 单个工具执行，含参数解析、错误处理、特殊工具处理
- **工具选择策略**：支持 `AUTO` / `NONE` / `REQUIRED` 三种模式

### 4.5 Manus (`app/agent/manus.py`)

**主 Agent**，也是 `main.py` 默认使用的 Agent。继承 ToolCallAgent，拥有最丰富的工具集：

- **默认工具**：PythonExecute, BrowserUseTool, StrReplaceEditor, AskHuman, Terminate
- **MCP 支持**：自动加载配置的 MCP 服务器工具
- **浏览器上下文**：BrowserContextHelper 自动注入当前浏览器状态
- **生命周期管理**：`create()` 工厂方法初始化 MCP 连接，`cleanup()` 释放资源

### 4.6 其他 Agent

| Agent | 用途 | 特有工具 |
|-------|------|---------|
| **BrowserAgent** | 纯浏览器自动化任务 | BrowserUseTool, Terminate |
| **SWEAgent** | 编程/代码任务 | Bash, StrReplaceEditor, Terminate |
| **MCPAgent** | 通过 MCP 协议连接外部工具服务器 | MCPClients 动态工具 |
| **DataAnalysis** | 数据分析与可视化 | PythonExecute, VisualizationPrepare, DataVisualization, Terminate |
| **SandboxManus** | Daytona 云沙箱中的完整环境 | SandboxBrowserTool, SandboxFilesTool, SandboxShellTool, SandboxVisionTool |

---

## 5. 工具系统

### 5.1 工具基类 (`app/tool/base.py`)

```python
class BaseTool(BaseModel):
    name: str                           # 工具名称
    description: str                    # 工具描述（给 LLM 看的）
    parameters: dict                    # OpenAI Function Calling 参数 schema

    async def execute(**kwargs)         # 抽象执行方法
    def to_param() -> Dict              # 转成 OpenAI function calling 格式
```

### 5.2 工具集合 (`app/tool/tool_collection.py`)

```python
class ToolCollection:
    def __init__(*tools: BaseTool)      # 初始化工具列表
    def to_params()                     # 全部转成参数列表给 LLM
    async def execute(name, tool_input) # 按名称执行工具
    def add_tool(tool)                  # 动态添加工具
    def add_tools(*tools)              # 批量添加
```

### 5.3 完整工具清单

| 工具名 | 文件 | 功能 |
|--------|------|------|
| **PythonExecute** | `python_execute.py` | 多进程安全执行 Python 代码，超时控制 |
| **Bash** | `bash.py` | 交互式 Bash 会话，支持长命令后台运行 |
| **BrowserUseTool** | `browser_use_tool.py` | 基于 `browser-use` 库的浏览器自动化（导航、点击、输入、截图、提取内容等 15 种操作） |
| **StrReplaceEditor** | `str_replace_editor.py` | 文件查看/创建/替换编辑/插入/撤销编辑 |
| **WebSearch** | `web_search.py` | 多引擎网页搜索，自动故障转移 |
| **Terminate** | `terminate.py` | 终止交互，标记成功/失败 |
| **CreateChatCompletion** | `create_chat_completion.py` | 结构化输出，支持 Pydantic 模型 |
| **PlanningTool** | `planning.py` | 计划创建/更新/标记状态/查询 |
| **Crawl4aiTool** | `crawl4ai.py` | 基于 Crawl4AI 的网页爬虫，输出 Markdown |
| **AskHuman** | `ask_human.py` | 向用户提问获取输入 |
| **ComputerUseTool** | `computer_use_tool.py` | 沙箱桌面自动化（鼠标/键盘/截图） |
| **MCPClientTool** | `mcp.py` | MCP 服务器远程工具代理 |
| **SandboxBrowserTool** | `sandbox/sb_browser_tool.py` | 沙箱内浏览器操作 |
| **SandboxFilesTool** | `sandbox/sb_files_tool.py` | 沙箱内文件操作 |
| **SandboxShellTool** | `sandbox/sb_shell_tool.py` | 沙箱内 Shell 执行 |
| **SandboxVisionTool** | `sandbox/sb_vision_tool.py` | 沙箱内视觉/截图分析 |

### 5.4 搜索引擎架构

```
WebSearchEngine (base.py)           # 搜索引擎抽象接口
  ├── GoogleSearchEngine            # Google 搜索
  ├── BaiduSearchEngine             # 百度搜索
  ├── DuckDuckGoSearchEngine        # DuckDuckGo 搜索
  └── BingSearchEngine              # Bing 搜索
```

WebSearch 工具按照配置依次尝试各引擎，失败时自动故障转移，支持重试机制。

---

## 6. LLM 集成

### 6.1 LLM 客户端 (`app/llm.py`)

支持以下 LLM 后端（通过 `api_type` 区分）：
- **OpenAI**：`AsyncOpenAI` 客户端（默认），兼容 Ollama、DeepSeek 等
- **Azure OpenAI**：`AsyncAzureOpenAI` 客户端
- **AWS Bedrock**：`BedrockClient` 适配器

核心方法：
```python
async def ask(messages, stream=True) -> str              # 普通对话
async def ask_with_images(messages, images) -> str        # 多模态对话
async def ask_tool(messages, tools, tool_choice) -> str   # 工具调用
```

**重试策略**：使用 `tenacity` 实现指数退避重试（最多 6 次），TokenLimitExceeded 不重试。

**Token 管理**：
- `TokenCounter` 精确计算文本/图片的 token 消耗
- 支持设置 `max_input_tokens` 限制累积输入 token
- 流式/非流式两种请求模式

### 6.2 AWS Bedrock 适配器 (`app/bedrock.py`)

通过 `boto3` 调用 AWS Bedrock 的 `converse` API，实现 OpenAI 格式的消息转换（`_convert_openai_messages_to_bedrock_format`）和响应转换（`_convert_bedrock_response_to_openai_format`），使 Bedrock 模型能够无缝替代 OpenAI API。

---

## 7. Flow 执行流

### 7.1 架构

```
BaseFlow (abstract)
  └── PlanningFlow      # 计划驱动的多步骤执行
```

### 7.2 PlanningFlow (`app/flow/planning.py`)

多 Agent 协作的计划执行流：

1. **创建计划**：调用 LLM 生成结构化计划，包含步骤列表和 Agent 分配
2. **步骤执行**：循环执行：获取当前步骤 → 分配 Agent → 执行 → 标记完成
3. **完成总结**：全部步骤完成后，调用 LLM 生成总结报告

**计划步骤状态**：`not_started` → `in_progress` → `completed` / `blocked`

使用 `run_flow.py` 启动，可通过配置开启数据分析 Agent 协作。

---

## 8. 沙箱系统

### 8.1 Docker 沙箱 (`app/sandbox/`)

基于 Docker 的隔离执行环境：

```
DockerSandbox                  # 单个沙箱容器
  ├── 容器创建与管理            # 资源限制（内存/CPU/网络）
  ├── 命令执行                  # 异步终端（AsyncDockerizedTerminal）
  ├── 文件操作                  # 读写、复制（tar 流）
  └── 生命周期管理              # 创建/清理

SandboxManager                 # 沙箱管理器
  ├── 多沙箱管理                # 上限 100 个
  ├── 并发控制                  # 异步锁
  ├── 闲置清理                  # 自动回收超时沙箱
  └── 全局清理                  # 关闭时全部清理
```

### 8.2 Daytona 云沙箱 (`app/daytona/`)

基于 Daytona 云服务的远程沙箱环境，提供完整桌面体验：
- 自动创建配置了 Chrome 浏览器、VNC 服务的沙箱镜像
- 支持 VNC 远程桌面访问
- 提供沙箱内浏览器、文件、Shell、视觉、桌面自动化工具

### 8.3 文件操作抽象

```
FileOperator (Protocol)         # 文件操作接口
  ├── LocalFileOperator         # 本地文件系统实现
  └── SandboxFileOperator       # 沙箱文件系统实现
```

---

## 9. 配置系统

### 9.1 架构

单例模式的配置管理器（`app/config.py`），从 `config/config.toml` 加载配置：

```python
class Config:                    # 线程安全的单例
    @property
    def llm(self) -> Dict[str, LLMSettings]     # LLM 配置（支持多模型）
    @property
    def sandbox(self) -> SandboxSettings        # 沙箱配置
    @property
    def browser_config(self) -> BrowserSettings # 浏览器配置
    @property
    def search_config(self) -> SearchSettings   # 搜索配置
    @property
    def mcp_config(self) -> MCPSettings         # MCP 配置
    @property
    def run_flow_config(self) -> RunflowSettings # Flow 配置
    @property
    def daytona(self) -> DaytonaSettings        # Daytona 配置
```

### 9.2 配置项说明

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `llm.model` | 模型名称 | `deepseek-chat` |
| `llm.base_url` | API 地址 | `https://api.deepseek.com` |
| `llm.api_type` | API 类型 | `openai`（可选 azure/aws/ollama） |
| `llm.max_tokens` | 最大回复 Token | 8192 |
| `llm.max_input_tokens` | 最大输入 Token（可选） | null |
| `sandbox.use_sandbox` | 是否使用沙箱 | false |
| `search.engine` | 默认搜索引擎 | Google |
| `browser.headless` | 浏览器无头模式 | false |
| `mcp.server_reference` | MCP 服务器模块引用 | `app.mcp.server` |

---

## 10. 执行入口

### 10.1 main.py（默认入口）

```python
async def main():
    agent = await Manus.create()
    prompt = args.prompt or input("Enter your prompt: ")
    await agent.run(prompt)
    await agent.cleanup()
```

启动流程：
1. 创建 Manus Agent 实例
2. 自动连接配置的 MCP 服务器
3. 接收用户提示词
4. 执行 Agent 主循环（think→act 迭代）
5. 清理资源（关闭浏览器、断开 MCP）

### 10.2 各入口对比

| 入口 | Agent | 用途 | 命令 |
|------|-------|------|------|
| `main.py` | Manus | 通用任务 | `python main.py --prompt "..."` |
| `run_flow.py` | PlanningFlow | 复杂多步骤任务 | `python run_flow.py` |
| `run_mcp.py` | MCPAgent | 连接 MCP 服务器 | `python run_mcp.py -c sse` |
| `sandbox_main.py` | SandboxManus | 沙箱隔离环境 | `python sandbox_main.py` |

---

## 11. 运行方式

### 11.1 安装

```bash
# 克隆仓库后安装依赖
pip install -r requirements.txt

# 或通过 setup.py 安装
pip install -e .
```

### 11.2 配置

编辑 `config/config.toml`，至少配置 LLM 的 API key：
```toml
[llm]
model = "deepseek-chat"
base_url = "https://api.deepseek.com"
api_key = "your-api-key"
max_tokens = 8192
temperature = 0.0
```

支持多种 LLM 后端（OpenAI、Azure、AWS Bedrock、Ollama、Jiekou.AI），可在 `[llm.xxx]` 子段配置不同模型。

### 11.3 基本使用

```bash
# 交互模式
python main.py

# 直接传入提示词
python main.py --prompt "帮我写一个Python脚本"

# Planning Flow 模式（复杂任务）
python run_flow.py

# MCP Agent 模式
python run_mcp.py -c stdio

# 沙箱模式（需配置 Daytona）
python sandbox_main.py
```

### 11.4 依赖要点

项目依赖约 40+ 个核心包，关键依赖包括：
- `pydantic`：数据模型验证
- `openai`：LLM API 客户端
- `browser-use`：浏览器自动化
- `crawl4ai`：网页爬虫
- `playwright`：浏览器引擎
- `mcp`：Model Context Protocol
- `docker`：Docker 沙箱
- `daytona-sdk`：云沙箱
- `boto3`：AWS Bedrock
- `loguru`：日志
- `tenacity`：重试机制

### 11.5 开发扩展

**添加新工具**：继承 `BaseTool`，实现 `execute` 方法，定义 `name/description/parameters`。

**添加新 Agent**：继承 `ToolCallAgent`，配置 `available_tools` 和 `system_prompt`。

**添加新 LLM 提供商**：扩展 `app/llm.py` 中的客户端初始化逻辑。

---

## 附录：关键技术点

### 状态管理
- Agent 状态机：IDLE → RUNNING → FINISHED/ERROR → IDLE
- 使用 `state_context` 上下文管理器确保异常时安全回滚

### 卡死检测
- 检测最近 AI 回复中的重复内容
- 连续 2 次相同回复触发卡死处理
- 自动注入策略变更提示

### MCP 集成
- 支持 SSE 和 stdio 两种传输协议
- 自动连接配置中的所有 MCP 服务器
- 工具名自动添加 `mcp_{server_id}_` 前缀防冲突
- Agent 运行时实时刷新工具列表

### 错误处理
- 独立的异常层次：`OpenManusError` → `TokenLimitExceeded` / `ToolError`
- LLM 调用自动重试（指数退避，最多 6 次）
- Token 超限异常不重试，直接终止
