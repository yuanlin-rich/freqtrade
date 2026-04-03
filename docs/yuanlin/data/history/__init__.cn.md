# __init__.py

## 概述

`freqtrade/data/history/__init__.py` 是历史数据子包的初始化文件。它从 `datahandlers` 子模块和 `history_utils` 模块中导出所有公共 API，提供统一的历史数据操作入口。

## 导出内容

### 来自 `datahandlers` 子模块
- `get_datahandler` -- 获取数据处理器实例的工厂函数

### 来自 `history_utils` 模块
- `convert_trades_to_ohlcv` -- 交易数据转 OHLCV（重新导出自 converter）
- `download_data_main` -- 数据下载主入口
- `get_timerange` -- 获取数据集的时间范围
- `load_data` -- 批量加载多交易对数据
- `load_pair_history` -- 加载单个交易对历史数据
- `refresh_backtest_ohlcv_data` -- 刷新回测 OHLCV 数据
- `refresh_backtest_trades_data` -- 刷新回测交易数据
- `refresh_data` -- 刷新数据
- `validate_backtest_data` -- 验证回测数据完整性

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.data.history.datahandlers` -- 数据处理器
- `freqtrade.data.history.history_utils` -- 历史数据工具函数

### 被依赖（谁引用了本文件）
- `freqtrade.data.dataprovider` -- DataProvider 使用 get_datahandler 和 load_pair_history
- `freqtrade.data.converter.converter` -- 格式转换使用 get_datahandler
- `freqtrade.data.converter.trade_converter` -- 交易转换使用 get_datahandler
- `freqtrade.data.converter.trade_converter_kraken` -- Kraken 导入使用 get_datahandler
- `freqtrade.optimize.backtesting` -- 回测引擎使用数据加载和验证
- `freqtrade.optimize.hyperopt` -- Hyperopt 使用数据加载
- `freqtrade.plot.plotting` -- 绘图使用数据加载
- `freqtrade.rpc.api_server.api_download_data` -- API 数据下载
- `freqtrade.commands.data_commands` -- CLI 数据命令
- `freqtrade.freqai.*` -- FreqAI 模块使用历史数据
- `freqtrade.exchange.exchange` -- 交易所模块使用 get_datahandler
