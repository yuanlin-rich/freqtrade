# __init__.py (api_server/ws)

## 概述

`freqtrade/rpc/api_server/ws/__init__.py` 是 WebSocket 子模块的包初始化文件，从子模块中导出核心类，为其他模块提供统一的导入路径。使用 `# isort: off` 注释控制导入顺序以避免循环依赖。

## 导出内容

- `WebSocketType` -- WebSocket 类型变量，兼容 FastAPI WebSocket 和 websockets 客户端连接
- `WebSocketProxy` -- WebSocket 代理类，统一 FastAPI 和 websockets 的 API 差异
- `HybridJSONWebSocketSerializer` -- 混合 JSON 序列化器，支持 DataFrame 的序列化/反序列化
- `WebSocketChannel` -- WebSocket 通道管理类，封装连接生命周期和消息收发
- `MessageStream` -- 异步消息流，用于 Producer-Consumer 模式的消息传递

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.rpc.api_server.ws.ws_types` -- 导入 `WebSocketType`
- `freqtrade.rpc.api_server.ws.proxy` -- 导入 `WebSocketProxy`
- `freqtrade.rpc.api_server.ws.serializer` -- 导入 `HybridJSONWebSocketSerializer`
- `freqtrade.rpc.api_server.ws.channel` -- 导入 `WebSocketChannel`
- `freqtrade.rpc.api_server.ws.message_stream` -- 导入 `MessageStream`

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.external_message_consumer` -- 导入 `WebSocketChannel`、`create_channel`
- `freqtrade.rpc.api_server.api_ws` -- 导入 `WebSocketChannel`、`MessageStream`
- `freqtrade.rpc.api_server.webserver` -- 导入 `MessageStream`
