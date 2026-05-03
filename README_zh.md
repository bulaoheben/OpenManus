<p align="center">
  <img src="assets/logo.jpg" width="200"/>
</p>

[English](README.md) | 中文 | [한국어](README_ko.md) | [日本語](README_ja.md)

[![GitHub stars](https://img.shields.io/github/stars/FoundationAgents/OpenManus?style=social)](https://github.com/FoundationAgents/OpenManus/stargazers)
&ensp;
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) &ensp;
[![Discord Follow](https://dcbadge.vercel.app/api/server/DYn29wFk9z?style=flat)](https://discord.gg/DYn29wFk9z)
[![Demo](https://img.shields.io/badge/Demo-Hugging%20Face-yellow)](https://huggingface.co/spaces/lyh-917/OpenManusDemo)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15186407.svg)](https://doi.org/10.5281/zenodo.15186407)

# 👋 OpenManus

Manus 非常棒，但 OpenManus 无需邀请码即可实现任何创意 🛫！

我们的团队成员 [@Xinbin Liang](https://github.com/mannaandpoem) 和 [@Jinyu Xiang](https://github.com/XiangJinyu)（核心作者），以及 [@Zhaoyang Yu](https://github.com/MoshiQAQ)、[@Jiayi Zhang](https://github.com/didiforgithub) 和 [@Sirui Hong](https://github.com/stellaHSR)，来自 [@MetaGPT](https://github.com/geekan/MetaGPT)团队。我们在 3
小时内完成了开发并持续迭代中！

这是一个简洁的实现方案，欢迎任何建议、贡献和反馈！

用 OpenManus 开启你的智能体之旅吧！

我们也非常高兴地向大家介绍 [OpenManus-RL](https://github.com/OpenManus/OpenManus-RL)，这是一个专注于基于强化学习（RL，例如 GRPO）的方法来优化大语言模型（LLM）智能体的开源项目，由来自UIUC 和 OpenManus 的研究人员合作开发。

## 项目演示

<video src="https://private-user-images.githubusercontent.com/61239030/420168772-6dcfd0d2-9142-45d9-b74e-d10aa75073c6.mp4?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDEzMTgwNTksIm5iZiI6MTc0MTMxNzc1OSwicGF0aCI6Ii82MTIzOTAzMC80MjAxNjg3NzItNmRjZmQwZDItOTE0Mi00NWQ5LWI3NGUtZDEwYWE3NTA3M2M2Lm1wND9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA3VDAzMjIzOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdiZjFkNjlmYWNjMmEzOTliM2Y3M2VlYjgyNDRlZDJmOWE3NWZhZjE1MzhiZWY4YmQ3NjdkNTYwYTU5ZDA2MzYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.UuHQCgWYkh0OQq9qsUWqGsUbhG3i9jcZDAMeHjLt5T4" data-canonical-src="https://private-user-images.githubusercontent.com/61239030/420168772-6dcfd0d2-9142-45d9-b74e-d10aa75073c6.mp4?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDEzMTgwNTksIm5iZiI6MTc0MTMxNzc1OSwicGF0aCI6Ii82MTIzOTAzMC80MjAxNjg3NzItNmRjZmQwZDItOTE0Mi00NWQ5LWI3NGUtZDEwYWE3NTA3M2M2Lm1wND9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA3VDAzMjIzOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdiZjFkNjlmYWNjMmEzOTliM2Y3M2VlYjgyNDRlZDJmOWE3NWZhZjE1MzhiZWY4YmQ3NjdkNTYwYTU5ZDA2MzYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.UuHQCgWYkh0OQq9qsUWqGsUbhG3i9jcZDAMeHjLt5T4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px; min-height: 200px"></video>

## 安装指南

我们提供两种安装方式。推荐使用方式二（uv），因为它能提供更快的安装速度和更好的依赖管理。

### 方式一：使用 conda

1. 创建新的 conda 环境：

```bash
conda create -n open_manus python=3.12
conda activate open_manus
```

2. 克隆仓库：

```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. 安装依赖：

```bash
pip install -r requirements.txt
```

### 方式二：使用 uv（推荐）

1. 安装 uv（一个快速的 Python 包管理器）：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2. 克隆仓库：

```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. 创建并激活虚拟环境：

```bash
uv venv --python 3.12
source .venv/bin/activate  # Unix/macOS 系统
# Windows 系统使用：
# .venv\Scripts\activate
```

4. 安装依赖：

```bash
uv pip install -r requirements.txt
```

### 浏览器自动化工具（可选）
```bash
playwright install
```

## 配置说明

OpenManus 需要配置使用的 LLM API，请按以下步骤设置：

1. 在 `config` 目录创建 `config.toml` 文件（可从示例复制）：

```bash
cp config/config.example.toml config/config.toml
```

2. 编辑 `config/config.toml` 添加 API 密钥和自定义设置：

```toml
# 全局 LLM 配置
[llm]
model = "gpt-4o"
base_url = "https://api.openai.com/v1"
api_key = "sk-..."  # 替换为真实 API 密钥
max_tokens = 4096
temperature = 0.0

# 可选特定 LLM 模型配置
[llm.vision]
model = "gpt-4o"
base_url = "https://api.openai.com/v1"
api_key = "sk-..."  # 替换为真实 API 密钥
```

## 快速启动

一行命令运行 OpenManus：

```bash
python main.py
```

然后通过终端输入你的创意！

如需使用 MCP 工具版本，可运行：
```bash
python run_mcp.py
```

如需体验不稳定的多智能体版本，可运行：

```bash
python run_flow.py
```

## 添加自定义多智能体

目前除了通用的 OpenManus Agent, 我们还内置了DataAnalysis Agent，适用于数据分析和数据可视化任务，你可以在`config.toml`中将这个智能体加入到`run_flow`中
```toml
# run-flow可选配置
[runflow]
use_data_analysis_agent = true     # 默认关闭，将其改为true则为激活
```
除此之外，你还需要安装相关的依赖来确保智能体正常运行：[具体安装指南](app/tool/chart_visualization/README_zh.md##安装)


## 贡献指南

我们欢迎任何友好的建议和有价值的贡献！可以直接创建 issue 或提交 pull request。

或通过 📧 邮件联系 @mannaandpoem：mannaandpoem@gmail.com

**注意**: 在提交 pull request 之前，请使用 pre-commit 工具检查您的更改。运行 `pre-commit run --all-files` 来执行检查。

## 交流群

加入我们的飞书交流群，与其他开发者分享经验！

<div align="center" style="display: flex; gap: 20px;">
    <img src="assets/community_group.jpg" alt="OpenManus 交流群" width="300" />
</div>

## Star 数量

[![Star History Chart](https://api.star-history.com/svg?repos=FoundationAgents/OpenManus&type=Date)](https://star-history.com/#FoundationAgents/OpenManus&Date)


## 赞助商
感谢[PPIO](https://ppinfra.com/user/register?invited_by=OCPKCN&utm_source=github_openmanus&utm_medium=github_readme&utm_campaign=link) 提供的算力支持。
> PPIO派欧云：一键调用高性价比的开源模型API和GPU容器

## 致谢

特别感谢 [anthropic-computer-use](https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo)
和 [browser-use](https://github.com/browser-use/browser-use) 为本项目提供的基础支持！

此外，我们感谢 [AAAJ](https://github.com/metauto-ai/agent-as-a-judge)，[MetaGPT](https://github.com/geekan/MetaGPT)，[OpenHands](https://github.com/All-Hands-AI/OpenHands) 和 [SWE-agent](https://github.com/SWE-agent/SWE-agent).

我们也感谢阶跃星辰 (stepfun) 提供的 Hugging Face 演示空间支持。

OpenManus 由 MetaGPT 社区的贡献者共同构建，感谢这个充满活力的智能体开发者社区！

## 引用
```bibtex
@misc{openmanus2025,
  author = {Xinbin Liang and Jinyu Xiang and Zhaoyang Yu and Jiayi Zhang and Sirui Hong and Sheng Fan and Xiao Tang},
  title = {OpenManus: An open-source framework for building general AI agents},
  year = {2025},
  publisher = {Zenodo},
  doi = {10.5281/zenodo.15186407},
  url = {https://doi.org/10.5281/zenodo.15186407},
}
```





已解决问题

| 序号 | 问题描述                                     | 文件 / 位置             | 根因分析                                                     | 修复方式                               |
| ---- | -------------------------------------------- | ----------------------- | ------------------------------------------------------------ | -------------------------------------- |
| 1    | DaytonaSettings () 空构造报错                | app/config.py:297       | config.toml 无 [daytona] 节点时，仍空参数实例化；daytona_api_key 为必填字段 | 改为 daytona_settings = None           |
| 2    | SandboxBrowserTool 模块级导入阻塞            | app/agent/browser.py:11 | 文件顶部直接 `from app.tool.sandbox.sb_browser_tool import SandboxBrowserTool`，触发 daytona 依赖链阻塞 | 改为在方法内部**延迟导入**             |
| 3    | API 请求返回 404                             | config/config.toml:4    | 配置 `base_url = "https://api.deepseek.com/anthropic"`，该代理接口路径已失效 | 修改为 `https://api.deepseek.com`      |
| 4    | 缺少 structlog 模块                          | app/utils/logger.py:4   | structlog 未写入 requirements.txt 依赖清单                   | 执行：`pip install structlog`          |
| 5    | RequestsDependencyWarning 版本不匹配         | open_manus conda 环境   | urllib3 2.6.3 与 requests 版本存在兼容冲突                   | 执行：`pip install --upgrade requests` |
| 6    | Playwright 浏览器未安装                      | 系统级环境              | 未自动下载安装 Chromium 内核浏览器                           | 执行：`playwright install chromium`    |
| 7    | config.toml 中 vision 模型 base_url 地址错误 | config/config.toml:45   | 沿用已失效的代理接口地址，和第 3 条同源问题                  | 统一改为 `https://api.deepseek.com`    |

---
  剩余问题（不影响运行）

  1. Pydantic 兼容警告（无害）

  UserWarning: Valid config keys have changed in V2:
  * 'underscore_attrs_are_private' has been removed
  - 原因：browser-use 第三方包使用了旧版 pydantic v1 的配置项
  - 影响：纯警告，功能完全正常
  - 能否修复：需等 browser-use 包升级适配 pydantic v2，或者降级 pydantic 到 v1（不推荐）

  2. 中文站点反自动化检测（行为限制，非错误）

  - 12306.cn、携程等网站对 headless 浏览器有限制，AI 自动操作时可能无法完整获取页面内容（如票价表格）
  - 表现：页面能打开，但输入框/查询结果被隐藏或需验证码
  - 建议：去 12306 App 购票更可靠

  3. Google 搜索超时

  RetryError[ConnectTimeout]
  - 原因：国内网络访问 Google 受限
  - 已有兜底：项目配置了 DuckDuckGo、Baidu、Bing 等多个搜索引擎作为 fallback

  4. 未安装的可选依赖

  ┌──────────────────┬─────────────────┬─────────────────────────────┬────────────┐
  │       功能       │      依赖       │          安装方式           │  是否需要  │
  ├──────────────────┼─────────────────┼─────────────────────────────┼────────────┤
  │ Daytona 远程沙箱 │ daytona==0.21.8 │ pip install daytona==0.21.8 │ 不用可忽略 │
  ├──────────────────┼─────────────────┼─────────────────────────────┼────────────┤
  │ Docker 本地沙箱  │ Docker Desktop  │ 手动安装 Docker             │ 不用可忽略 │
  ├──────────────────┼─────────────────┼─────────────────────────────┼────────────┤
  │ 图表可视化       │ Node.js >= 18   │ 手动安装 + npm install      │ 不用可忽略 │
  ├──────────────────┼─────────────────┼─────────────────────────────┼────────────┤
  │ A2A 协议         │ a2a-sdk==0.2.5  │ pip install a2a-sdk==0.2.5  │ 不用可忽略 │
  └──────────────────┴─────────────────┴─────────────────────────────┴────────────┘

  一句话：核心功能（LLM 对话 + 浏览器 + 搜索 + Python 执行）全部就绪，可以直接用了。
