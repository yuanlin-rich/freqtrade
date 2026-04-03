# ws_types.py

## 概述

`freqtrade/rpc/api_server/ws/ws_types.py` 定义了 WebSocket 子系统中使用的类型变量和类型别名。它将 FastAPI 的 WebSocket 和 websockets 库的客户端连接统一到一个类型变量下，使得 WebSocket 相关的代码可以无差别地处理两种不同的 WebSocket 实现。

## 核心类/函数

### WebSocketType
- **类型**: `TypeVar("WebSocketType", FastAPIWebSocket, WebSocket)`
- **说明**: 一个受约束的类型变量，只能绑定到 FastAPI 的 `WebSocket` 或 websockets 库的 `ClientConnection`
- **用途**: 使 `WebSocketProxy`、`WebSocketChannel` 等类在构造时可以接受任一类型的 WebSocket 对象

### MessageType
- **类型**: `dict[str, Any]`
- **说明**: WebSocket 消息的基础类型，本质上是一个字符串键的字典
- **用途**: 表示通过 WebSocket 传输的通用消息格式

## 依赖关系

### 内部依赖（本项目模块）
无

### 外部依赖（第三方库）
- `typing` -- `Any`、`TypeVar`
- `fastapi` -- 导入 `WebSocket as FastAPIWebSocket`
- `websockets.asyncio.client` -- 导入 `ClientConnection as WebSocket`

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.api_server.ws.__init__` -- 导出 `WebSocketType`
- `freqtrade.rpc.api_server.ws.channel` -- 使用 `WebSocketType` 作为构造参数类型
- `freqtrade.rpc.api_server.ws.proxy` -- 使用 `WebSocketType` 作为构造参数类型
