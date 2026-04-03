# persistence/__init__.py

## 概述

`persistence` 包的初始化模块，负责从子模块中导出所有核心持久化类和函数，为项目其他模块提供统一的导入入口。这是 freqtrade 数据持久化层的公共 API 入口点。

## 导出内容

| 导出名称 | 来源模块 | 说明 |
|----------|---------|------|
| `CustomDataWrapper` | `custom_data` | 自定义数据中间件，抽象数据库层 |
| `KeyStoreKeys` | `key_value_store` | 键值存储的合法键类型 |
| `KeyValueStore` | `key_value_store` | 通用键值持久存储 |
| `init_db` | `models` | 数据库初始化函数 |
| `PairLocks` | `pairlock_middleware` | 交易对锁定中间件 |
| `LocalTrade` | `trade_model` | 本地交易模型（用于回测） |
| `Order` | `trade_model` | 订单数据库模型 |
| `Trade` | `trade_model` | 交易数据库模型 |
| `FtNoDBContext` | `usedb_context` | 无数据库上下文管理器 |
| `disable_database_use` | `usedb_context` | 禁用数据库使用 |
| `enable_database_use` | `usedb_context` | 启用数据库使用 |

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.persistence.custom_data` -- CustomDataWrapper
- `freqtrade.persistence.key_value_store` -- KeyStoreKeys, KeyValueStore
- `freqtrade.persistence.models` -- init_db
- `freqtrade.persistence.pairlock_middleware` -- PairLocks
- `freqtrade.persistence.trade_model` -- LocalTrade, Order, Trade
- `freqtrade.persistence.usedb_context` -- FtNoDBContext, disable_database_use, enable_database_use

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
项目中超过 50 个模块通过 `from freqtrade.persistence import ...` 引用本包，主要包括：
- `freqtrade.freqtradebot` -- 核心交易机器人
- `freqtrade.rpc.rpc` -- RPC 接口
- `freqtrade.optimize.backtesting` -- 回测引擎
- `freqtrade.strategy.interface` -- 策略接口
- `freqtrade.wallets` -- 钱包管理
- `freqtrade.plugins.protectionmanager` -- 保护管理器
- `freqtrade.commands.db_commands` -- 数据库命令
