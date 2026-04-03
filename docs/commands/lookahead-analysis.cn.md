``` output
usage: freqtrade lookahead-analysis [-h] [-v] [--no-color] [--logfile FILE]
                                    [-V] [-c PATH] [-d PATH] [--userdir PATH]
                                    [-s NAME] [--strategy-path PATH]
                                    [--recursive-strategy-search]
                                    [--freqaimodel NAME]
                                    [--freqaimodel-path PATH] [-i TIMEFRAME]
                                    [--timerange TIMERANGE]
                                    [--data-format-ohlcv {json,jsongz,feather,parquet}]
                                    [--max-open-trades INT]
                                    [--stake-amount STAKE_AMOUNT]
                                    [--fee FLOAT] [-p PAIRS [PAIRS ...]]
                                    [--enable-protections]
                                    [--enable-dynamic-pairlist]
                                    [--dry-run-wallet DRY_RUN_WALLET]
                                    [--timeframe-detail TIMEFRAME_DETAIL]
                                    [--strategy-list STRATEGY_LIST [STRATEGY_LIST ...]]
                                    [--export {none,trades,signals}]
                                    [--backtest-filename PATH]
                                    [--backtest-directory PATH]
                                    [--freqai-backtest-live-models]
                                    [--minimum-trade-amount INT]
                                    [--targeted-trade-amount INT]
                                    [--lookahead-analysis-exportfilename LOOKAHEAD_ANALYSIS_EXPORTFILENAME]
                                    [--allow-limit-orders]

options:
  -h, --help            显示此帮助信息并退出
  -i, --timeframe TIMEFRAME
                        指定时间周期（`1m`、`5m`、`30m`、`1h`、`1d`）。
  --timerange TIMERANGE
                        指定要使用的数据时间范围。
  --data-format-ohlcv {json,jsongz,feather,parquet}
                        下载的蜡烛图（OHLCV）数据的存储格式。
                        （默认值：`feather`）。
  --max-open-trades INT
                        覆盖 `max_open_trades` 配置设置的值。
  --stake-amount STAKE_AMOUNT
                        覆盖 `stake_amount` 配置设置的值。
  --fee FLOAT           指定手续费比率。将应用两次（在交易进入和退出时）。
  -p, --pairs PAIRS [PAIRS ...]
                        将命令限制为这些交易对。交易对以空格分隔。
  --enable-protections, --enableprotections
                        为回测启用保护。这将大大减慢回测速度，
                        但会包含配置的保护
  --enable-dynamic-pairlist
                        在回测中启用动态交易对列表刷新。如果您使用
                        支持此功能的交易对列表处理程序（例如 ShuffleFilter），
                        则将为每个新蜡烛图生成交易对列表。
  --dry-run-wallet, --starting-balance DRY_RUN_WALLET
                        起始余额，用于回测/超参数优化和模拟运行。
  --timeframe-detail TIMEFRAME_DETAIL
                        指定回测的详细时间周期（`1m`、`5m`、
                        `30m`、`1h`、`1d`）。
  --strategy-list STRATEGY_LIST [STRATEGY_LIST ...]
                        提供要回测的策略的空格分隔列表。
                        请注意，时间周期需要在配置中或通过命令行设置。
  --export {none,trades,signals}
                        导出回测结果（默认值：trades）。
  --backtest-filename, --export-filename PATH
                        使用此文件名作为回测结果。例如：
                        `--backtest-
                        filename=backtest_results_2020-09-27_16-20-48.json`。
                        假定 `user_data/backtest_results/` 或
                        `--export-directory` 为基础目录。
  --backtest-directory, --export-directory PATH
                        用于回测结果的目录。例如：
                        `--export-directory=user_data/backtest_results/`。
  --freqai-backtest-live-models
                        使用准备好的模型运行回测。
  --minimum-trade-amount INT
                        前瞻分析的最小交易数量
  --targeted-trade-amount INT
                        前瞻分析的目标交易数量
  --lookahead-analysis-exportfilename LOOKAHEAD_ANALYSIS_EXPORTFILENAME
                        使用此 csv 文件名存储前瞻分析结果
  --allow-limit-orders  在前瞻分析中允许限价订单（可能导致
                        前瞻分析结果中的误报）。

Common arguments:
  -v, --verbose         详细模式（-vv 获取更多信息，-vvv 获取所有消息）。
  --no-color            禁用 hyperopt 结果的着色。如果您将输出重定向到
                        文件，这可能很有用。
  --logfile, --log-file FILE
                        记录到指定的文件。特殊值为：
                        'syslog'、'journald'。有关更多详细信息，请参阅文档。
  -V, --version         显示程序的版本号并退出
  -c, --config PATH     指定配置文件（默认值：
                        `userdir/config.json` 或 `config.json`，以存在的为准）。
                        可以使用多个 --config 选项。可以设置为 `-` 以从 stdin 读取配置。
  -d, --datadir, --data-dir PATH
                        包含历史回测数据的交易所基础目录的路径。
                        要查看期货数据，请额外使用 trading-mode。
  --userdir, --user-data-dir PATH
                        用户数据目录的路径。

Strategy arguments:
  -s, --strategy NAME   指定机器人将使用的策略类名称。
  --strategy-path PATH  指定额外的策略查找路径。
  --recursive-strategy-search
                        在策略文件夹中递归搜索策略。
  --freqaimodel NAME    指定自定义 freqaimodels。
  --freqaimodel-path PATH
                        指定 freqaimodels 的额外查找路径。

```
