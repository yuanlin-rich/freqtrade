# optimize_reports/__init__.py

## 概述

`freqtrade/optimize/optimize_reports/__init__.py` 是 optimize_reports 子包的初始化文件，负责从子模块导入并统一导出回测报告相关的所有公共函数。作为该包的门面（Facade），使得外部模块可以通过 `from freqtrade.optimize.optimize_reports import ...` 直接访问所有功能。

## 导出内容

### 从 bt_output 导出（终端显示相关）
| 函数名 | 说明 |
|--------|------|
| `generate_wins_draws_losses` | 格式化胜/平/负统计 |
| `show_backtest_result` | 显示单个策略的回测结果 |
| `show_backtest_results` | 显示所有策略的回测结果 |
| `show_sorted_pairlist` | 按利润排序显示交易对 |
| `text_table_add_metrics` | 显示汇总指标表格 |
| `text_table_bt_results` | 显示回测结果表格 |
| `text_table_periodic_breakdown` | 显示按周期分解的表格 |
| `text_table_strategy` | 显示策略比较表格 |
| `text_table_tags` | 显示标签统计表格 |

### 从 bt_storage 导出（存储相关）
| 函数名 | 说明 |
|--------|------|
| `store_backtest_results` | 存储回测结果到 zip 文件 |

### 从 optimize_reports 导出（数据生成相关）
| 函数名 | 说明 |
|--------|------|
| `generate_all_periodic_breakdown_stats` | 生成所有周期的分解统计 |
| `generate_backtest_stats` | 生成完整回测统计 |
| `generate_daily_stats` | 生成每日统计 |
| `generate_pair_metrics` | 生成交易对指标 |
| `generate_periodic_breakdown_stats` | 生成指定周期的分解统计 |
| `generate_rejected_signals` | 生成被拒绝信号 |
| `generate_strategy_comparison` | 生成策略比较 |
| `generate_strategy_stats` | 生成策略统计 |
| `generate_tag_metrics` | 生成标签指标 |
| `generate_trade_signal_candles` | 生成交易信号 K 线 |
| `generate_trading_stats` | 生成交易统计 |

## 依赖关系

### 被依赖（谁引用了本文件）
- `freqtrade.optimize.backtesting` — 回测引擎导入报告生成和展示函数
- `freqtrade.optimize.hyperopt.hyperopt_optimizer` — Hyperopt 导入统计生成函数
- `freqtrade.optimize.hyperopt.hyperopt_output` — Hyperopt 输出导入展示函数
- `freqtrade.rpc.api_server.api_backtest` — API 服务器导入结果处理函数
- `freqtrade.commands.hyperopt_commands` — CLI 命令导入展示函数
- `freqtrade.commands.optimize_commands` — CLI 命令导入展示函数
