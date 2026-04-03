``` output
usage: freqtrade hyperopt [-h] [-v] [--no-color] [--logfile FILE] [-V]
                          [-c PATH] [-d PATH] [--userdir PATH] [-s NAME]
                          [--strategy-path PATH] [--recursive-strategy-search]
                          [--freqaimodel NAME] [--freqaimodel-path PATH]
                          [-i TIMEFRAME] [--timerange TIMERANGE]
                          [--data-format-ohlcv {json,jsongz,feather,parquet}]
                          [--max-open-trades INT]
                          [--stake-amount STAKE_AMOUNT] [--fee FLOAT]
                          [-p PAIRS [PAIRS ...]] [--hyperopt-path PATH]
                          [--eps] [--enable-protections]
                          [--dry-run-wallet DRY_RUN_WALLET]
                          [--timeframe-detail TIMEFRAME_DETAIL] [-e INT]
                          [--spaces SPACES [SPACES ...]] [--print-all]
                          [--print-json] [-j JOBS] [--random-state INT]
                          [--min-trades INT] [--hyperopt-loss NAME]
                          [--disable-param-export] [--ignore-missing-spaces]
                          [--analyze-per-epoch] [--early-stop INT]

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
  --hyperopt-path PATH  指定超参数优化损失函数的额外查找路径。
  --eps, --enable-position-stacking
                        允许多次买入同一交易对（仓位叠加）。仅适用于
                        回测和超参数优化。由此归档的结果无法在
                        模拟/实盘交易中重现。
  --enable-protections, --enableprotections
                        为回测启用保护机制。将显著减慢回测速度，
                        但将包含已配置的保护机制。
  --dry-run-wallet, --starting-balance DRY_RUN_WALLET
                        起始余额，用于回测/超参数优化和模拟运行。
  --timeframe-detail TIMEFRAME_DETAIL
                        指定回测的详细时间框架（`1m`、`5m`、
                        `30m`、`1h`、`1d`）。
  -e, --epochs INT      指定轮次数量（默认值：100）。
  --spaces SPACES [SPACES ...]
                        指定要优化的参数。以空格分隔的列表。
                        可用的内置选项（自定义空间不会在此列出）：
                        default、all、buy、sell、enter、exit、roi、
                        stoploss、trailing、protection、trades。
                        默认值：`default` - 包含除 'trailing'、
                        'protection' 和 'trades' 之外的所有空间。
  --print-all           打印所有结果，不仅仅是最佳结果。
  --print-json          以 JSON 格式打印输出。
  -j, --job-workers JOBS
                        超参数优化的并发运行作业数量
                        （hyperopt 工作进程）。如果为 -1
                        （默认值），使用所有 CPU；如果为 -2，
                        使用除一个之外的所有 CPU，依此类推。
                        如果为 1，则完全不使用并行计算代码。
  --random-state INT    设置随机状态为某个正整数以获得
                        可重现的超参数优化结果。
  --min-trades INT      设置超参数优化路径中评估所需的
                        最小交易数量（默认值：1）。
  --hyperopt-loss, --hyperoptloss NAME
                        指定超参数优化损失函数的类名
                        (IHyperOptLoss)。不同的函数可能产生
                        完全不同的结果，因为优化目标不同。
                        内置的超参数优化损失函数包括：
                        ShortTradeDurHyperOptLoss、OnlyProfitHyperOptLoss、
                        SharpeHyperOptLoss、SharpeHyperOptLossDaily、
                        SortinoHyperOptLoss、SortinoHyperOptLossDaily、
                        CalmarHyperOptLoss、MaxDrawDownHyperOptLoss、
                        MaxDrawDownRelativeHyperOptLoss、
                        MaxDrawDownPerPairHyperOptLoss、
                        ProfitDrawDownHyperOptLoss、MultiMetricHyperOptLoss
  --disable-param-export
                        禁用自动超参数优化参数导出。
  --ignore-missing-spaces, --ignore-unparameterized-spaces
                        抑制不包含任何参数的已请求超参数优化
                        空间的错误。
  --analyze-per-epoch   每轮运行一次 populate_indicators。
  --early-stop INT      如果在（默认值：0）轮后没有改善，
                        则提前停止超参数优化。

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
