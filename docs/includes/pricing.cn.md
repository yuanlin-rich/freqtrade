## 订单使用的价格

常规订单的价格可以通过参数结构 `entry_pricing`（用于交易入场）和 `exit_pricing`（用于交易退出）来控制。
价格始终在下单之前获取，可以通过查询交易所行情数据或使用订单簿数据来获取。

!!! Note
    Freqtrade 使用的订单簿数据是通过 ccxt 的 `fetch_order_book()` 函数从交易所获取的数据，即通常是 L2 聚合订单簿的数据，而行情数据是 ccxt 的 `fetch_ticker()`/`fetch_tickers()` 函数返回的结构。有关更多详细信息，请参阅 ccxt 库的[文档](https://github.com/ccxt/ccxt/wiki/Manual#market-data)。

!!! Warning "使用市价单"
    使用市价单时，请阅读[市价单定价](#market-order-pricing)部分。

### 入场价格

#### 入场价格方向

配置项 `entry_pricing.price_side` 定义了机器人在买入时查看订单簿的哪一侧。

以下展示了一个订单簿。

``` explanation
...
103
102
101  # ask（卖价）
-------------当前价差
99   # bid（买价）
98
97
...
```

如果 `entry_pricing.price_side` 设置为 `"bid"`，则机器人将使用 99 作为入场价格。
相应地，如果 `entry_pricing.price_side` 设置为 `"ask"`，则机器人将使用 101 作为入场价格。

根据订单方向（_做多_/_做空_），这将导致不同的结果。因此我们建议使用 `"same"` 或 `"other"` 来代替此配置。
这将产生以下定价矩阵：

| 方向 | 订单 | 设置 | 价格 | 是否跨越价差 |
|------ |--------|-----|-----|-----|
| long  | buy  | ask   | 101 | 是 |
| long  | buy  | bid   | 99  | 否  |
| long  | buy  | same  | 99  | 否  |
| long  | buy  | other | 101 | 是 |
| short | sell | ask   | 101 | 否  |
| short | sell | bid   | 99  | 是 |
| short | sell | same  | 101 | 否  |
| short | sell | other | 99  | 是 |

使用订单簿的另一侧通常可以保证更快的订单成交，但机器人也可能支付比必要更多的费用。
即使使用限价买单，也很可能适用吃单（taker）费用而非挂单（maker）费用。
此外，价差"另一侧"的价格高于订单簿中"买价"侧的价格，因此订单的行为类似于市价单（但有最高价格限制）。

#### 启用订单簿时的入场价格

当启用订单簿进行入场交易时（`entry_pricing.use_order_book=True`），Freqtrade 从订单簿中获取 `entry_pricing.order_book_top` 个条目，并使用配置侧（`entry_pricing.price_side`）订单簿中指定为 `entry_pricing.order_book_top` 的条目。1 表示订单簿中最顶部的条目，2 表示第二个条目，以此类推。

#### 未启用订单簿时的入场价格

以下部分使用 `side` 作为已配置的 `entry_pricing.price_side`（默认为 `"same"`）。

当不使用订单簿时（`entry_pricing.use_order_book=False`），如果行情数据中的最佳 `side` 价格低于行情数据中的最后成交价（`last`），Freqtrade 将使用该 `side` 价格。否则（当 `side` 价格高于 `last` 价格时），它会根据 `entry_pricing.price_last_balance` 在 `side` 和 `last` 价格之间计算一个价格。

`entry_pricing.price_last_balance` 配置参数控制此行为。值为 `0.0` 将使用 `side` 价格，而 `1.0` 将使用 `last` 价格，介于两者之间的值将在 ask 和 last 价格之间插值。

#### 检查市场深度

当启用检查市场深度时（`entry_pricing.check_depth_of_market.enabled=True`），入场信号将根据每侧订单簿的深度（所有金额的总和）进行过滤。

订单簿 `bid`（买入）侧的深度除以订单簿 `ask`（卖出）侧的深度，得到的差值与 `entry_pricing.check_depth_of_market.bids_to_ask_delta` 参数的值进行比较。只有当订单簿差值大于或等于配置的差值时，才会执行入场订单。

!!! Note
    差值低于 1 意味着 `ask`（卖出）订单簿侧的深度大于 `bid`（买入）订单簿侧的深度，而差值大于 1 则相反（买入侧的深度大于卖出侧的深度）。

### 退出价格

#### 退出价格方向

配置项 `exit_pricing.price_side` 定义了机器人在退出交易时查看价差的哪一侧。

以下展示了一个订单簿：

``` explanation
...
103
102
101  # ask（卖价）
-------------当前价差
99   # bid（买价）
98
97
...
```

如果 `exit_pricing.price_side` 设置为 `"ask"`，则机器人将使用 101 作为退出价格。
相应地，如果 `exit_pricing.price_side` 设置为 `"bid"`，则机器人将使用 99 作为退出价格。

根据订单方向（_做多_/_做空_），这将导致不同的结果。因此我们建议使用 `"same"` 或 `"other"` 来代替此配置。
这将产生以下定价矩阵：

| 方向 | 订单 | 设置 | 价格 | 是否跨越价差 |
|------ |--------|-----|-----|-----|
| long  | sell | ask   | 101 | 否  |
| long  | sell | bid   | 99  | 是 |
| long  | sell | same  | 101 | 否  |
| long  | sell | other | 99  | 是 |
| short | buy  | ask   | 101 | 是 |
| short | buy  | bid   | 99  | 否  |
| short | buy  | same  | 99  | 否  |
| short | buy  | other | 101 | 是 |

#### 启用订单簿时的退出价格

当启用订单簿退出交易时（`exit_pricing.use_order_book=True`），Freqtrade 从订单簿中获取 `exit_pricing.order_book_top` 个条目，并使用配置侧（`exit_pricing.price_side`）中指定为 `exit_pricing.order_book_top` 的条目作为交易退出价格。

1 表示订单簿中最顶部的条目，2 表示第二个条目，以此类推。

#### 未启用订单簿时的退出价格

以下部分使用 `side` 作为已配置的 `exit_pricing.price_side`（默认为 `"ask"`）。

当不使用订单簿时（`exit_pricing.use_order_book=False`），如果行情数据中的最佳 `side` 价格高于行情数据中的最后成交价（`last`），Freqtrade 将使用该 `side` 价格。否则（当 `side` 价格低于 `last` 价格时），它会根据 `exit_pricing.price_last_balance` 在 `side` 和 `last` 价格之间计算一个价格。

`exit_pricing.price_last_balance` 配置参数控制此行为。值为 `0.0` 将使用 `side` 价格，而 `1.0` 将使用 last 价格，介于两者之间的值将在 `side` 和 last 价格之间插值。

### 市价单定价

使用市价单时，价格应配置为使用订单簿的"正确"一侧，以允许进行真实的价格检测。
假设入场和退出都使用市价单，必须使用类似以下的配置：

``` jsonc
  "order_types": {
    "entry": "market",
    "exit": "market"
    // ...
  },
  "entry_pricing": {
    "price_side": "other",
    // ...
  },
  "exit_pricing":{
    "price_side": "other",
    // ...
  },
```

显然，如果只有一侧使用限价单，可以使用不同的定价组合。
