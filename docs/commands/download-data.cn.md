``` output
usage: freqtrade download-data [-h] [-v] [--no-color] [--logfile FILE] [-V]
                               [-c PATH] [-d PATH] [--userdir PATH]
                               [-p PAIRS [PAIRS ...]] [--pairs-file FILE]
                               [--days INT] [--new-pairs-days INT]
                               [--include-inactive-pairs]
                               [--no-parallel-download]
                               [--timerange TIMERANGE] [--dl-trades]
                               [--convert] [--exchange EXCHANGE]
                               [-t TIMEFRAMES [TIMEFRAMES ...]] [--erase]
                               [--data-format-ohlcv {json,jsongz,feather,parquet}]
                               [--data-format-trades {json,jsongz,feather,parquet}]
                               [--trading-mode {spot,margin,futures}]
                               [--candle-types {spot,futures,mark,index,premiumIndex,funding_rate} [{spot,futures,mark,index,premiumIndex,funding_rate} ...]]
                               [--prepend]

options:
  -h, --help            显示此帮助信息并退出
  -p, --pairs PAIRS [PAIRS ...]
                        将命令限制为这些交易对。交易对以空格分隔。
  --pairs-file FILE     包含交易对列表的文件。优先于 --pairs 或配置中配置的交易对。
  --days INT            下载指定天数的数据。
  --new-pairs-days INT  为新交易对下载指定天数的数据。
                        默认值：`None`。
  --include-inactive-pairs
                        同时下载非活跃交易对的数据。
  --no-parallel-download
                        禁用并行启动下载。仅在遇到问题时使用。
  --timerange TIMERANGE
                        指定要使用的数据时间范围。
  --dl-trades           下载交易数据而不是 OHLCV 数据。
  --convert             将下载的交易数据转换为 OHLCV 数据。仅适用于
                        与 `--dl-trades` 结合使用。对于没有历史
                        OHLCV 的交易所（例如 Kraken）将自动执行。如果未提供，
                        使用 `trades-to-ohlcv` 将交易数据转换为 OHLCV 数据。
  --exchange EXCHANGE   交易所名称。仅在未提供配置时有效。
  -t, --timeframes TIMEFRAMES [TIMEFRAMES ...]
                        指定要下载的时间周期。以空格分隔的列表。
                        默认值：`1m 5m`。
  --erase               清除所选交易所/交易对/时间周期的所有现有数据。
  --data-format-ohlcv {json,jsongz,feather,parquet}
                        下载的蜡烛图（OHLCV）数据的存储格式。
                        （默认值：`feather`）。
  --data-format-trades {json,jsongz,feather,parquet}
                        下载的交易数据的存储格式。（默认值：
                        `feather`）。
  --trading-mode, --tradingmode {spot,margin,futures}
                        选择交易模式
  --candle-types {spot,futures,mark,index,premiumIndex,funding_rate} [{spot,futures,mark,index,premiumIndex,funding_rate} ...]
                        选择要下载的蜡烛图类型。默认为所选交易模式所需的
                        蜡烛图（例如 'spot' 或 ('futures', 'funding_rate' 和 'mark')
                        用于期货）。
  --prepend             允许数据前置。（数据追加已禁用）

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
