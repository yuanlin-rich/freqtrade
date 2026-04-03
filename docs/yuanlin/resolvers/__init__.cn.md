# __init__.py

## 概述

`freqtrade/resolvers/__init__.py` 是解析器模块的包初始化文件，负责导出各种对象解析器。

## 导出内容

- `IResolver` — 解析器基类
- `ExchangeResolver` — 交易所解析器
- `PairListResolver` — 交易对列表解析器
- `ProtectionResolver` — 保护机制解析器
- `StrategyResolver` — 策略解析器

**注意：** `HyperOptResolver` 被注释掉，未在此导入，以避免加载整个 Optimize 依赖树。需要时应直接从 `freqtrade.resolvers.hyperopt_resolver` 导入。

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.resolvers.iresolver` — IResolver 基类
- `freqtrade.resolvers.exchange_resolver` — ExchangeResolver
- `freqtrade.resolvers.pairlist_resolver` — PairListResolver
- `freqtrade.resolvers.protection_resolver` — ProtectionResolver
- `freqtrade.resolvers.strategy_resolver` — StrategyResolver

### 被依赖（谁引用了本文件）
- 项目中大量模块通过 `from freqtrade.resolvers import ...` 使用各解析器
- `freqtrade.freqtradebot` — 加载策略和交易所
- `freqtrade.optimize.backtesting` — 加载策略
- `freqtrade.plot.plotting` — 加载策略和交易所
- `freqtrade.rpc.api_server.*` — API 服务加载各种对象
- `freqtrade.plugins.pairlistmanager` — 加载 pairlist
- `freqtrade.plugins.protectionmanager` — 加载 protection
