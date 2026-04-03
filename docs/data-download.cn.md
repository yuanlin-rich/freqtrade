# 数据下载

## 获取回测和超参数优化的数据

要下载回测和超参数优化所需的数据（K线 / OHLCV），请使用 `freqtrade download-data` 命令。

如果未指定其他参数，freqtrade 将下载过去 30 天的 `"1m"` 和 `"5m"` 时间周期数据。
交易所和交易对将来自 `config.json`（如果使用 `-c/--config` 指定）。
如果未提供配置文件，则 `--exchange` 参数变为必填。

您可以使用相对时间范围（`--days 20`）或绝对起始点（`--timerange 20200101-`）。对于增量下载，应使用相对方式。

!!! Tip "提示：更新现有数据"
    如果您的数据目录中已有可用的回测数据，并且想将此数据刷新到今天，freqtrade 会自动计算现有交易对的缺失时间范围，下载将从最新的可用时间点开始直到"现在"，不需要 `--days` 或 `--timerange` 参数。Freqtrade 会保留现有数据，仅下载缺失的数据。
    如果您在插入没有数据的新交易对后更新现有数据，请使用 `--new-pairs-days xx` 参数。指定的天数将用于下载新交易对的数据，而旧交易对只会更新缺失的数据。

### 用法

--8<-- "commands/download-data.md"

!!! Tip "下载某一计价货币的所有数据"
    通常，您会想要下载特定计价货币的所有交易对数据。在这种情况下，您可以使用以下简写方式：
    `freqtrade download-data --exchange binance --pairs ".*/USDT" <...>`。提供的"pairs"字符串将被扩展为包含交易所上所有活跃的交易对。
    要同时下载不活跃（已下架）交易对的数据，请在命令中添加 `--include-inactive-pairs`。

!!! Note "启动周期"
    `download-data` 是一个与策略无关的命令。其设计思路是一次性下载大量数据，然后逐步增加存储的数据量。

    因此，`download-data` 不关心策略中定义的"startup-period"。如果回测应从特定时间点开始（同时考虑启动周期），用户需要自行下载额外的天数。

### 开始下载

一个非常简单的命令（假设有可用的 `config.json` 文件）如下所示。

```bash
freqtrade download-data --exchange binance
```

这将下载配置中定义的所有货币对的历史 K 线（OHLCV）数据。

或者，直接指定交易对

```bash
freqtrade download-data --exchange binance --pairs ETH/USDT XRP/USDT BTC/USDT
```

或使用正则表达式（在此例中，下载所有活跃的 USDT 交易对）

```bash
freqtrade download-data --exchange binance --pairs ".*/USDT"
```

### 其他说明

* 要使用与交易所特定默认值不同的目录，请使用 `--datadir user_data/data/some_directory`。
* 要更改用于下载历史数据的交易所，可以使用 `--exchange <exchange>` 或指定不同的配置文件。
* 要使用其他目录中的 `pairs.json`，请使用 `--pairs-file some_other_dir/pairs.json`。
* 要仅下载 10 天的历史 K 线（OHLCV）数据，请使用 `--days 10`（默认为 30 天）。
* 要从固定起始点下载历史 K 线（OHLCV）数据，请使用 `--timerange 20200101-`，这将下载从 2020 年 1 月 1 日起的所有数据。
* 如果数据已经存在，给定的起始点将被忽略，仅下载到今天为止的缺失数据。
* 使用 `--timeframes` 指定要下载哪些时间周期的历史 K 线（OHLCV）数据。默认值为 `--timeframes 1m 5m`，将下载 1 分钟和 5 分钟数据。
* 要使用配置文件中定义的交易所、时间周期和交易对列表，请使用 `-c/--config` 选项。使用此选项时，脚本将使用配置中定义的白名单作为要下载数据的货币对列表，不需要 pairs.json 文件。您可以将 `-c/--config` 与大多数其他选项组合使用。
* 下载期货数据时（`--trading-mode futures` 或指定期货模式的配置），freqtrade 将自动下载所需的 K 线类型（例如 `mark` 和 `funding_rate` K线），除非通过 `--candle-types` 另行指定。

??? Note "权限被拒绝错误"
    如果您的配置目录 `user_data` 是由 Docker 创建的，您可能会遇到以下错误：

    ```
    cp: cannot create regular file 'user_data/data/binance/pairs.json': Permission denied
    ```

    您可以按如下方式修复用户数据目录的权限：

    ```
    sudo chown -R $UID:$GID user_data
    ```

### 在当前时间范围之前下载额外数据

假设您下载了 2022 年的所有数据（`--timerange 20220101-`），但现在您还想使用更早的数据进行回测。
您可以通过使用 `--prepend` 标志并结合 `--timerange`（指定结束日期）来实现。

``` bash
freqtrade download-data --exchange binance --pairs ETH/USDT XRP/USDT BTC/USDT --prepend --timerange 20210101-20220101
```

!!! Note
    如果数据已存在，Freqtrade 将忽略此模式下的结束日期，将结束日期更新为现有数据的起始点。

### 数据格式

Freqtrade 目前支持以下数据格式：

* `feather` - 基于 Apache Arrow 的数据格式
* `json` - 纯文本 json 文件
* `jsongz` - gzip 压缩版本的 json 文件
* `parquet` - 列式数据存储（仅限 OHLCV）

默认情况下，OHLCV 数据和交易数据都以 `feather` 格式存储。

可以分别通过 `--data-format-ohlcv` 和 `--data-format-trades` 命令行参数进行更改。
要持久化此更改，您还应将以下代码片段添加到配置中，这样就不必每次都插入上述参数：

``` jsonc
    // ...
    "dataformat_ohlcv": "feather",
    "dataformat_trades": "feather",
    // ...
```

如果在下载过程中更改了默认数据格式，则配置文件中的 `dataformat_ohlcv` 和 `dataformat_trades` 键也需要调整为所选的数据格式。

!!! Note
    您可以使用 [convert-data](#sub-command-convert-data) 和 [convert-trade-data](#sub-command-convert-trade-data) 方法在数据格式之间进行转换。

#### 数据格式比较

以下比较使用了以下数据，并使用 Linux `time` 命令进行测量。

```
Found 6 pair / timeframe combinations.
+----------+-------------+--------+---------------------+---------------------+
|     Pair |   Timeframe |   Type |                From |                  To |
|----------+-------------+--------+---------------------+---------------------|
| BTC/USDT |          5m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:25:00 |
| ETH/USDT |          1m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:26:00 |
| BTC/USDT |          1m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:30:00 |
| XRP/USDT |          5m |   spot | 2018-05-04 08:10:00 | 2022-09-13 19:15:00 |
| XRP/USDT |          1m |   spot | 2018-05-04 08:11:00 | 2022-09-13 19:22:00 |
| ETH/USDT |          5m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:20:00 |
+----------+-------------+--------+---------------------+---------------------+
```

计时以非严格科学的方式使用以下命令进行测量，该命令强制将数据读入内存。

``` bash
time freqtrade list-data --show-timerange --data-format-ohlcv <dataformat>
```

|  格式 | 大小 | 耗时 |
|------------|-------------|-------------|
| `feather` | 72Mb | 3.5s |
| `json` | 149Mb | 25.6s |
| `jsongz` | 39Mb | 27s |
| `parquet` | 83Mb | 3.8s |

大小取自上述时间范围内 BTC/USDT 1m 现货组合。

为了获得最佳的性能/大小平衡，我们建议使用默认的 feather 格式或 parquet。

### 交易对文件

作为 `config.json` 白名单的替代方案，可以使用 `pairs.json` 文件。
如果您使用的是 Binance，例如：

* 创建目录 `user_data/data/binance` 并在该目录中复制或创建 `pairs.json` 文件。
* 更新 `pairs.json` 文件以包含您感兴趣的货币对。

```bash
mkdir -p user_data/data/binance
touch user_data/data/binance/pairs.json
```

`pairs.json` 文件的格式是一个简单的 JSON 列表。
此文件允许混合不同的计价货币，因为它仅用于下载。

``` json
[
    "ETH/BTC",
    "ETH/USDT",
    "BTC/USDT",
    "XRP/ETH"
]
```

!!! Note
    `pairs.json` 文件仅在未加载配置时使用（通过命名隐式加载或通过 `--config` 标志加载）。
    您可以通过 `--pairs-file pairs.json` 强制使用此文件，但我们建议使用配置中的交易对列表，可以通过 `exchange.pair_whitelist` 或配置中的 `pairs` 设置。

## Sub-command convert data

--8<-- "commands/convert-data.md"

### 转换数据示例

以下命令将把 `~/.freqtrade/data/binance` 中所有可用的 K 线（OHLCV）数据从 json 转换为 jsongz，从而节省磁盘空间。
它还会删除原始的 json 数据文件（`--erase` 参数）。

``` bash
freqtrade convert-data --format-from json --format-to jsongz --datadir ~/.freqtrade/data/binance -t 5m 15m --erase
```

## Sub-command convert trade data

--8<-- "commands/convert-trade-data.md"

### 转换交易数据示例

以下命令将把 `~/.freqtrade/data/kraken` 中所有可用的交易数据从 jsongz 转换为 json。
它还会删除原始的 jsongz 数据文件（`--erase` 参数）。

``` bash
freqtrade convert-trade-data --format-from jsongz --format-to json --datadir ~/.freqtrade/data/kraken --erase
```

## Sub-command trades to ohlcv

当您需要使用 `--dl-trades`（仅限 Kraken）下载数据时，将交易数据转换为 OHLCV 数据是最后一步。
此命令允许您在不重新下载数据的情况下为其他时间周期重复此最后一步。

--8<-- "commands/trades-to-ohlcv.md"

### 交易转 OHLCV 转换示例

``` bash
freqtrade trades-to-ohlcv --exchange kraken -t 5m 1h 1d --pairs BTC/EUR ETH/EUR
```

## Sub-command list-data

您可以使用 `list-data` 子命令获取已下载数据的列表。

--8<-- "commands/list-data.md"

### list-data 示例

```bash
> freqtrade list-data --userdir ~/.freqtrade/user_data/

              Found 33 pair / timeframe combinations.
┏━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━┓
┃          Pair ┃                                 Timeframe ┃ Type ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━┩
│       ADA/BTC │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│       ADA/ETH │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│       ETH/BTC │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│      ETH/USDT │                  5m, 15m, 30m, 1h, 2h, 4h │ spot │
└───────────────┴───────────────────────────────────────────┴──────┘

```

显示所有交易数据（包括起止时间范围）

``` bash
> freqtrade list-data --show --trades
                     Found trades data for 1 pair.
┏━━━━━━━━━┳━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━┓
┃    Pair ┃ Type ┃                From ┃                  To ┃ Trades ┃
┡━━━━━━━━━╇━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━┩
│ XRP/ETH │ spot │ 2019-10-11 00:00:11 │ 2019-10-13 11:19:28 │  12477 │
└─────────┴──────┴─────────────────────┴─────────────────────┴────────┘

```

## 逐笔交易（Tick）数据

默认情况下，`download-data` 子命令下载 K 线（OHLCV）数据。大多数交易所也通过其 API 提供历史逐笔交易数据。
如果您需要许多不同的时间周期，这些数据可能很有用，因为它只需下载一次，然后在本地重新采样为所需的时间周期。

由于这些数据默认较大，文件默认使用 feather 文件格式。它们存储在您的数据目录中，命名约定为 `<pair>-trades.feather`（`ETH_BTC-trades.feather`）。也支持增量模式，与历史 OHLCV 数据一样，因此每周使用 `--days 8` 下载一次数据将创建一个增量数据仓库。

要使用此模式，只需在您的调用中添加 `--dl-trades`。这将把下载方法切换为下载逐笔交易数据。
如果同时提供了 `--convert`，重新采样步骤将自动发生，并覆盖给定交易对/时间周期组合中可能已存在的 OHLCV 数据。

!!! Warning "请勿使用"
    除非您是 Kraken 用户（Kraken 不提供历史 OHLCV 数据），否则您不应使用此功能。
    大多数其他交易所提供具有足够历史记录的 OHLCV 数据，因此通过该方法下载多个时间周期仍然比下载逐笔交易数据快得多。

!!! Note "Kraken 用户"
    Kraken 用户在开始下载数据之前应阅读[此内容](exchanges.md#historic-kraken-data)。

    Kraken Futures 使用标准 OHLCV 下载，不需要 `--dl-trades`。

示例调用：

```bash
freqtrade download-data --exchange kraken --pairs XRP/EUR ETH/EUR --days 20 --dl-trades
```

!!! Note
    虽然此方法使用异步调用，但速度会很慢，因为它需要前一个调用的结果来生成对交易所的下一个请求。

## 下一步

很好，您现在已经下载了一些数据，可以开始对您的策略进行[回测](backtesting.md)了。
