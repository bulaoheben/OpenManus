# `app/config.py` — 配置系统

[toc]



## 文件位置

`app/config.py`

## 核心作用
管理 OpenManus 的全部配置：<u>从 `config/config.toml` 和 `config/mcp.json` 读取配置，通过 **Pydantic 模型验证**，提供全局单例访问</u>。

（Pydantic 是一个**用于数据验证和设置管理**的 Python 库。它通过使用 Python 类型注解（type hints），提供了简单而高效的数据验证机制。Pydantic 的核心组件是 BaseModel 类，通过继承这个类，我们可以定义具有数据验证和序列化功能的模型。）

## 路径常量
```python
PROJECT_ROOT = Path(__file__).resolve().parent.parent  # 项目根目录
WORKSPACE_ROOT = PROJECT_ROOT / "workspace"            # 工作空间目录
```

## 配置模型（Pydantic BaseModel）

### `LLMSettings`
| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `model` | `str` | 必填 | 模型名称 |
| `base_url` | `str` | 必填 | API 地址 |
| `api_key` | `str` | 必填 | API 密钥 |
| `max_tokens` | `int` | `4096` | 每次请求最大 token 数 |
| `max_input_tokens` | `Optional[int]` | `None` | 全局输入 token 上限 |
| <u>`temperature`</u> | <u>`float`</u> | <u>`1.0`</u> | <u>采样温度</u> |
| <u>`api_type`</u> | <u>`str`</u> | <u>必填</u> | <u>azure / openai / ollama 等</u> |
| `api_version` | `str` | 必填 | Azure API 版本 |

(temperature：LLM 生成文本时的随机性参数，值越高（如
1.5）输出越有创意/不确定，值越低（如 0.1）输出越确定/保守。)

(指定 LLM 后端的连接类型。项目支持三种：openai（标准 OpenAI 兼容
  API）、azure（Azure OpenAI 服务）、ollama（本地 Ollama）。在
  app/config.py:29 定义，<u>这个值决定了 app/llm.py 中初始化哪个客户端</u>)

### `SearchSettings`

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `engine` | `str` | `"Bing"` | 首选搜索引擎 |
| `fallback_engines` | `List[str]` | `["Baidu", "Bing"]` | 备用引擎 |
| `retry_delay` | `int` | `60` | 全部失败后重试等待（秒） |
| `max_retries` | `int` | `3` | 最大重试次数 |
| <u>`lang`</u> | <u>`str`</u> | <u>`"en"`</u> | <u>搜索语言</u> |
| <u>`country`</u> | <u>`str`</u> | <u>`"us"`</u> | <u>搜索国家</u> |

(搜索结果的语言偏好，传给搜索引擎 API 的参数。设为 "zh"优先返回中文结果，"en" 优先英文)

(搜索结果的地域偏好，也是传给搜索引擎的参数。"us" 偏向美国来源，"cn"偏向中国来源)

### `BrowserSettings`

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `headless` | `bool` | `False` | 是否无头模式 |
| `disable_security` | `bool` | `True` | 禁用安全特性 |
| `extra_chromium_args` | `List[str]` | `[]` | 额外 Chromium 参数 |
| `chrome_instance_path` | `Optional[str]` | `None` | Chrome 实例路径 |
| `wss_url` | `Optional[str]` | `None` | WebSocket 连接 URL |
| `cdp_url` | `Optional[str]` | `None` | CDP 连接 URL |
| `proxy` | `Optional[ProxySettings]` | `None` | 代理设置 |
| `max_content_length` | `int` | `2000` | 内容获取最大长度 |

### `SandboxSettings`
| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `use_sandbox` | `bool` | `False` | 是否使用沙箱 |
| `image` | `str` | `python:3.12-slim` | Docker 镜像 |
| `work_dir` | `str` | `/workspace` | 工作目录 |
| `memory_limit` | `str` | `512m` | 内存限制 |
| `cpu_limit` | `float` | `1.0` | CPU 限制 |
| `timeout` | `int` | `300` | 命令超时（秒） |
| `network_enabled` | `bool` | `False` | 是否启用网络 |

### `DaytonaSettings`
| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `daytona_api_key` | `str` | 必填 | Daytona API 密钥 |
| `daytona_server_url` | `str` | `https://app.daytona.io/api` | 服务器地址 |
| `daytona_target` | `str` | `us` | 目标区域 |
| `sandbox_image_name` | `str` | `whitezxj/sandbox:0.1.0` | 沙箱镜像 |
| `VNC_password` | `str` | `123456` | VNC 密码 |

### `MCPServerConfig` / `MCPSettings`
MCP 服务器配置：
- `server_reference`: 引用 `app.mcp.server`
- `servers`: 从 `config/mcp.json` 加载的服务器配置字典
- `load_server_config()`: 类方法，读取并解析 `config/mcp.json`

### `RunflowSettings`
```python
use_data_analysis_agent: bool = False  # 是否启用数据分析 Agent
```

### `ProxySettings`
```python
server: str      # 代理地址
username: str    # 代理用户名
password: str    # 代理密码
```

### `AppConfig`

顶层配置聚合模型：
```python
class AppConfig(BaseModel):
    llm: Dict[str, LLMSettings]
    sandbox: Optional[SandboxSettings]
    browser_config: Optional[BrowserSettings]
    search_config: Optional[SearchSettings]
    mcp_config: Optional[MCPSettings]
    run_flow_config: Optional[RunflowSettings]
    daytona_config: Optional[DaytonaSettings]
```

## 全局单例 `Config`

### 单例模式
<u>使用双重检查锁定（Double-Checked Locking）实现线程安全单例：</u>

( 一种多线程环境下的单例模式实现。Config 类（app/config.py:197-215）先检查 _instance 是否为 None（第一次检查），不为 None 则直接返回；为 None时才加锁再检查一次（第二次检查），确保只在首次创建时加锁，避免每次访问都有锁开销)

```python
class Config:
    _instance = None
    _lock = threading.Lock()
    _initialized = False
```

### 配置加载流程
```
Config._load_initial_config()
  │
  ├─ tomllib.load("config/config.toml")   # 读取 TOML 配置
  ├─ 处理 LLM 配置（默认 + 命名覆盖）
  ├─ 处理 Browser 配置（含 Proxy）
  ├─ 处理 Search 配置
  ├─ 处理 Sandbox 配置
  ├─ 处理 Daytona 配置
  ├─ MCPSettings.load_server_config()     # 读取 config/mcp.json
  ├─ 处理 Runflow 配置
  └─ 构建 AppConfig 实例
```

### 属性访问
| 属性 | 返回类型 | 说明 |
|------|---------|------|
| `config.llm` | `Dict[str, LLMSettings]` | LLM 配置字典 |
| `config.sandbox` | `SandboxSettings` | 沙箱配置 |
| `config.daytona` | `DaytonaSettings` | Daytona 配置 |
| `config.browser_config` | `Optional[BrowserSettings]` | 浏览器配置 |
| `config.search_config` | `Optional[SearchSettings]` | 搜索配置 |
| `config.mcp_config` | `MCPSettings` | MCP 配置 |
| `config.run_flow_config` | `RunflowSettings` | 运行流配置 |
| `config.workspace_root` | `Path` | 工作空间路径 |
| `config.root_path` | `Path` | 项目根路径 |

## 配置文件查找
```python
# 优先使用 config/config.toml
# 不存在则使用 config/config.example.toml
# 都不存在则抛出 FileNotFoundError
```

## 调用关系
- 基于 `pydantic.BaseModel` 进行数据验证
- 使用标准库 `tomllib` 解析 TOML
- 被项目中几乎所有模块引用（`app/llm.py`、`app/tool/`、`app/agent/`、`app/daytona/`、`app/sandbox/` 等）
- `config` 在模块级别实例化，全局共享
