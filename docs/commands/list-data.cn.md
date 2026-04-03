``` output
usage: freqtrade list-data [-h] [-v] [--no-color] [--logfile FILE] [-V]
                           [-c PATH] [-d PATH] [--userdir PATH]
                           [--exchange EXCHANGE]
                           [--data-format-ohlcv {json,jsongz,feather,parquet}]
                           [--data-format-trades {json,jsongz,feather,parquet}]
                           [--trades] [-p PAIRS [PAIRS ...]]
                           [--trading-mode {spot,margin,futures}]
                           [--show-timerange]

options:
  -h, --help            显示此帮助信息并退出
  --exchange EXCHANGE   交易所名称。仅在未提供配置时有效。
  --data-format-ohlcv {json,jsongz,feather,parquet}
                        下载的蜡烛图（OHLCV）数据的存储格式。
                        （默认值：`feather`）。
  --data-format-trades {json,jsongz,feather,parquet}
                        下载的交易数据的存储格式。（默认值：
                        `feather`）。
  --trades              处理交易数据而不是 OHLCV 数据。
  -p, --pairs PAIRS [PAIRS ...]
                        将命令限制为这些交易对。交易对以空格分隔。
  --trading-mode, --tradingmode {spot,margin,futures}
                        选择交易模式
  --show-timerange      显示可用数据的时间范围。（可能需要一段时间来计算）。

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

```
