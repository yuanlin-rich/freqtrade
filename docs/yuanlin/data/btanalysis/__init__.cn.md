# __init__.py

## 概述

`freqtrade/data/btanalysis/__init__.py` 是回测分析子包的初始化文件。它从三个子模块中统一导出所有公共 API，使外部代码可以直接通过 `from freqtrade.data.btanalysis import ...` 访问所有回测分析功能。

## 导出内容

### 来自 `bt_fileutils` 模块
- `BT_DATA_COLUMNS` -- 回测数据列定义
- `delete_backtest_result` -- 删除回测结果文件
- `extract_trades_of_period` -- 提取指定时间段的交易
- `find_existing_backtest_stats` -- 查找已有回测统计
- `get_backtest_market_change` -- 获取回测市场变动数据
- `get_backtest_result` -- 获取单个回测结果
- `get_backtest_resultlist` -- 获取回测结果列表
- `get_latest_backtest_filename` -- 获取最新回测文件名
- `get_latest_hyperopt_file` -- 获取最新 hyperopt 文件路径
- `get_latest_hyperopt_filename` -- 获取最新 hyperopt 文件名
- `get_latest_optimize_filename` -- 获取最新优化文件名
- `load_and_merge_backtest_result` -- 加载并合并回测结果
- `load_backtest_analysis_data` -- 加载回测分析数据（信号/拒绝/退出）
- `load_backtest_data` -- 加载回测交易数据
- `load_backtest_metadata` -- 加载回测元数据
- `load_backtest_stats` -- 加载回测统计信息
- `load_file_from_zip` -- 从 zip 文件中加载数据
- `load_trades` -- 根据来源加载交易数据
- `load_trades_from_db` -- 从数据库加载交易数据
- `trade_list_to_dataframe` -- Trade 对象列表转 DataFrame
- `update_backtest_metadata` -- 更新回测元数据

### 来自 `historic_precision` 模块
- `get_tick_size_over_time` -- 获取历史 tick size 变化

### 来自 `trade_parallelism` 模块
- `analyze_trade_parallelism` -- 分析交易并行度
- `evaluate_result_multi` -- 评估多交易对并行结果

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.data.btanalysis.bt_fileutils` -- 回测文件工具
- `freqtrade.data.btanalysis.historic_precision` -- 历史精度分析
- `freqtrade.data.btanalysis.trade_parallelism` -- 交易并行度分析

### 被依赖（谁引用了本文件）
- `freqtrade.data.entryexitanalysis` -- 入场/出场分析
- `freqtrade.optimize.backtesting` -- 回测引擎
- `freqtrade.plot.plotting` -- 绘图模块
- `freqtrade.rpc.api_server.api_backtest` -- API 回测接口
- `freqtrade.commands.optimize_commands` -- CLI 优化命令
- `freqtrade.commands.hyperopt_commands` -- CLI hyperopt 命令
