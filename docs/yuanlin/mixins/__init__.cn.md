# mixins/__init__.py

## 概述
`freqtrade/mixins/__init__.py` 是 mixins 子包的入口文件，导出 `LoggingMixin` 类。该包提供可复用的 Mixin 类，用于通过多重继承为其他类添加通用功能。

## 导出列表
- `LoggingMixin` — 来自 `freqtrade.mixins.logging_mixin`，提供日志去重功能

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.mixins.logging_mixin` — LoggingMixin 的实际定义

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- `freqtrade.freqtradebot` — FreqtradeBot 继承 LoggingMixin
- `freqtrade.exchange.common` — 交易所通用模块
- `freqtrade.optimize.backtesting` — 回测引擎
- `freqtrade.plugins.pairlistmanager` — Pairlist 管理器
- `freqtrade.plugins.pairlist.IPairList` — Pairlist 基类
- `freqtrade.plugins.protections.iprotection` — 保护机制基类
- `freqtrade.rpc.fiat_convert` — 法币转换
