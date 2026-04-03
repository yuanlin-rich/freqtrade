``` output
usage: freqtrade backtesting [-h] [-v] [--no-color] [--logfile FILE] [-V]
                             [-c PATH] [-d PATH] [--userdir PATH] [-s NAME]
                             [--strategy-path PATH]
                             [--recursive-strategy-search]
                             [--freqaimodel NAME] [--freqaimodel-path PATH]
                             [-i TIMEFRAME] [--timerange TIMERANGE]
                             [--data-format-ohlcv {json,jsongz,feather,parquet}]
                             [--max-open-trades INT]
                             [--stake-amount STAKE_AMOUNT] [--fee FLOAT]
                             [-p PAIRS [PAIRS ...]] [--eps]
                             [--enable-protections]
                             [--enable-dynamic-pairlist]
                             [--dry-run-wallet DRY_RUN_WALLET]
                             [--timeframe-detail TIMEFRAME_DETAIL]
                             [--strategy-list STRATEGY_LIST [STRATEGY_LIST ...]]
                             [--export {none,trades,signals}]
                             [--backtest-filename PATH]
                             [--backtest-directory PATH]
                             [--breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]]
                             [--cache {none,day,week,month}]
                             [--freqai-backtest-live-models] [--notes TEXT]

选项:
  -h, --help            显示帮助信息并退出
  -i, --timeframe TIMEFRAME
                        指定时间框架（`1m`、`5m`、`30m`、`1h`、`1d`）。
  --timerange TIMERANGE
                        指定要使用的数据时间范围。
  --data-format-ohlcv {json,jsongz,feather,parquet}
                        已下载的蜡烛图 (OHLCV) 数据的存储格式。
                        （默认值：`feather`）。
  --max-open-trades INT
                        覆盖 `max_open_trades` 配置项的值。
  --stake-amount STAKE_AMOUNT
                        覆盖 `stake_amount` 配置项的值。
  --fee FLOAT           指定手续费比率。将应用两次（交易入场和退出时）。
  -p, --pairs PAIRS [PAIRS ...]
                        将命令限制在这些交易对。交易对以空格分隔。
  --eps, --enable-position-stacking
                        允许多次买入同一交易对（仓位叠加）。仅适用于
                        回测和超参数优化。由此归档的结果无法在
                        模拟/实盘交易中重现。
  --enable-protections, --enableprotections
                        为回测启用保护机制。将显著减慢回测速度，
                        但将包含已配置的保护机制。
  --enable-dynamic-pairlist
                        在回测中启用动态交易对列表刷新。如果你使用
                        支持此功能的交易对列表处理器（例如
                        ShuffleFilter），则会在每根新蜡烛图时
                        生成交易对列表。
  --dry-run-wallet, --starting-balance DRY_RUN_WALLET
                        起始余额，用于回测/超参数优化和模拟运行。
  --timeframe-detail TIMEFRAME_DETAIL
                        指定回测的详细时间框架（`1m`、`5m`、
                        `30m`、`1h`、`1d`）。
  --strategy-list STRATEGY_LIST [STRATEGY_LIST ...]
                        提供以空格分隔的策略列表进行回测。请注意
                        时间框架需要在配置或命令行中设置。
  --export {none,trades,signals}
                        导出回测结果（默认值：trades）。
  --backtest-filename, --export-filename PATH
                        已弃用：此选项在回测中已弃用，将在未来版本中
                        移除。不再支持使用自定义文件名保存回测结果。
                        请使用 `--backtest-directory` 指定目录。
  --backtest-directory, --export-directory PATH
                        用于回测结果的目录。示例：
                        `--export-directory=user_data/backtest_results/`。
  --breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]
                        按 [天、周、月、年、星期几] 显示回测细分。
  --cache {none,day,week,month}
                        加载不超过指定时间的缓存回测结果
                        （默认值：day）。
  --freqai-backtest-live-models
                        使用已准备好的模型运行回测。
  --notes TEXT          为回测结果添加备注。

通用参数:
  -v, --verbose         详细模式（-vv 显示更多，-vvv 获取所有消息）。
  --no-color            禁用超参数优化结果的着色。如果你将输出重定向到
                        文件，这可能很有用。
  --logfile, --log-file FILE
                        将日志记录到指定文件。特殊值包括：
                        'syslog'、'journald'。详情请参阅文档。
  -V, --version         显示程序版本号并退出
  -c, --config PATH     指定配置文件（默认值：
                        `userdir/config.json` 或 `config.json`，
                        以存在的为准）。可以使用多个 --config 选项。
                        可以设置为 `-` 从标准输入读取配置。
  -d, --datadir, --data-dir PATH
                        包含历史回测数据的交易所基础目录路径。
                        要查看合约数据，需额外使用 trading-mode。
  --userdir, --user-data-dir PATH
                        用户数据目录路径。

策略参数:
  -s, --strategy NAME   指定机器人将使用的策略类名。
  --strategy-path PATH  指定额外的策略查找路径。
  --recursive-strategy-search
                        在策略文件夹中递归搜索策略。
  --freqaimodel NAME    指定自定义 freqaimodels。
  --freqaimodel-path PATH
                        指定 freqaimodels 的额外查找路径。

```
