# __init__.py

## 概述

Exchange 模块的包初始化文件，负责将 exchange 子包中的所有交易所类和工具函数导出，方便外部模块通过 `from freqtrade.exchange import ...` 直接导入使用。

## 导出内容

### 交易所类
- `Exchange` -- 基础交易所类
- `Binance`, `Binanceus`, `Binanceusdm` -- Binance 系列
- `Bingx` -- BingX 交易所
- `Bitget` -- Bitget 交易所
- `Bitmart` -- Bitmart 交易所
- `Bitpanda` -- Bitpanda 交易所
- `Bitvavo` -- Bitvavo 交易所
- `Bybit` -- Bybit 交易所
- `Coinex` -- CoinEx 交易所
- `Cryptocom` -- Crypto.com 交易所
- `Gate` -- Gate.io 交易所
- `Hitbtc` -- HitBTC 交易所
- `Htx` -- HTX (原火币) 交易所
- `Hyperliquid` -- Hyperliquid DEX
- `Idex` -- IDEX 交易所
- `Kraken` -- Kraken 交易所
- `Krakenfutures` -- Kraken Futures 交易所
- `Kucoin` -- KuCoin 交易所
- `Lbank` -- LBank 交易所
- `Luno` -- Luno 交易所
- `Modetrade` -- ModeTrade 交易所
- `Okx`, `Myokx`, `Okxus` -- OKX 系列

### 工具函数
- `ROUND_DOWN`, `ROUND_UP` -- 舍入常量
- `amount_to_contract_precision` -- 金额转合约精度
- `amount_to_contracts` / `contracts_to_amount` -- 金额与合约数互转
- `amount_to_precision` / `price_to_precision` -- 精度处理
- `available_exchanges` / `ccxt_exchanges` -- 可用交易所列表
- `date_minus_candles` -- 日期减去 K 线数
- `is_exchange_known_ccxt` -- 检查交易所是否 ccxt 支持
- `list_available_exchanges` -- 列出所有可用交易所
- `market_is_active` -- 市场是否活跃
- `validate_exchange` -- 验证交易所
- `timeframe_to_*` 系列 -- 时间周期转换函数

### 映射字典
- `MAP_EXCHANGE_CHILDCLASS` -- 交易所名称到子类名称的映射

## 依赖关系

### 内部依赖
- `freqtrade.exchange.common` -- 提供 `MAP_EXCHANGE_CHILDCLASS`
- `freqtrade.exchange.exchange` -- 提供基础 `Exchange` 类
- `freqtrade.exchange.exchange_utils` -- 提供工具函数
- `freqtrade.exchange.exchange_utils_timeframe` -- 提供时间周期工具函数
- 各交易所子模块 -- 提供各交易所具体实现

### 被依赖
- 项目中几乎所有需要使用交易所功能的模块都通过此 `__init__.py` 导入
