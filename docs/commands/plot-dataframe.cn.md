``` output
usage: freqtrade plot-dataframe [-h] [-v] [--no-color] [--logfile FILE] [-V]
                                [-c PATH] [-d PATH] [--userdir PATH] [-s NAME]
                                [--strategy-path PATH]
                                [--recursive-strategy-search]
                                [--freqaimodel NAME] [--freqaimodel-path PATH]
                                [-p PAIRS [PAIRS ...]]
                                [--indicators1 INDICATORS1 [INDICATORS1 ...]]
                                [--indicators2 INDICATORS2 [INDICATORS2 ...]]
                                [--plot-limit INT] [--db-url PATH]
                                [--trade-source {DB,file}]
                                [--export {none,trades,signals}]
                                [--backtest-filename PATH]
                                [--timerange TIMERANGE] [-i TIMEFRAME]
                                [--no-trades]

options:
  -h, --help            显示此帮助信息并退出
  -p, --pairs PAIRS [PAIRS ...]
                        将命令限制为这些交易对。交易对以空格分隔。
  --indicators1 INDICATORS1 [INDICATORS1 ...]
                        设置您希望在图表第一行显示的策略指标。
                        以空格分隔的列表。例如：
                        `ema3 ema5`。默认值：`['sma', 'ema3', 'ema5']`。
  --indicators2 INDICATORS2 [INDICATORS2 ...]
                        设置您希望在图表第三行显示的策略指标。
                        以空格分隔的列表。例如：
                        `fastd fastk`。默认值：`['macd', 'macdsignal']`。
  --plot-limit INT      指定绘图的刻度限制。注意：过高的值会导致
                        文件过大。默认值：750。
  --db-url PATH         覆盖交易数据库 URL，这在自定义部署中很有用
                        （默认值：实盘运行模式为 `sqlite:///tradesv3.sqlite`，
                        模拟运行为 `sqlite:///tradesv3.dryrun.sqlite`）。
  --trade-source {DB,file}
                        指定交易的来源（可以是 DB 或 file（回测文件））
                        默认值：file
  --export {none,trades,signals}
                        导出回测结果（默认值：trades）。
  --backtest-filename, --export-filename PATH
                        使用此文件名作为回测结果。例如：
                        `--backtest-
                        filename=backtest_results_2020-09-27_16-20-48.json`。
                        假定 `user_data/backtest_results/` 或
                        `--export-directory` 为基础目录。
  --timerange TIMERANGE
                        指定要使用的数据时间范围。
  -i, --timeframe TIMEFRAME
                        指定时间周期（`1m`、`5m`、`30m`、`1h`、`1d`）。
  --no-trades           跳过使用回测文件和数据库中的交易。

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
