``` output
usage: freqtrade backtesting-show [-h] [-v] [--no-color] [--logfile FILE] [-V]
                                  [-c PATH] [-d PATH] [--userdir PATH]
                                  [--backtest-filename PATH]
                                  [--backtest-directory PATH]
                                  [--show-pair-list]
                                  [--breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]]

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
  --show-pair-list      显示按利润排序的回测交易对列表。
  --breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]
                        按 [天、周、月、年、星期几] 显示回测细分。

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
