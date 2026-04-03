# webserver.py

## 概述
Freqtrade API Server 的核心模块，实现了 `ApiServer` 类作为整个 REST API 和 WebSocket 服务器的入口。`ApiServer` 是一个单例类（Singleton），继承自 `RPCHandler`，负责创建 FastAPI 应用实例、配置路由、中间件、CORS 策略、异常处理，以及启动 Uvicorn HTTP 服务器。同时定义了自定义 JSON 响应类 `FTJSONResponse` 和 FastAPI 的 lifespan 管理。

## 架构图
```mermaid
classDiagram
    class RPCHandler {
        <<abstract>>
        +add_rpc_handler(rpc)
        +cleanup()
        +send_msg(msg)
    }

    class ApiServer {
        -__instance: ApiServer
        -__initialized: bool
        +_rpc: RPC
        +_has_rpc: bool
        +_config: Config
        +_message_stream: MessageStream
        +app: FastAPI
        +_server: UvicornServer
        +_standalone: bool
        +__new__()
        +__init__(config, standalone)
        +add_rpc_handler(rpc)
        +cleanup()
        +shutdown()
        +send_msg(msg)
        +handle_rpc_exception(request, exc)
        +configure_app(app, config)
        +start_api()
    }

    RPCHandler <|-- ApiServer

    class FTJSONResponse {
        +media_type: str
        +render(content) bytes
    }

    class lifespan {
        <<asynccontextmanager>>
        创建/销毁 MessageStream
    }

    ApiServer --> FTJSONResponse : "default_response_class"
    ApiServer --> lifespan : "lifespan"
    ApiServer --> UvicornServer : "_server"

    subgraph "路由注册"
        R1["api_v1_public"]
        R2["router_login"]
        R3["api_v1"]
        R4["api_trading"]
        R5["api_webserver"]
        R6["api_backtest"]
        R7["api_bg_tasks"]
        R8["api_pair_history"]
        R9["api_pairlists"]
        R10["api_download_data"]
        R11["ws_router"]
        R12["router_ui"]
    end
```

## 核心类/函数

### FTJSONResponse
自定义 JSON 响应类，继承自 Starlette 的 `JSONResponse`。
- **关键逻辑**: 使用 `orjson` 替代标准 `json` 进行序列化，支持 NumPy 类型序列化（`OPT_SERIALIZE_NUMPY`），能正确处理 NaN 和 Inf 值

### lifespan(app)
FastAPI 应用的生命周期管理器（async context manager）。
- **启动时**: 创建 `MessageStream` 实例，确保其在 Uvicorn 的 event loop 中创建
- **关闭时**: 清理 `MessageStream`

### ApiServer
API 服务器主类，实现为单例模式。

#### __new__(cls)
单例模式实现。确保全局只有一个 `ApiServer` 实例。

#### __init__(self, config, standalone=False)
初始化 API 服务器。
- **参数**: `config` (Config) - 系统配置；`standalone` (bool) - 是否独立运行模式（webserver 命令）
- **关键逻辑**:
  1. 如已初始化且为 standalone 模式，直接返回（避免重复初始化）
  2. 创建 FastAPI 应用，配置 docs_url（可通过 `enable_openapi` 配置开启）
  3. 调用 `configure_app()` 注册路由和中间件
  4. 调用 `start_api()` 启动服务器

#### add_rpc_handler(self, rpc)
附加 RPC handler。
- **参数**: `rpc` (RPC) - RPC 实例
- **异常**: `OperationalException` - 如果已经附加了 RPC handler

#### cleanup(self)
清理资源。
- **关键逻辑**: 清除 RPC handler、Exchange 缓存和后台任务，停止 Uvicorn 服务器（非 standalone 模式）

#### shutdown(cls)
类方法，完全重置单例状态。清除 `__instance`、`__initialized`、`_has_rpc`、`_rpc`。

#### send_msg(self, msg)
发布消息到 WebSocket 消息流。
- **参数**: `msg` (RPCSendMsg) - 要发送的消息

#### handle_rpc_exception(self, request, exc)
全局 RPCException 异常处理器，返回 502 状态码和错误信息。

#### configure_app(self, app, config)
配置 FastAPI 应用的所有路由和中间件。
- **路由注册顺序**:
  1. `api_v1_public` — 公共端点（`/api/v1`，无需认证）
  2. `router_login` — 认证端点（`/api/v1`，Auth 标签）
  3. `api_v1` — 通用私有端点（需 JWT/Basic 认证）
  4. `api_trading` — 交易端点（需认证 + 交易模式）
  5. `api_webserver` — Webserver 模式端点（需认证 + webserver 模式）
  6. `api_backtest` — 回测端点（需认证 + webserver 模式）
  7. `api_bg_tasks` — 后台任务端点（需认证 + webserver 模式）
  8. `api_pair_history` — 历史数据端点（需认证 + webserver 模式）
  9. `api_pairlists` — Pairlist 端点（需认证 + webserver 模式）
  10. `api_download_data` — 数据下载端点（需认证 + webserver 模式）
  11. `ws_router` — WebSocket 端点
  12. `router_ui` — Web UI 静态文件（**必须最后注册**）
- **中间件**: 配置 CORS 中间件，使用 `api_server.CORS_origins` 配置
- **异常处理**: 注册 `RPCException` 全局异常处理器

#### start_api(self)
启动 API 服务器。
- **关键逻辑**:
  1. 从配置读取 `listen_ip_address` 和 `listen_port`
  2. **安全检查**: 如果监听地址不是 loopback 且不在 Docker 中，发出安全警告
  3. **安全检查**: 如果未设置密码，发出安全警告
  4. **安全检查**: 如果 `jwt_secret_key` 使用默认值，发出安全警告
  5. 配置 Uvicorn（禁用颜色、配置日志级别、禁用 ws_ping_interval）
  6. standalone 模式直接运行，非 standalone 模式在线程中运行

### _OPENAPI_TAGS
OpenAPI 标签定义列表，为 API 文档提供分类描述，包括 Auth、Info、Bot-control、Pairlist、Locks、Candle data、Trading-info、Trades、Strategy、Hyperopt、FreqAI、Download-data、Backtest、Pairlists、Trading、Webserver。

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.configuration` — `running_in_docker` 检测 Docker 环境
- `freqtrade.constants` — `Config` 类型
- `freqtrade.exceptions` — `OperationalException`
- `freqtrade.rpc.api_server.uvicorn_threaded` — `UvicornServer` 多线程服务器
- `freqtrade.rpc.api_server.webserver_bgwork` — `ApiBG` 状态管理
- `freqtrade.rpc.api_server.ws.message_stream` — `MessageStream` 消息流
- `freqtrade.rpc.rpc` — `RPC`、`RPCException`、`RPCHandler`
- `freqtrade.rpc.rpc_types` — `RPCSendMsg` 消息类型
- 所有 api_* 路由模块（延迟导入于 `configure_app()` 中）

### 外部依赖（第三方库）
- `fastapi` — Web 框架（FastAPI、Depends）
- `fastapi.middleware.cors` — CORS 中间件
- `starlette.responses` — `JSONResponse`
- `orjson` — 高性能 JSON 序列化
- `uvicorn` — ASGI 服务器配置

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.api_server.__init__` — 重新导出 `ApiServer`
- `freqtrade.rpc.api_server.deps` — 导入 `ApiServer` 访问全局状态
- `tests.rpc.test_rpc_manager` — 测试文件
