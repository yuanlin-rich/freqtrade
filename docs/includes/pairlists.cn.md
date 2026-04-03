## 交易对列表和交易对列表处理器

交易对列表处理器（Pairlist Handlers）定义了机器人应该交易的交易对列表（pairlist）。它们在配置的 `pairlists` 部分中进行配置。

在配置中，你可以使用静态交易对列表（由 [`StaticPairList`](#static-pair-list) 处理器定义）和动态交易对列表（由 [`VolumePairList`](#volume-pair-list)、[`CrossMarketPairList`](#crossmarketpairlist)、[`MarketCapPairlist`](#marketcappairlist) 和 [`PercentChangePairList`](#percent-change-pair-list) 处理器定义）。

此外，[`AgeFilter`](#agefilter)、[`DelistFilter`](#delistfilter)、[`PrecisionFilter`](#precisionfilter)、[`PriceFilter`](#pricefilter)、[`ShuffleFilter`](#shufflefilter)、[`SpreadFilter`](#spreadfilter) 和 [`VolatilityFilter`](#volatilityfilter) 作为交易对过滤器，用于移除某些交易对和/或调整它们在列表中的位置。

如果使用了多个交易对列表处理器，它们会被链式执行，所有处理器的组合结果构成机器人用于交易和回测的最终交易对列表。处理器按照配置的顺序依次执行。你可以将 `StaticPairList`、`VolumePairList`、`ProducerPairList`、`RemotePairList`、`MarketCapPairList`、`PercentChangePairList` 或 `CrossMarketPairList` 定义为起始处理器。

非活跃市场的交易对始终会从结果列表中移除。明确列入黑名单的交易对（在 `pair_blacklist` 配置中的交易对）也始终会从结果列表中移除。

### 交易对黑名单

交易对黑名单（通过配置中的 `exchange.pair_blacklist` 配置）禁止某些交易对参与交易。
这可以很简单，比如排除 `DOGE/BTC` - 这将精确移除该交易对。

交易对黑名单也支持通配符（正则表达式风格）- 所以 `BNB/.*` 将排除所有以 BNB 开头的交易对。
你也可以使用类似 `.*DOWN/BTC` 或 `.*UP/BTC` 的表达式来排除杠杆代币（请检查你所在交易所的交易对命名约定！）

### 可用的交易对列表处理器

* [`StaticPairList`](#static-pair-list)（默认，如果没有其他配置）
* [`VolumePairList`](#volume-pair-list)
* [`PercentChangePairList`](#percent-change-pair-list)
* [`ProducerPairList`](#producerpairlist)
* [`RemotePairList`](#remotepairlist)
* [`MarketCapPairList`](#marketcappairlist)
* [`CrossMarketPairList`](#crossmarketpairlist)
* [`AgeFilter`](#agefilter)
* [`DelistFilter`](#delistfilter)
* [`FullTradesFilter`](#fulltradesfilter)
* [`OffsetFilter`](#offsetfilter)
* [`PerformanceFilter`](#performancefilter)
* [`PrecisionFilter`](#precisionfilter)
* [`PriceFilter`](#pricefilter)
* [`ShuffleFilter`](#shufflefilter)
* [`SpreadFilter`](#spreadfilter)
* [`RangeStabilityFilter`](#rangestabilityfilter)
* [`VolatilityFilter`](#volatilityfilter)

!!! Tip "测试交易对列表"
    交易对列表配置可能相当复杂。最好使用 [webserver 模式](freq-ui.md#webserver-mode)下的 freqUI 或 [`test-pairlist`](utils.md#test-pairlist) 工具子命令来快速测试你的交易对列表配置。

#### Static Pair List

默认情况下使用 `StaticPairList` 方法，它使用配置中静态定义的交易对白名单。该列表也支持通配符（正则表达式风格）- 所以 `.*/BTC` 将包含所有以 BTC 作为质押币的交易对。

它使用 `exchange.pair_whitelist` 和 `exchange.pair_blacklist` 的配置，在下面的示例中，将交易 BTC/USDT 和 ETH/USDT - 并阻止 BNB/USDT 的交易。

`pair_*list` 参数都支持正则表达式 - 所以类似 `.*/USDT` 的值将允许交易所有不在黑名单中的交易对。

```json
"exchange": {
    "name": "...",
    // ...
    "pair_whitelist": [
        "BTC/USDT",
        "ETH/USDT",
        // ...
    ],
    "pair_blacklist": [
        "BNB/USDT",
        // ...
    ]
},
"pairlists": [
    {"method": "StaticPairList"}
],
```

默认情况下，只允许当前启用的交易对。
要跳过对活跃市场的交易对验证，请在 `StaticPairList` 配置中设置 `"allow_inactive": true`。
这对于回测已过期的交易对（如季度现货市场）很有用。

当用于"后续"位置时（例如在 VolumePairlist 之后），`'pair_whitelist'` 中的所有交易对将被添加到列表末尾。

#### Volume Pair List

`VolumePairList` 使用交易量对交易对进行排序/过滤。它根据 `sort_key`（只能是 `quoteVolume`）选择交易量排名前 `number_assets` 的交易对。

当在非首位位置的交易对列表处理器链中使用时（在 StaticPairList 和其他过滤器之后），`VolumePairList` 会考虑前面处理器的输出，并按交易量对交易对进行排序/选择。

当用于处理器链的首位时，`pair_whitelist` 配置将被忽略。`VolumePairList` 会从交易所所有具有匹配质押货币的可用市场中选择排名靠前的资产。

`refresh_period` 设置允许定义刷新交易对列表的周期（以秒为单位）。默认为 1800 秒（30 分钟）。
`VolumePairList` 的交易对列表缓存（`refresh_period`）仅适用于生成交易对列表。
过滤实例（不在列表首位）不会应用任何缓存（除了在高级模式下缓存蜡烛图数据的持续时间），并且始终使用最新数据。

`VolumePairList` 默认基于交易所的行情数据，由 ccxt 库提供：

* `quoteVolume` 是过去 24 小时内交易（买入或卖出）的报价（质押）货币数量。

```json
"pairlists": [
    {
        "method": "VolumePairList",
        "number_assets": 20,
        "sort_key": "quoteVolume",
        "min_value": 0,
        "max_value": 8000000,
        "refresh_period": 1800
    }
],
```

你可以使用 `min_value` 定义最小交易量 - 这将过滤掉在指定时间范围内交易量低于指定值的交易对。
此外，你还可以使用 `max_value` 定义最大交易量 - 这将过滤掉在指定时间范围内交易量高于指定值的交易对。

##### VolumePairList 高级模式

`VolumePairList` 还可以在高级模式下运行，基于指定蜡烛图大小的给定时间范围构建交易量。它利用交易所的历史蜡烛图数据，构建典型价格（通过 (open+high+low)/3 计算），并将典型价格与每根蜡烛图的交易量相乘。总和即为给定范围内的 `quoteVolume`。这允许不同的场景：使用较大蜡烛图的较长范围可以获得更平滑的交易量，反之使用较小蜡烛图的较短范围则相反。

为方便起见，可以指定 `lookback_days`，这意味着将使用 1d 蜡烛图进行回溯。在下面的示例中，交易对列表将基于过去 7 天的数据创建：

```json
"pairlists": [
    {
        "method": "VolumePairList",
        "number_assets": 20,
        "sort_key": "quoteVolume",
        "min_value": 0,
        "refresh_period": 86400,
        "lookback_days": 7
    }
],
```

!!! Warning "范围回溯和刷新周期"
    当与 `lookback_days` 和 `lookback_timeframe` 一起使用时，`refresh_period` 不能小于蜡烛图大小的秒数。否则会导致对交易所 API 的不必要请求。

!!! Warning "使用回溯范围时的性能影响"
    如果在首位与回溯功能结合使用，基于范围的交易量计算可能会消耗大量时间和资源，因为它会下载所有可交易对的蜡烛图数据。因此，强烈建议使用标准方法配合 `VolumeFilter` 来缩小交易对列表范围，然后再进行进一步的范围交易量计算。

??? Tip "不支持的交易所"
    在某些交易所（如 Gemini）上，常规 VolumePairList 无法工作，因为 API 原生不提供 24 小时交易量。可以通过使用蜡烛图数据来构建交易量来解决此问题。
    要大致模拟 24 小时交易量，可以使用以下配置。
    请注意，这些交易对列表每天只会刷新一次。

    ```json
    "pairlists": [
        {
            "method": "VolumePairList",
            "number_assets": 20,
            "sort_key": "quoteVolume",
            "min_value": 0,
            "refresh_period": 86400,
            "lookback_days": 1
        }
    ],
    ```

更复杂的方法可以使用 `lookback_timeframe` 指定蜡烛图大小，使用 `lookback_period` 指定蜡烛图数量。以下示例将基于 3 天的 1 小时蜡烛图的滚动周期构建交易量交易对：

```json
"pairlists": [
    {
        "method": "VolumePairList",
        "number_assets": 20,
        "sort_key": "quoteVolume",
        "min_value": 0,
        "refresh_period": 3600,
        "lookback_timeframe": "1h",
        "lookback_period": 72
    }
],
```

!!! Note
    `VolumePairList` 不支持回测模式。

#### Percent Change Pair List

`PercentChangePairList` 根据过去 24 小时或任何定义的时间范围（作为高级选项的一部分）内价格的百分比变化来过滤和排序交易对。这使交易者能够专注于经历了显著价格波动（无论是正向还是负向）的资产。

**配置选项**

* `number_assets`：指定根据 24 小时百分比变化选择的顶部交易对数量。
* `min_value`：设置最小百分比变化阈值。百分比变化低于此值的交易对将被过滤掉。
* `max_value`：设置最大百分比变化阈值。百分比变化高于此值的交易对将被过滤掉。
* `sort_direction`：指定交易对根据百分比变化排序的顺序。接受两个值：`asc` 表示升序，`desc` 表示降序。
* `refresh_period`：定义交易对列表刷新的间隔（以秒为单位）。默认为 1800 秒（30 分钟）。
* `lookback_days`：回溯的天数。当选择 `lookback_days` 时，`lookback_timeframe` 默认为 1 天。
* `lookback_timeframe`：用于回溯周期的时间框架。
* `lookback_period`：回溯的周期数。

当 PercentChangePairList 在其他处理器之后使用时，它将基于那些处理器的输出进行操作。如果它是首位处理器，它将从所有具有指定质押货币的可用市场中选择交易对。

`PercentChangePairList` 使用交易所通过 ccxt 库提供的行情数据：
百分比变化计算为过去 24 小时内的价格变化。

??? Note "不支持的交易所"
    在某些交易所（如 HTX）上，常规 PercentChangePairList 无法工作，因为 API 原生不提供 24 小时价格百分比变化。可以通过使用蜡烛图数据来计算百分比变化来解决此问题。要大致模拟 24 小时百分比变化，可以使用以下配置。请注意，这些交易对列表每天只会刷新一次。
    ```json
    "pairlists": [
        {
            "method": "PercentChangePairList",
            "number_assets": 20,
            "min_value": 0,
            "refresh_period": 86400,
            "lookback_days": 1
        }
    ],
    ```

**从行情数据读取的示例配置**

```json
"pairlists": [
    {
        "method": "PercentChangePairList",
        "number_assets": 15,
        "min_value": -10,
        "max_value": 50
    }
],
```

在此配置中：

1. 根据过去 24 小时内最高百分比价格变化选择前 15 个交易对。
2. 只考虑百分比变化在 -10% 到 50% 之间的交易对。

**从蜡烛图数据读取的示例配置**

```json
"pairlists": [
    {
        "method": "PercentChangePairList",
        "number_assets": 15,
        "sort_key": "percentage",
        "min_value": 0,
        "refresh_period": 3600,
        "lookback_timeframe": "1h",
        "lookback_period": 72
    }
],
```

此示例通过使用 `lookback_timeframe` 指定蜡烛图大小和 `lookback_period` 指定蜡烛图数量，基于 3 天的 1 小时蜡烛图的滚动周期构建百分比变化交易对。

价格百分比变化使用以下公式计算，该公式表示当前蜡烛图收盘价与前一根蜡烛图收盘价之间的百分比差异（由指定的时间框架和回溯周期定义）：

$$ Percent Change = (\frac{Current Close - Previous Close}{Previous Close}) * 100 $$

!!! Warning "范围回溯和刷新周期"
    当与 `lookback_days` 和 `lookback_timeframe` 一起使用时，`refresh_period` 不能小于蜡烛图大小的秒数。否则会导致对交易所 API 的不必要请求。

!!! Warning "使用回溯范围时的性能影响"
    如果在首位与回溯功能结合使用，基于范围的百分比变化计算可能会消耗大量时间和资源，因为它会下载所有可交易对的蜡烛图数据。因此，强烈建议使用标准方法配合 `PercentChangePairList` 来缩小交易对列表范围，然后再进行进一步的百分比变化计算。

!!! Note "回测"
    `PercentChangePairList` 不支持回测模式。

#### ProducerPairList

使用 `ProducerPairList`，你可以复用 [Producer](producer-consumer.md) 的交易对列表，而无需在每个消费者上显式定义交易对列表。

此交易对列表需要 [消费者模式](producer-consumer.md) 才能工作。

该交易对列表将根据当前交易所配置对活跃交易对进行检查，以避免在无效市场上尝试交易。

你可以使用可选参数 `number_assets` 限制交易对列表的长度。使用 `"number_assets"=0` 或省略此键将复用所有对当前设置有效的生产者交易对。

```json
"pairlists": [
    {
        "method": "ProducerPairList",
        "number_assets": 5,
        "producer_name": "default",
    }
],
```

!!! Tip "组合交易对列表"
    此交易对列表可以与所有其他交易对列表和过滤器组合使用以进一步缩减列表，也可以作为已定义交易对之上的"附加"交易对列表。
    `ProducerPairList` 也可以按顺序多次使用，组合来自多个生产者的交易对。
    显然在这种复杂配置中，生产者可能不会为所有交易对提供数据，因此策略必须适应这种情况。

#### RemotePairList

它允许用户从远程服务器或 freqtrade 目录中本地存储的 JSON 文件获取交易对列表，从而实现交易对列表的动态更新和自定义。

RemotePairList 在配置的 pairlists 部分中定义。它使用以下配置选项：

```json
"pairlists": [
    {
        "method": "RemotePairList",
        "mode": "whitelist",
        "processing_mode": "filter",
        "pairlist_url": "https://example.com/pairlist",
        "number_assets": 10,
        "refresh_period": 1800,
        "keep_pairlist_on_failure": true,
        "read_timeout": 60,
        "bearer_token": "my-bearer-token",
        "save_to_file": "user_data/filename.json"
    }
]
```

可选的 `mode` 选项指定交易对列表应作为 `blacklist`（黑名单）还是 `whitelist`（白名单）使用。默认值为 "whitelist"。

可选的 `processing_mode` 选项决定如何处理获取的交易对列表。它可以有两个值："filter" 或 "append"。默认值为 "filter"。

可选的 `number_assets` 选项决定在白名单 `mode` 下返回多少个交易对。默认情况下，将返回所有交易对。在黑名单 `mode` 下，此选项将被忽略。

在 "filter" 模式下，获取的交易对列表用作过滤器。只有同时存在于原始列表和获取列表中的交易对才会被包含在最终列表中。其他交易对将被过滤掉。

在 "append" 模式下，获取的交易对列表将添加到原始列表中。两个列表中的所有交易对都将被包含在最终列表中，不进行任何过滤。

`pairlist_url` 选项指定远程服务器上交易对列表的 URL，或本地文件的路径（如果以 file:/// 开头）。这允许用户使用远程服务器或本地文件作为交易对列表的来源。

`save_to_file` 选项，当提供有效的文件名时，会将处理后的交易对列表以 JSON 格式保存到该文件。此选项是可选的，默认情况下不会将交易对列表保存到文件。

??? Example "多机器人共享交易对列表示例"

    `save_to_file` 可用于 Bot1 将交易对列表保存到文件：

    ```json
    "pairlists": [
        {
            "method": "RemotePairList",
            "mode": "whitelist",
            "pairlist_url": "https://example.com/pairlist",
            "number_assets": 10,
            "refresh_period": 1800,
            "keep_pairlist_on_failure": true,
            "read_timeout": 60,
            "save_to_file": "user_data/filename.json"
        }
    ]
    ```

    Bot2 或任何其他机器人可以使用以下配置加载此保存的交易对列表文件：

    ```json
    "pairlists": [
        {
            "method": "RemotePairList",
            "mode": "whitelist",
            "pairlist_url": "file:///user_data/filename.json",
            "number_assets": 10,
            "refresh_period": 10,
            "keep_pairlist_on_failure": true,
        }
    ]
    ```

用户负责提供一个返回以下结构 JSON 对象的服务器或本地文件：

```json
{
    "pairs": ["XRP/USDT", "ETH/USDT", "LTC/USDT"],
    "refresh_period": 1800
}
```

`pairs` 属性应包含机器人要使用的交易对字符串列表。`refresh_period` 属性是可选的，指定交易对列表在刷新前应缓存的秒数。

可选的 `keep_pairlist_on_failure` 指定当远程服务器不可达或返回错误时，是否应使用之前接收的交易对列表。默认值为 true。

可选的 `read_timeout` 指定等待远程源响应的最大时间（以秒为单位），默认值为 60。

可选的 `bearer_token` 将包含在请求的 Authorization 头中。

!!! Note
    在服务器错误的情况下，如果 `keep_pairlist_on_failure` 设置为 true，将保留最后接收的交易对列表；设置为 false 时，将返回空的交易对列表。

#### MarketCapPairList

`MarketCapPairList` 基于 CoinGecko 的市值排名对交易对进行排序/过滤。如果在白名单 `mode` 下使用，返回的交易对列表将按市值排名排序。

```json
"pairlists": [
    {
        "method": "MarketCapPairList",
        "number_assets": 20,
        "max_rank": 50,
        "refresh_period": 86400,
        "mode": "whitelist",
        "categories": ["layer-1"]
    }
]
```

`number_assets` 定义在白名单 `mode` 下交易对列表返回的最大交易对数量。在黑名单 `mode` 下，此设置将被忽略。

`max_rank` 将确定创建/过滤交易对列表时使用的最大排名。预计市值排名前 `max_rank` 的某些代币不会被包含在结果列表中，因为并非所有交易对在你偏好的市场/质押币/交易所组合中都有活跃的交易对。
虽然支持使用大于 250 的 `max_rank`，但不推荐这样做，因为这会导致对 CoinGecko 的多次 API 调用，可能导致速率限制问题。

`refresh_period` 设置定义市值排名数据刷新的间隔（以秒为单位）。默认为 86,400 秒（1 天）。交易对列表缓存（`refresh_period`）适用于生成交易对列表（在列表首位时）和过滤实例（不在列表首位时）。

`mode` 设置定义插件是过滤保留（白名单 `mode`）还是过滤排除（黑名单 `mode`）市值排名靠前的代币。默认情况下，插件将处于白名单模式。

`categories` 设置指定从哪些 [coingecko 分类](https://www.coingecko.com/en/categories) 中选择代币。默认为空列表 `[]`，表示不应用分类过滤。
如果选择了不正确的分类字符串，插件将打印 CoinGecko 的可用分类并失败。分类应为分类的 ID，例如，对于 `https://www.coingecko.com/en/categories/layer-1`，分类 ID 为 `layer-1`。你可以传入多个分类，如 `["layer-1", "meme-token"]` 来从多个分类中选择。

像 1000PEPE/USDT 或 KPEPE/USDT:USDT 这样的代币是在尽力而为的基础上检测的，使用前缀 `1000` 和 `K` 来识别它们。

!!! Warning "过多分类"
    每个添加的分类对应一次对 CoinGecko 的 API 调用。添加的分类越多，交易对列表生成时间越长，可能导致速率限制问题。

!!! Danger "CoinGecko 中的重复代币符号"
    CoinGecko 经常存在重复符号，即同一符号用于不同的代币。Freqtrade 将按原样使用该符号并尝试在交易所上搜索它。如果该符号存在，它将被使用。然而 Freqtrade 不会检查 CoinGecko 所指的是否是_预期的_符号。这有时会导致意外结果，特别是在低交易量代币或 meme 代币分类中。

#### CrossMarketPairList

根据交易对在对立市场上的可用性来生成或过滤交易对。

`pairs_exist_on` 设置定义交易对是否应存在于现货和合约两个市场上（`both_markets`），还是仅存在于指定的交易模式上（`current_market_only`）。默认情况下，插件使用 `both_markets` 设置，这意味着白名单中的交易对必须同时存在于现货和合约市场上。

#### AgeFilter

移除在交易所上市时间少于 `min_days_listed` 天（默认为 `10`）或超过 `max_days_listed` 天（默认为 `None` 表示无限）的交易对。

当交易对首次在交易所上市时，在最初几天的价格发现期间，它们可能会遭受巨大的价格下跌和波动。机器人常常会在交易对尚未完成价格下跌之前买入。

此过滤器允许 freqtrade 忽略上市时间少于 `min_days_listed` 天且在 `max_days_listed` 之前上市的交易对。

#### DelistFilter

移除将在从现在起最多 `max_days_from_now` 天内从交易所下架的交易对（默认为 `0`，即无论距离多远都移除所有将要下架的交易对）。目前此过滤器仅支持以下交易所：

!!! Note "可用交易所"
    DelistFilter 可在 Bybit 合约、Bitget 合约和 Binance 上使用，其中 Binance 合约在模拟和实盘模式下均可工作，而 Binance 现货仅限于实盘模式（出于技术原因）。

!!! Warning "回测"
    `DelistFilter` 不支持回测模式。

#### FullTradesFilter

当交易槽位已满时（当配置中 `max_open_trades` 未设置为 `-1` 时），将白名单缩小为仅包含正在交易中的交易对。

当交易槽位已满时，无需计算其余交易对的指标（除了信息性交易对），因为无法开启新交易。通过将白名单缩小为仅包含正在交易的交易对，可以提高计算速度并减少 CPU 使用率。当交易槽位空闲时（无论是交易关闭还是配置中的 `max_open_trades` 值增加），白名单将恢复正常状态。

当使用多个交易对列表过滤器时，建议将此过滤器放在主交易对列表正下方的第二个位置，这样当交易槽位已满时，机器人不必为其余过滤器下载数据。

!!! Warning "回测"
    `FullTradesFilter` 不支持回测模式。

#### OffsetFilter

按给定的 `offset` 值偏移传入的交易对列表。

例如，它可以与 `VolumeFilter` 结合使用，移除交易量排名前 X 的交易对。或者将较大的交易对列表拆分到两个机器人实例上。

示例：从交易对列表中移除前 10 个交易对，并取接下来的 20 个（取初始列表的第 10-30 项）：

```json
"pairlists": [
    // ...
    {
        "method": "OffsetFilter",
        "offset": 10,
        "number_assets": 20
    }
],
```

!!! Warning
    当 `OffsetFilter` 与 `VolumeFilter` 结合使用来在多个机器人之间拆分较大的交易对列表时，由于 `VolumeFilter` 的刷新间隔略有不同，无法保证交易对不会重叠。

!!! Note
    偏移量大于传入交易对列表的总长度将导致空的交易对列表。

#### PerformanceFilter

按过往交易表现排序交易对，顺序如下：

1. 正收益。
2. 尚无已关闭交易。
3. 负收益。

交易次数用作平局打破条件。

你可以使用 `minutes` 参数仅考虑过去 X 分钟内的表现（滚动窗口）。
不定义此参数（或设置为 0）将使用历史全部表现。

可选的 `min_profit`（作为比率 -> 设置 `0.01` 对应 1%）参数定义交易对被考虑所需的最低利润。
低于此水平的交易对将被过滤掉。
在没有 `minutes` 的情况下使用此参数是非常不推荐的，因为这可能导致空的交易对列表且无法恢复。

```json
"pairlists": [
    // ...
    {
        "method": "PerformanceFilter",
        "minutes": 1440,  // 滚动 24 小时
        "min_profit": 0.01  // 最低利润 1%
    }
],
```

由于此过滤器使用机器人的历史表现，它会有一些启动期 - 应仅在机器人数据库中有几百笔交易后使用。

!!! Warning "回测"
    `PerformanceFilter` 不支持回测模式。

#### PrecisionFilter

过滤掉因精度问题无法设置止损的低价值代币。

具体来说，如果价格精度舍入导致止损价格的 1% 或更大的变化，则交易对将被列入黑名单，即 `rounded(stop_price) <= rounded(stop_price * 0.99)`。其目的是避免价值非常接近其最低交易边界的代币，从而无法设置适当的止损。

!!! Tip "PrecisionFilter 对合约交易无意义"
    以上内容不适用于做空。对于做多，理论上交易会先被强制平仓。

!!! Warning "回测"
    `PrecisionFilter` 不支持使用多策略的回测模式。

#### PriceFilter

`PriceFilter` 允许按价格过滤交易对。目前支持以下价格过滤器：

* `min_price`
* `max_price`
* `max_value`
* `low_price_ratio`

`min_price` 设置移除价格低于指定价格的交易对。如果你希望避免交易非常低价的交易对，这很有用。
此选项默认禁用，仅在设置为 > 0 时生效。

`max_price` 设置移除价格高于指定价格的交易对。如果你只想交易低价交易对，这很有用。
此选项默认禁用，仅在设置为 > 0 时生效。

`max_value` 设置移除最小价值变化高于指定值的交易对。
当交易所有不平衡的限制时，这很有用。例如，如果步长 = 1（所以你只能买 1 个、2 个或 3 个，但不能买 1.1 个代币）- 而价格相当高（如 20\$），因为该代币自上次限制调整以来大幅上涨。
因此，你只能买 20\$ 或 40\$ - 但不能买 25\$。
在从接收货币中扣除手续费的交易所（如 Binance）上 - 这可能导致高价值代币/金额因金额略低于限制而无法卖出。

`low_price_ratio` 设置移除 1 个价格单位（pip）的涨幅高于 `low_price_ratio` 比率的交易对。
此选项默认禁用，仅在设置为 > 0 时生效。

`PriceFilter` 的 `min_price`、`max_price` 或 `low_price_ratio` 设置中至少需要应用一个。

计算示例：

SHITCOIN/BTC 的最小价格精度为 8 位小数。如果其价格为 0.00000011 - 高一个价格步长为 0.00000012，比前一个价格值高约 9%。你可以通过将 `low_price_ratio` 设置为 0.09（9%）或将 `min_price` 设置为 0.00000011 来相应地过滤掉此交易对。

!!! Warning "低价交易对"
    具有高"1 pip 波动"的低价交易对是危险的，因为它们通常流动性不足，而且可能无法设置所需的止损，这常常导致高额损失，因为价格需要四舍五入到下一个可交易价格 - 所以你可能最终得到 -9% 的止损而不是 -5% 的止损，仅仅是因为价格舍入。

#### ShuffleFilter

随机打乱交易对列表中的交易对顺序。当你希望所有交易对以相同优先级处理时，它可以防止机器人比其他交易对更频繁地交易某些交易对。

默认情况下，ShuffleFilter 每根蜡烛图打乱一次。
要在每次迭代时打乱，请将 `"shuffle_frequency"` 设置为 `"iteration"` 而不是默认的 `"candle"`。

``` json
    {
        "method": "ShuffleFilter",
        "shuffle_frequency": "candle",
        "seed": 42
    }

```

!!! Tip
    你可以为此交易对列表设置 `seed` 值以获得可重复的结果，这对于重复的回测会话很有用。如果未设置 `seed`，交易对将以不可重复的随机顺序打乱。ShuffleFilter 会自动检测运行模式，仅在回测模式下应用 `seed` - 如果设置了 `seed` 值的话。

#### SpreadFilter

移除买卖价差高于指定比率 `max_spread_ratio`（默认为 `0.005`）的交易对。

示例：

如果 `DOGE/BTC` 的最高买价为 0.00000026，最低卖价为 0.00000027，则比率计算为：`1 - bid/ask ~= 0.037`，这大于 `> 0.005`，因此该交易对将被过滤掉。

#### RangeStabilityFilter

移除在 `lookback_days` 天内最低价和最高价之间的差异低于 `min_rate_of_change` 或高于 `max_rate_of_change` 的交易对。由于这是一个需要额外数据的过滤器，结果会缓存 `refresh_period` 时间。

在下面的示例中：
如果过去 10 天的交易范围 <1% 或 >99%，则从白名单中移除该交易对。

```json
"pairlists": [
    {
        "method": "RangeStabilityFilter",
        "lookback_days": 10,
        "min_rate_of_change": 0.01,
        "max_rate_of_change": 0.99,
        "refresh_period": 86400
    }
]
```

添加 `"sort_direction": "asc"` 或 `"sort_direction": "desc"` 可启用此交易对列表的排序功能。

!!! Tip
    此过滤器可用于自动移除稳定币交易对，这些交易对的交易范围非常低，因此极难获利。
    此外，它还可用于自动移除在给定时间内波动极高/极低的交易对。

#### VolatilityFilter

波动率是交易对历史变化的程度，通过对数日收益率的标准差来衡量。收益率被假定为正态分布，尽管实际分布可能不同。在正态分布中，68% 的观测值落在一个标准差内，95% 的观测值落在两个标准差内。假设波动率为 0.05 意味着 30 天中有 20 天的预期收益率预计低于 5%（一个标准差）。波动率是预期收益偏差的正比率，可以大于 1.00。请参阅维基百科对 [`volatility`](https://en.wikipedia.org/wiki/Volatility_(finance)) 的定义。

如果 `lookback_days` 天内的平均波动率低于 `min_volatility` 或高于 `max_volatility`，此过滤器将移除相应交易对。由于这是一个需要额外数据的过滤器，结果会缓存 `refresh_period` 时间。

此过滤器可用于将交易对缩小到特定波动率范围或避免波动性极高的交易对。

在下面的示例中：
如果过去 10 天的波动率不在 0.05-0.50 的范围内，则从白名单中移除该交易对。过滤器每 24 小时应用一次。

```json
"pairlists": [
    {
        "method": "VolatilityFilter",
        "lookback_days": 10,
        "min_volatility": 0.05,
        "max_volatility": 0.50,
        "refresh_period": 86400
    }
]
```

添加 `"sort_direction": "asc"` 或 `"sort_direction": "desc"` 可启用此交易对列表的排序模式。

### 交易对列表处理器完整示例

以下示例将 `BNB/BTC` 列入黑名单，使用 `VolumePairList` 选取 `20` 个资产并按 `quoteVolume` 排序，然后使用 [`DelistFilter`](#delistfilter) 过滤即将下架的交易对，使用 [`AgeFilter`](#agefilter) 移除上市不到 10 天的交易对。之后应用 [`PrecisionFilter`](#precisionfilter) 和 [`PriceFilter`](#pricefilter)，过滤掉 1 个价格单位 > 1% 的所有资产。然后应用 [`SpreadFilter`](#spreadfilter) 和 [`VolatilityFilter`](#volatilityfilter)，最后使用预定义的随机种子打乱交易对。

```json
"exchange": {
    "pair_whitelist": [],
    "pair_blacklist": ["BNB/BTC"]
},
"pairlists": [
    {
        "method": "VolumePairList",
        "number_assets": 20,
        "sort_key": "quoteVolume"
    },
    {
        "method": "DelistFilter",
        "max_days_from_now": 0,
    },
    {"method": "AgeFilter", "min_days_listed": 10},
    {"method": "PrecisionFilter"},
    {"method": "PriceFilter", "low_price_ratio": 0.01},
    {"method": "SpreadFilter", "max_spread_ratio": 0.005},
    {
        "method": "RangeStabilityFilter",
        "lookback_days": 10,
        "min_rate_of_change": 0.01,
        "refresh_period": 86400
    },
    {
        "method": "VolatilityFilter",
        "lookback_days": 10,
        "min_volatility": 0.05,
        "max_volatility": 0.50,
        "refresh_period": 86400
    },
    {"method": "ShuffleFilter", "seed": 42}
],
```
