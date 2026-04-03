# ft_types/__init__.py

## 概述

`ft_types` 包的初始化模块，从子模块中导出所有自定义类型定义，为项目提供统一的类型导入入口。这些类型主要用于回测结果、图表注释和交易所信息的结构化表示。

## 导出内容

| 导出名称 | 来源模块 | 说明 |
|----------|---------|------|
| `BacktestContentType` | `backtest_result_type` | 回测内容类型（完整） |
| `BacktestContentTypeIcomplete` | `backtest_result_type` | 回测内容类型（可选字段） |
| `BacktestHistoryEntryType` | `backtest_result_type` | 回测历史条目类型 |
| `BacktestMetadataType` | `backtest_result_type` | 回测元数据类型 |
| `BacktestResultType` | `backtest_result_type` | 回测结果类型 |
| `get_BacktestResultType_default` | `backtest_result_type` | 获取回测结果默认值的工厂函数 |
| `AnnotationType` | `plot_annotation_type` | 图表注释联合类型 |
| `TradeModeType` | `valid_exchanges_type` | 交易模式类型 |
| `ValidExchangesType` | `valid_exchanges_type` | 合法交易所类型 |

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.ft_types.backtest_result_type` -- 回测结果类型
- `freqtrade.ft_types.plot_annotation_type` -- 图表注释类型
- `freqtrade.ft_types.valid_exchanges_type` -- 交易所类型

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
- `freqtrade.rpc.rpc` -- RPC 接口
- `freqtrade.strategy.interface` -- 策略接口（AnnotationType）
- `freqtrade.optimize.backtesting` -- 回测引擎
- `freqtrade.optimize.optimize_reports` -- 回测报告
- `freqtrade.rpc.api_server.api_backtest` -- API 回测接口
- `freqtrade.rpc.api_server.api_schemas` -- API schema 定义
- `freqtrade.exchange.exchange_utils` -- 交易所工具
- `freqtrade.data.btanalysis.bt_fileutils` -- 回测文件工具
- `freqtrade.commands.list_commands` -- 列表命令
