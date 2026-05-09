# `app/daytona/sandbox.py` — Daytona Sandbox 管理

## 文件位置
`app/daytona/sandbox.py`

## 核心作用
提供基于 Daytona 云沙箱的生命周期管理功能，包括沙箱的创建、获取/启动、删除以及 supervisord 会话管理。

## 全局配置
```python
daytona_settings = config.daytona
daytona = Daytona(DaytonaConfig(api_key=..., server_url=..., target=...))
```
在模块加载时从 `config.daytona` 读取配置并初始化 Daytona 客户端。

## 函数说明

### `get_or_start_sandbox(sandbox_id)` — 获取/启动沙箱
异步函数。根据 ID 获取沙箱，检查状态：
- `ARCHIVED` 或 `STOPPED` → 调用 `daytona.start()` 启动，然后执行 `start_supervisord_session()`
- 其他状态 → 直接返回就绪的沙箱

### `start_supervisord_session(sandbox)` — 启动 supervisord
同步函数。在沙箱中创建 `"supervisord-session"` 会话，执行 supervisord 命令（异步模式），等待 25 秒确保服务启动完成。

### `create_sandbox(password, project_id=None)` — 创建沙箱
同步函数。使用 `CreateSandboxFromImageParams` 配置沙箱参数：
- 镜像：由 `daytona_settings.sandbox_image_name` 指定
- 资源：2 CPU / 4GB 内存 / 5GB 磁盘
- 环境变量：Chrome 调试端口 9222、VNC 分辨率 1024x768、匿名遥测关闭等
- 自动停止：15 分钟无活动后停止
- 自动归档：24 小时后归档
创建后调用 `start_supervisord_session()` 启动服务。

### `delete_sandbox(sandbox_id)` — 删除沙箱
异步函数。获取沙箱后调用 `daytona.delete()` 删除。

## 调用关系
- 使用 `app.config` 的 `config` 对象获取 Daytona 配置
- 使用 `app.utils.logger` 记录日志
- 被 `app/daytona/tool_base.py` 中的 `SandboxToolsBase` 调用
- 基于第三方 `daytona` SDK

## 设计说明
- 沙箱环境预配置了 Chrome 浏览器（调试端口 9222）、VNC（端口 6080）、Web 服务（端口 8080）
- supervisord 用于管理沙箱内多个后台服务的启停
