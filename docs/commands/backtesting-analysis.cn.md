``` output
usage: freqtrade backtesting-analysis [-h] [-v] [--no-color] [--logfile FILE]
                                      [-V] [-c PATH] [-d PATH]
                                      [--userdir PATH]
                                      [--backtest-filename PATH]
                                      [--backtest-directory PATH]
                                      [--analysis-groups {0,1,2,3,4,5} [{0,1,2,3,4,5} ...]]
                                      [--enter-reason-list ENTER_REASON_LIST [ENTER_REASON_LIST ...]]
                                      [--exit-reason-list EXIT_REASON_LIST [EXIT_REASON_LIST ...]]
                                      [--indicator-list INDICATOR_LIST [INDICATOR_LIST ...]]
                                      [--entry-only] [--exit-only]
                                      [--timerange TIMERANGE]
                                      [--rejected-signals] [--analysis-to-csv]
                                      [--analysis-csv-path ANALYSIS_CSV_PATH]

选项:
  -h, --help            显示帮助信息并退出
  --backtest-filename, --export-filename PATH
                        使用此文件名作为回测结果。示例：
                        `--backtest-
                        filename=backtest_results_2020-09-27_16-20-48.json`。
                        假定 `user_data/backtest_results/` 或
                        `--export-directory` 为基础目录。
  --backtest-directory, --export-directory PATH
                        用于回测结果的目录。示例：
                        `--export-directory=user_data/backtest_results/`。
  --analysis-groups {0,1,2,3,4,5} [{0,1,2,3,4,5} ...]
                        分组输出 - 0: 按入场标签的简单胜负统计,
                        1: 按 enter_tag, 2: 按 enter_tag 和 exit_tag,
                        3: 按交易对和 enter_tag, 4: 按交易对、enter_
                        和 exit_tag（结果可能非常大），5: 按 exit_tag
  --enter-reason-list ENTER_REASON_LIST [ENTER_REASON_LIST ...]
                        以空格分隔的入场信号列表用于分析。
                        默认值: all。例如 'entry_tag_a entry_tag_b'
  --exit-reason-list EXIT_REASON_LIST [EXIT_REASON_LIST ...]
                        以空格分隔的退出信号列表用于分析。
                        默认值: all。例如 'exit_tag_a roi stop_loss
                        trailing_stop_loss'
  --indicator-list INDICATOR_LIST [INDICATOR_LIST ...]
                        以空格分隔的指标列表用于分析。例如
                        'close rsi bb_lowerband profit_abs'
  --entry-only          仅分析入场信号。
  --exit-only           仅分析退出信号。
  --timerange TIMERANGE
                        指定要使用的数据时间范围。
  --rejected-signals    分析被拒绝的信号
  --analysis-to-csv     将选定的分析表保存为单独的 CSV 文件
  --analysis-csv-path ANALYSIS_CSV_PATH
                        如果启用了 --analysis-to-csv，指定保存
                        分析 CSV 的路径。默认值：
                        user_data/basktesting_results/

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

```
