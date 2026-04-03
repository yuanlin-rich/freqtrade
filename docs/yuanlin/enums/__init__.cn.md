# __init__.py

## 概述

`freqtrade/enums/__init__.py` 是 enums 子包的入口文件，将所有枚举类和相关常量导出到包级别。这是整个项目中使用最广泛的包之一，几乎所有需要类型安全的状态表示和模式区分的模块都会从此包导入。

## 导出内容

| 导出名称 | 来源模块 | 说明 |
|---------|---------|------|
| `BacktestState` | `backteststate` | 回测状态枚举 |
| `CandleType` | `candletype` | K 线类型枚举 |
| `ExitCheckTuple` | `exitchecktuple` | 退出检查结果元组 |
| `ExitType` | `exittype` | 退出原因枚举 |
| `HyperoptState` | `hyperoptstate` | 超参数优化状态枚举 |
| `MarginMode` | `marginmode` | 保证金模式枚举 |
| `MarketDirection` | `marketstatetype` | 市场方向枚举 |
| `OrderTypeValues` | `ordertypevalue` | 订单类型枚举 |
| `PriceType` | `pricetype` | 价格类型枚举 |
| `RPCMessageType` | `rpcmessagetype` | RPC 消息类型枚举 |
| `RPCRequestType` | `rpcmessagetype` | RPC 请求类型枚举 |
| `NO_ECHO_MESSAGES` | `rpcmessagetype` | 不需要回显的消息类型集合 |
| `RunMode` | `runmode` | 运行模式枚举 |
| `TRADE_MODES` | `runmode` | 交易模式列表 |
| `OPTIMIZE_MODES` | `runmode` | 优化模式列表 |
| `NON_UTIL_MODES` | `runmode` | 非工具模式列表 |
| `SignalType` | `signaltype` | 信号类型枚举 |
| `SignalDirection` | `signaltype` | 信号方向枚举 |
| `SignalTagType` | `signaltype` | 信号标签类型枚举 |
| `State` | `state` | 机器人应用状态枚举 |
| `TradingMode` | `tradingmode` | 交易模式枚举 |

## 依赖关系

### 内部依赖（本项目模块）
- 本包下的所有子模块

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- 项目中几乎所有模块通过 `from freqtrade.enums import ...` 引用本包，涵盖：
  - `freqtrade.configuration.*` — 配置模块
  - `freqtrade.exchange.*` — 交易所接口
  - `freqtrade.optimize.*` — 优化和回测
  - `freqtrade.rpc.*` — RPC 通信
  - `freqtrade.strategy.*` — 策略接口
  - `freqtrade.persistence.*` — 数据持久化
  - `freqtrade.freqai.*` — FreqAI 模块
  - `freqtrade.plugins.*` — 插件系统
  - `freqtrade.data.*` — 数据处理
  - `freqtrade.commands.*` — CLI 命令
