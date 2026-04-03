# 交易所特定说明

本页汇总了特定于交易所的常见问题和信息，这些内容很可能不适用于其他交易所。

## 支持的交易所功能快速概览

--8<-- "includes/exchange-features.md"

## 交易所配置

Freqtrade 基于 [CCXT 库](https://github.com/ccxt/ccxt)，该库支持超过100个加密货币交易市场和交易 API。完整的最新列表可以在 [CCXT 仓库主页](https://github.com/ccxt/ccxt/tree/master/python)找到。
但是，开发团队仅在少数交易所上进行了测试。
当前列表可以在本文档的"首页"部分找到。

欢迎测试其他交易所并提交反馈或 PR 来改进机器人或确认运行良好的交易所。

某些交易所需要特殊配置，具体内容见下文。

### 交易所配置示例

"binance"的交易所配置如下所示：

```json
"exchange": {
    "name": "binance",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "ccxt_config": {},
    "ccxt_async_config": {},
    // ...
```

### 设置速率限制

通常，CCXT 设置的速率限制是可靠且运行良好的。
如果遇到与速率限制相关的问题（通常是日志中的 DDOS 异常），可以轻松地将 rateLimit 设置更改为其他值。

```json
"exchange": {
    "name": "kraken",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "ccxt_config": {"enableRateLimit": true},
    "ccxt_async_config": {
        "enableRateLimit": true,
        "rateLimit": 3100
    },
```

此配置启用了 kraken 交易所以及速率限制以避免被交易所封禁。
`"rateLimit": 3100` 定义了每次调用之间3.1秒的等待时间。也可以通过将 `"enableRateLimit"` 设置为 false 来完全禁用。

!!! Note
    速率限制的最佳设置取决于交易所和白名单的大小，因此理想的参数会因许多其他设置而异。
    我们尽可能为每个交易所提供合理的默认值，如果您遇到封禁，请确保 `"enableRateLimit"` 已启用并逐步增加 `"rateLimit"` 参数。

## Binance

!!! Warning "服务器位置和地理 IP 限制"
    请注意，Binance 对服务器所在国家的 API 访问有限制。当前被限制的国家（不完全列表）包括加拿大、马来西亚、荷兰和美国。请访问 [binance 条款 > b. 资格](https://www.binance.com/en/terms) 查看最新列表。

Binance 支持 [time_in_force](configuration.md#understand-order_time_in_force)。

!!! Tip "交易所止损"
    Binance 支持 `stoploss_on_exchange` 并使用 `stop-loss-limit` 订单。它提供了很大的优势，因此我们建议通过启用交易所止损来利用它。
    在期货上，Binance 支持 `stop-limit` 和 `stop-market` 订单。您可以在 `order_types.stoploss` 配置设置中使用 `"limit"` 或 `"market"` 来决定使用哪种类型。

### Binance 黑名单建议

对于 Binance，建议将 `"BNB/<STAKE>"` 添加到黑名单中以避免问题，除非您愿意在账户上维持足够的额外 `BNB`，或者您愿意禁用使用 `BNB` 支付手续费。
Binance 账户可能使用 `BNB` 支付手续费，如果交易恰好在 `BNB` 上，后续交易可能会消耗此仓位，导致初始 BNB 交易因预期数量不足而无法卖出。

如果没有足够的 `BNB` 来支付交易费用，则费用将不会由 `BNB` 支付，也不会享受费用减免。Freqtrade 永远不会购买 BNB 来支付费用。BNB 需要手动购买和监控。

### Binance 站点

Binance 已拆分为2个站点，用户必须为其交易所使用正确的 ccxt exchange ID，否则 API 密钥将不被识别。

* [binance.com](https://www.binance.com/) - 国际用户。使用 exchange id：`binance`。
* [binance.us](https://www.binance.us/) - 美国用户。使用 exchange id：`binanceus`。

### Binance RSA 密钥

Freqtrade 支持 binance RSA API 密钥。

我们建议将其用作环境变量。

``` bash
export FREQTRADE__EXCHANGE__SECRET="$(cat ./rsa_binance.private)"
```

但是也可以通过配置文件进行配置。由于 json 不支持多行字符串，您需要将所有换行符替换为 `\n` 以获得有效的 json 文件。

``` json
// ...
 "key": "<someapikey>",
 "secret": "-----BEGIN PRIVATE KEY-----\nMIIEvQIBABACAFQA<...>s8KX8=\n-----END PRIVATE KEY-----"
// ...
```

### Binance 期货

Binance 有特定的（不幸的是相当复杂的）[期货交易量化规则](https://www.binance.com/en/support/faq/4f462ebe6ff445d4a170be7d9e897272)需要遵守，其中禁止对太多订单使用过低的下注金额（等等）。
违反这些规则将导致交易限制。

在 Binance 期货市场交易时，必须使用订单簿，因为期货没有价格行情数据。

``` jsonc
  "entry_pricing": {
      "use_order_book": true,
      "order_book_top": 1,
      "check_depth_of_market": {
          "enabled": false,
          "bids_to_ask_delta": 1
      }
  },
  "exit_pricing": {
      "use_order_book": true,
      "order_book_top": 1
  },
```

#### Binance 逐仓期货设置

用户还需要将期货设置中的"Position Mode"设置为"One-way Mode"，并将"Asset Mode"设置为"Single-Asset Mode"。
这些设置将在启动时进行检查，如果设置不正确，freqtrade 将显示错误。

![Binance futures settings](assets/binance_futures_settings.png)

Freqtrade 不会尝试更改这些设置。

#### Binance BNFCR 期货

BNFCR 模式是 Binance 上一种特殊的期货模式，用于解决欧洲的监管问题。
要使用 BNFCR 期货，您需要以下配置组合：

``` jsonc
{
    // ...
    "trading_mode": "futures",
    "margin_mode": "cross",
    "proxy_coin": "BNFCR",
    "stake_currency": "USDT" // or "USDC"
    // ...
}
```

`stake_currency` 设置定义了机器人将在其中操作的市场。这个选择实际上是任意的。

在交易所上，您需要使用"Multi-asset Mode"——并将"Position Mode"设置为"One-way Mode"。
Freqtrade 将在启动时检查这些设置，但不会尝试更改它们。

## Bingx

BingX 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）、"IOC"（立即成交或取消）和"PO"（仅挂单）设置。

!!! Tip "交易所止损"
    Bingx 支持 `stoploss_on_exchange`，可以使用止损限价单和止损市价单。它提供了很大的优势，因此我们建议通过启用交易所止损来利用它。

## Kraken

Kraken 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）、"IOC"（立即成交或取消）和"PO"（仅挂单）设置。

!!! Tip "交易所止损"
    Kraken 支持 `stoploss_on_exchange`，可以使用止损市价单和止损限价单。它提供了很大的优势，因此我们建议利用它。
    您可以在 `order_types.stoploss` 配置设置中使用 `"limit"` 或 `"market"` 来决定使用哪种类型。

### 历史 Kraken 数据

Kraken API 仅提供720根历史K线，这对于 Freqtrade 的模拟运行和实盘交易模式来说足够了，但对回测来说是一个问题。
要下载 Kraken 交易所的数据，必须使用 `--dl-trades`，否则机器人将反复下载相同的720根K线，您将没有足够的回测数据。

为了加速下载，您可以下载 Kraken 提供的[交易 zip 文件](https://support.kraken.com/hc/en-us/articles/360047543791-Downloadable-historical-market-data-time-and-sales-)。
这些文件通常每季度更新一次。Freqtrade 期望这些文件放置在 `user_data/data/kraken/trades_csv` 中。

如果使用增量文件，以下结构可能很有用，将"完整"历史数据放在一个目录中，增量文件放在不同的目录中。
此模式的假设是数据已下载并解压缩，保持文件名不变。
重复内容将被忽略（基于时间戳）——但假设数据中没有间隔。

这意味着，如果您的"完整"历史记录在2022年第四季度结束——那么增量更新 Q1 2023 和 Q2 2023 都应该可用。
如果没有这些，将导致数据不完整，从而在使用数据时产生无效结果。

```
└── trades_csv
    ├── Kraken_full_history
    │   ├── BCHEUR.csv
    │   └── XBTEUR.csv
    ├── Kraken_Trading_History_Q1_2023
    │   ├── BCHEUR.csv
    │   └── XBTEUR.csv
    └── Kraken_Trading_History_Q2_2023
        ├── BCHEUR.csv
        └── XBTEUR.csv
```

您可以将这些文件转换为 freqtrade 文件：

``` bash
freqtrade convert-trade-data --exchange kraken --format-from kraken_csv --format-to feather
# Convert trade data to different ohlcv timeframes
freqtrade trades-to-ohlcv -p BTC/EUR BCH/EUR --exchange kraken -t 1m 5m 15m 1h
```

转换后的数据也使下载数据成为可能，并将在最后加载的交易之后开始下载。

``` bash
freqtrade download-data --exchange kraken --dl-trades -p BTC/EUR BCH/EUR
```

!!! Warning "从 kraken 下载数据"
    从 kraken 下载数据将需要比任何其他交易所显著更多的内存（RAM），因为交易数据需要在您的机器上转换为K线。
    这也将需要很长时间，因为 freqtrade 需要下载该交易对/时间范围组合在交易所上发生的每一笔交易，因此请耐心等待。

!!! Warning "rateLimit 调优"
    请注意，rateLimit 配置条目保存的是请求之间的延迟（以毫秒为单位），而不是每秒请求数。
    因此，为了缓解 Kraken API 的"Rate limit exceeded"异常，此配置应该增加，而不是减少。

## Kraken 期货

Kraken 期货使用 exchange id `krakenfutures`，支持逐仓期货模式。

```jsonc
"exchange": {
    "name": "krakenfutures",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret"
},
"trading_mode": "futures",
"margin_mode": "isolated",
"stake_currency": "USD"
```

!!! Tip "交易所止损"
    Kraken 期货支持 `stoploss_on_exchange`，支持 `limit` 和 `market` 止损订单。
    使用 `order_types.stoploss_price_type` 选择触发价格来源（`mark`、`last` 或 `index`）。

!!! Note "抵押品"
    Kraken 期货以 USD 结算。请使用 USD 作为您的计价货币。

!!! Note "Flex（多抵押品）账户"
    Kraken 期货 flex 账户允许以多种货币作为抵押品，而交易仍以 USD 结算。
    Freqtrade 从 Kraken 保证金字段中推导 `USD` 余额，因此请保持 `stake_currency` 设置为 `USD`。

## Kucoin

Kucoin 要求每个 API 密钥都有一个密码短语，因此您需要将此密钥添加到配置中，使您的交易所部分如下所示：

```json
"exchange": {
    "name": "kucoin",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "password": "your_exchange_api_key_password",
    // ...
}
```

Kucoin 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）、"FOK"（全部成交或取消）和"IOC"（立即成交或取消）设置。

!!! Tip "交易所止损"
    Kucoin 支持 `stoploss_on_exchange`，可以使用止损市价单和止损限价单。它提供了很大的优势，因此我们建议利用它。
    您可以在 `order_types.stoploss` 配置设置中使用 `"limit"` 或 `"market"` 来决定使用哪种类型的止损。

### Kucoin 黑名单

对于 Kucoin，建议将 `"KCS/<STAKE>"` 添加到黑名单中以避免问题，除非您愿意在账户上维持足够的额外 `KCS`，或者您愿意禁用使用 `KCS` 支付手续费。
Kucoin 账户可能使用 `KCS` 支付手续费，如果交易恰好在 `KCS` 上，后续交易可能会消耗此仓位，导致初始 `KCS` 交易因预期数量不足而无法卖出。

## HTX

!!! Tip "交易所止损"
    HTX 支持 `stoploss_on_exchange` 并使用 `stop-limit` 订单。它提供了很大的优势，因此我们建议通过启用交易所止损来利用它。

## OKX

OKX 要求每个 API 密钥都有一个密码短语，因此您需要将此密钥添加到配置中，使您的交易所部分如下所示：

```json
"exchange": {
    "name": "okx",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "password": "your_exchange_api_key_password",
    // ...
}
```

如果您在 my.okx.com（OKX EAA）上注册——您需要使用 `"myokx"` 作为交易所名称。
使用错误的交易所将导致错误"OKX Error 50119: API key doesn't exist"——因为这两个是独立的实体。

!!! Warning
    OKX 每次 API 调用仅提供100根K线。因此，在回测模式下策略可用的数据量相当有限。

!!! Warning "期货"
    OKX 期货有"仓位模式"的概念——可以是"买入/卖出"或多头/空头（对冲模式）。
    Freqtrade 支持两种模式（我们建议使用买入/卖出模式）——但不支持在交易过程中更改模式，这会导致异常和下单失败。
    OKX 也仅提供大约过去3个月的 MARK K线。因此，在该日期之前回测期货将导致轻微偏差，因为没有此数据就无法正确计算资金费率。

## Gate.io

!!! Tip "交易所止损"
    Gate.io 支持 `stoploss_on_exchange` 并使用 `stop-loss-limit` 订单。它提供了很大的优势，因此我们建议通过启用交易所止损来利用它。

Gate.io 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）和"IOC"（立即成交或取消）设置。

Gate.io 允许使用 `POINT` 支付手续费。由于这不是可交易的货币（没有常规市场可用），自动费用计算将失败（并默认为0的费用）。
配置参数 `exchange.unknown_fee_rate` 可用于指定 Point 与计价货币之间的汇率。显然，更改计价货币也需要更改此值。

Gate API 密钥除了您想交易的市场类型外，还需要以下权限：

* "Spot Trade" 或 "Perpetual Futures"（读写权限）（两者都选择，或选择与您想交易的市场匹配的那个）
* "Wallet"（只读）
* "Account"（只读）

没有这些权限，机器人将无法正确启动并显示"permission missing"等错误。

## Bybit

!!! Tip "交易所止损"
    Bybit（仅限期货）支持 `stoploss_on_exchange` 并使用 `stop-loss-limit` 订单。它提供了很大的优势，因此我们建议通过启用交易所止损来利用它。
    在期货上，Bybit 支持 `stop-limit` 和 `stop-market` 订单。您可以在 `order_types.stoploss` 配置设置中使用 `"limit"` 或 `"market"` 来决定使用哪种类型。

Bybit 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）、"FOK"（全部成交或取消）、"IOC"（立即成交或取消）和"PO"（仅挂单）设置。

!!! Warning "统一账户"
    Freqtrade 假设账户是专门用于机器人的。
    因此，我们建议每个机器人使用一个子账户。这在使用统一账户时尤其重要。
    其他配置（一个账户上运行多个机器人、在机器人账户上进行手动非机器人交易）不受支持，可能导致意外行为。

### Bybit 期货

Bybit 支持逐仓期货模式的期货交易。

在启动时，freqtrade 将为整个（子）账户设置仓位模式为"One-way Mode"。这避免了反复进行此调用（减慢机器人操作），但意味着手动更改此设置可能导致异常和错误。

由于 bybit 不提供资金费率历史数据，实盘交易也使用模拟运行的计算方式。

实盘期货交易的 API 密钥必须具有以下权限：

* 读写权限
* Contract - Orders
* Contract - Positions

我们强烈建议将所有 API 密钥限制为您将使用的 IP。

## Bitmart

Bitmart 要求 API 密钥备注（您给 API 密钥起的名称）与交易所密钥和密码一起使用。
因此也需要传递 UID。

```json
"exchange": {
    "name": "bitmart",
    "uid": "your_bitmart_api_key_memo",
    "secret": "your_exchange_secret",
    "password": "your_exchange_api_key_password",
    // ...
}
```

!!! Warning "必要的验证"
    Bitmart 需要 Lvl2 验证才能通过 API 在现货市场上成功交易——即使通过 UI 进行交易只需要 Lvl1 验证就可以正常工作。

## Bitget

Bitget 要求每个 API 密钥都有一个密码短语，因此您需要将此密钥添加到配置中，使您的交易所部分如下所示：

```json
"exchange": {
    "name": "bitget",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "password": "your_exchange_api_key_password",
    // ...
}
```

Bitget 支持 [time_in_force](configuration.md#understand-order_time_in_force)，包括"GTC"（有效直到取消）、"FOK"（全部成交或取消）、"IOC"（立即成交或取消）和"PO"（仅挂单）设置。

!!! Tip "交易所止损"
    Bitget 支持 `stoploss_on_exchange`，可以使用止损市价单和止损限价单。它提供了很大的优势，因此我们建议利用它。
    您可以在 `order_types.stoploss` 配置设置中使用 `"limit"` 或 `"market"` 来决定使用哪种类型的止损。

### Bitget 期货

Bitget 支持逐仓期货模式的期货交易。

在启动时，freqtrade 将为整个（子）账户设置仓位模式为"One-way Mode"。这避免了反复进行此调用（减慢机器人操作），但意味着手动更改此设置可能导致异常和错误。

## Hyperliquid

!!! Tip "交易所止损"
    Hyperliquid 支持 `stoploss_on_exchange` 并使用 `stop-loss-limit` 订单。它提供了很大的优势，因此我们建议利用它。

!!! Warning "统一账户"
    Hyperliquid 统一账户受支持——但这依赖于 freqtrade "拥有"该账户的假设，并且是唯一在其上进行交易的（在这种情况下，扩展到现货和期货）。
    因此，我们建议尽可能使用子账户，并在机器人运行时避免在同一账户上进行手动交易。
    Freqtrade 将在启动时尝试检测账户类型——不支持在交易过程中更改账户类型，这可能导致异常和错误。

Hyperliquid 是一个去中心化交易所（DEX）。去中心化交易所的工作方式与普通交易所有些不同。私有 API 调用不是使用 API 密钥进行身份验证，而是需要使用您钱包的私钥进行签名（我们建议为此使用 API 钱包，可以在 Hyperliquid 上或在您选择的钱包中生成）。
需要如下配置：

```json
"exchange": {
    "name": "hyperliquid",
    "walletAddress": "your_eth_wallet_address",  // This should NOT be your API Wallet Address!
    "privateKey": "your_api_private_key",
    // ...
}
```

* walletAddress 十六进制格式：`0x<40 hex characters>` - 可以从您的钱包中轻松复制——应该是您的主钱包地址，而不是 API 钱包地址。
* privateKey 十六进制格式：`0x<64 hex characters>` - 使用 API 钱包在创建时显示的密钥。

Hyperliquid 在 Arbitrum One 链上处理充值和提现，这是建立在 Ethereum 之上的 Layer 2 扩展解决方案。Hyperliquid 使用 USDC 作为报价/抵押品。在 Hyperliquid 上充值 USDC 的过程需要几个步骤，详见[如何开始交易](https://hyperliquid.gitbook.io/hyperliquid-docs/onboarding/how-to-start-trading)了解所需步骤的详细信息。

!!! Note "Hyperliquid 一般使用说明"
    Hyperliquid 不支持市价单，但 ccxt 会通过下限价单并设置5%的最大滑点来模拟市价单。
    不幸的是，hyperliquid 仅提供5000根历史K线，因此回测要么需要历史性地构建K线（通过等待并随时间逐步下载数据）——要么将限于最近的5000根K线。

!!! Info "一些通用最佳实践（非详尽列表）"
    * 注意供应链攻击，如 pip 包投毒等。每当您使用私钥时，请确保您的环境是安全的。
    * 不要使用您实际钱包的私钥进行交易。使用 Hyperliquid [API 生成器](https://app.hyperliquid.xyz/API) 创建单独的 API 钱包。
    * 不要将您实际钱包的私钥存储在用于 freqtrade 的服务器上。使用 API 钱包私钥代替。此密钥不允许提现，仅允许交易。
    * 始终保持您的助记词和私钥的私密性。
    * 不要使用与初始化硬件钱包时备份的相同助记词，使用相同的助记词基本上会删除硬件钱包的安全性。
    * 创建一个不同的软件钱包，仅将您想要交易的资金转移到该钱包，并使用该钱包在 Hyperliquid 上进行交易。
    * 如果您有不想用于交易的资金（例如获利后），请将其转回您的硬件钱包。

### Hyperliquid 金库/子账户

Hyperliquid 允许您创建金库或子账户。
要将它们与 Freqtrade 一起使用，您需要使用以下配置模式：

``` json
"exchange": {
    "name": "hyperliquid",
    "walletAddress": "your_master_wallet_address", // Your master wallet address (not the API wallet address and not the vault/subaccount address).
    "privateKey": "your_api_private_key", // API wallet private key (see https://app.hyperliquid.xyz/API). You'll only need the private key.
    "ccxt_config": {
        "options": {
            "vaultAddress": "your_vault_address", // Optional, only if you want to use a vault ...
            "subAccountAddress": "your_subaccount_address" // OR optional, only if you want to use a subaccount
        }
    },
    // ...
}
```

您的余额和交易现在将从金库/子账户中使用——不再从主账户中使用。

!!! Note
    您只能使用金库或子账户——不能同时使用两者。


### 历史 Hyperliquid 数据

Hyperliquid API 除了获取当前数据的单次调用外不提供历史数据，因此下载数据是不可能的，因为下载的数据不会构成正确的历史数据。

### HIP-3 DEX

Hyperliquid 支持 HIP-3 去中心化交易所（DEX），这些是建立在 Hyperliquid 基础设施之上的独立交易所。
这些 DEX 的运作方式与主要的 Hyperliquid 交易所类似，但由社区创建和管理。

要使用 Freqtrade 在 HIP-3 DEX 上交易，您需要使用 `hip3_dexes` 参数将它们添加到您的配置中：

```json
"exchange": {
    "name": "hyperliquid",
    "walletAddress": "your_master_wallet_address",
    "privateKey": "your_api_private_key",
    "hip3_dexes": ["dex_name_1", "dex_name_2"]
}
```

将 `"dex_name_1"` 和 `"dex_name_2"` 替换为您想要交易的 HIP-3 DEX 的实际名称（例如 `vntl` 和 `xyz`）。

!!! Warning "性能和速率限制影响"
    您添加的每个 HIP-3 DEX 都会显著影响机器人性能和速率限制。

    * **额外的 API 调用**：对于配置的每个 HIP-3 DEX，Freqtrade 需要进行额外的 API 调用。
    * **速率限制压力**：额外的 API 调用会增加 Hyperliquid 严格速率限制的压力。使用多个 DEX，您可能更快达到速率限制，或者更准确地说，由于强制延迟而减慢机器人操作。

    请仅添加您正在积极交易的 HIP-3 DEX。监控日志中的速率限制警告或操作减慢的迹象，并相应调整您的配置。
    不同的 HIP-3 DEX 也可能使用不同的报价货币——因此请确保仅添加与您的计价货币兼容的 DEX，以避免不必要的延迟。

!!! Note
    HIP-3 DEX 与您的主 Hyperliquid 账户共享相同的钱包和可用抵押品。在不同 DEX 上的交易将影响您的整体账户余额和保证金。

    HIP-3 交易对的命名与非 HIP-3 交易对略有不同。请使用 `list-pairs` 子命令获取指定 DEX 的所有交易对的正确命名。

## Bitvavo

如果您的账户需要使用 operatorId，您可以在配置文件中按如下方式设置：

``` json
"exchange": {
        "name": "bitvavo",
        "key": "",
        "secret": "",
        "ccxt_config": {
            "options": {
                "operatorId": "123567"
            }
        },
   }
```

Bitvavo 期望 `operatorId` 是一个整数。

## 所有交易所

如果您遇到持续的 Nonce 错误（如 `InvalidNonce`），最好重新生成 API 密钥。重置 Nonce 很困难，通常重新生成 API 密钥更容易。

## 其他交易所的随机说明

* The Ocean（exchange id：`theocean`）交易所使用 Web3 功能，需要安装 `web3` Python 包：

```shell
pip3 install web3
```

### 获取最新价格/不完整K线

大多数交易所通过其 OHLCV/klines API 接口返回当前不完整的K线。
默认情况下，Freqtrade 假设从交易所获取了不完整的K线，并移除最后一根K线（假设它是不完整的K线）。

您的交易所是否返回不完整的K线可以使用贡献者文档中的[辅助脚本](developer.md#incomplete-candles)进行检查。

由于重绘的风险，Freqtrade 不允许您使用此不完整的K线。

但是，如果是基于策略需要最新价格的需求——那么可以从策略内部使用[数据提供者](strategy-customization.md#possible-options-for-dataprovider)来获取此需求。

### 高级 Freqtrade 交易所配置

可以使用 `_ft_has_params` 设置配置高级选项，这将覆盖默认值和交易所特定的行为。

可用选项列在交易所类中的 `_ft_has_default` 中。

例如，要使用 Kraken 测试订单类型 `FOK`，并将K线限制修改为200（这样每次 API 调用只获取200根K线）：

```json
"exchange": {
    "name": "kraken",
    "_ft_has_params": {
        "order_time_in_force": ["GTC", "FOK"],
        "ohlcv_candle_limit": 200
        }
    //...
}
```

!!! Warning
    在修改这些设置之前，请确保完全了解其影响。
    使用 `_ft_has_params` 覆盖可能导致意外行为，甚至可能破坏您的机器人。
    对于由 `_ft_has_params` 中自定义设置引起的问题，我们将无法提供支持。
