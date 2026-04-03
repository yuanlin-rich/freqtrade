# __init__.py

## 概述

`freqtrade/data/converter/__init__.py` 是数据转换子包的初始化文件。它从三个子模块中统一导入并通过 `__all__` 明确暴露所有公共 API，作为 converter 包的统一入口。

## 导出内容

### 来自 `converter.py` 模块
- `clean_ohlcv_dataframe` -- 清洗 OHLCV DataFrame
- `convert_ohlcv_format` -- 转换 OHLCV 数据格式
- `ohlcv_fill_up_missing_data` -- 填充缺失 OHLCV 数据
- `ohlcv_to_dataframe` -- OHLCV 列表转 DataFrame
- `order_book_to_dataframe` -- 订单簿转 DataFrame
- `reduce_dataframe_footprint` -- 减小 DataFrame 内存占用
- `trim_dataframe` -- 按时间范围裁剪 DataFrame
- `trim_dataframes` -- 批量裁剪 DataFrame 字典

### 来自 `orderflow.py` 模块
- `populate_dataframe_with_trades` -- 使用交易数据填充订单流数据

### 来自 `trade_converter.py` 模块
- `convert_trades_format` -- 转换交易数据格式
- `convert_trades_to_ohlcv` -- 交易数据转 OHLCV
- `trades_convert_types` -- 交易数据类型转换
- `trades_df_remove_duplicates` -- 去除交易数据重复项
- `trades_dict_to_list` -- 交易字典转列表
- `trades_list_to_df` -- 交易列表转 DataFrame
- `trades_to_ohlcv` -- 单次交易数据转 OHLCV

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.data.converter.converter` -- OHLCV 数据转换
- `freqtrade.data.converter.orderflow` -- 订单流数据处理
- `freqtrade.data.converter.trade_converter` -- 交易数据转换

### 被依赖（谁引用了本文件）
- `freqtrade.data.__init__` -- data 包入口导入 converter
- `freqtrade.data.history.history_utils` -- 历史数据工具使用转换函数
- `freqtrade.data.history.datahandlers.idatahandler` -- 数据处理器接口使用清洗和转换函数
- `freqtrade.exchange.exchange` -- 交易所模块使用 OHLCV 转换
- `freqtrade.optimize.backtesting` -- 回测引擎使用裁剪和转换
- `freqtrade.strategy.interface` -- 策略接口使用内存优化
- `freqtrade.commands.data_commands` -- CLI 数据命令使用格式转换
