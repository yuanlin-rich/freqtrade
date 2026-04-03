# __init__.py

## 概述

`freqtrade/rpc/__init__.py` 是 RPC 模块的包初始化文件。它从子模块中导出核心类，为其他模块提供简洁的导入路径。

## 导出内容

- `RPC` -- RPC 核心业务逻辑类，提供所有远程可调用的方法
- `RPCException` -- RPC 专用异常类，用于在 RPC 方法中抛出格式化错误
- `RPCHandler` -- RPC 处理器抽象基类，所有通信渠道（Telegram、Webhook 等）的基类
- `RPCManager` -- RPC 管理器，负责初始化和协调所有已注册的 RPC 模块

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.rpc.rpc` -- 导入 `RPC`、`RPCException`、`RPCHandler`
- `freqtrade.rpc.rpc_manager` -- 导入 `RPCManager`

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.rpc_manager` -- 导入 `RPC`、`RPCHandler`
- `freqtrade.rpc.telegram` -- 导入 `RPC`、`RPCException`、`RPCHandler`
- `freqtrade.rpc.webhook` -- 导入 `RPC`、`RPCHandler`
- `freqtrade.rpc.discord` -- 导入 `RPC`
- `freqtrade.rpc.api_server.*` -- 多个 API Server 子模块导入 `RPC`、`RPCException`
- `freqtrade.freqtradebot` -- 导入 `RPCManager`
- `freqtrade.data.dataprovider` -- 导入 `RPCManager`
