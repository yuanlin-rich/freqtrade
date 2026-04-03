# __init__.py

## 概述
API Server 包的初始化文件。该文件仅导出 `ApiServer` 类，作为整个 `api_server` 模块的公共入口。

## 架构图
```mermaid
graph LR
    A["__init__.py"] -->|"re-export"| B["webserver.py::ApiServer"]
    C["外部模块"] -->|"from freqtrade.rpc.api_server import ApiServer"| A
```

## 核心导出

### ApiServer
从 `webserver.py` 中重新导出 `ApiServer` 类，使外部模块可以通过 `from freqtrade.rpc.api_server import ApiServer` 直接访问。

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.rpc.api_server.webserver.ApiServer` — 导入并重新导出 ApiServer 类

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.rpc_manager` — 导入 ApiServer 用于管理 RPC handler
- `freqtrade.commands.webserver_commands` — 在 webserver 命令中使用 ApiServer
- `tests.rpc.test_rpc_apiserver` — 测试文件
