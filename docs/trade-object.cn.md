# Trade 对象

## Trade

Freqtrade 进入的仓位存储在 `Trade` 对象中——该对象会持久化到数据库。
它是 freqtrade 的核心概念——您会在文档的许多章节中遇到它，这些章节很可能会指向此处。

它将在许多[策略回调](strategy-callbacks.md)中传递给策略。传递给策略的对象不能直接修改。间接修改可能基于回调结果发生。

## Trade - 可用属性

以下属性/属性可用于每个单独的交易——可以通过 `trade.<property>` 使用（例如 `trade.pair`）。

|  属性 | 数据类型 | 描述 |
|------------|-------------|-------------|
| `pair` | string | 此交易的交易对。 |
| `safe_base_currency` | string | 基础货币的兼容层。 |
| `safe_quote_currency` | string | 报价货币的兼容层。 |
| `is_open` | boolean | 交易当前是否开放，还是已经结束。 |
| `exchange` | string | 执行此交易的交易所。 |
| `open_rate` | float | 此交易的入场价格（如果有加仓调整，则为平均入场价格）。 |
| `open_rate_requested` | float | 开仓时请求的价格。 |
| `open_trade_value` | float | 包含手续费的开仓交易价值。 |
| `close_rate` | float | 平仓价格——仅在 is_open = False 时设置。 |
| `close_rate_requested` | float | 请求的平仓价格。 |
| `safe_close_rate` | float | 平仓价格或 `close_rate_requested` 或 0.0（如果两者都不可用）。仅在交易关闭后有意义。 |
| `stake_amount` | float | 以 Stake（或报价）货币计的金额。 |
| `max_stake_amount` | float | 此交易中使用的最大 stake 金额（所有已成交入场订单的总和）。 |
| `amount` | float | 当前持有的以资产/基础货币计的数量。在初始订单成交之前为 0.0。 |
| `amount_requested` | float | 作为首笔入场订单最初请求的数量。 |
| `open_date` | datetime | 交易开仓时间戳 **请使用 `open_date_utc` 代替** |
| `open_date_utc` | datetime | 交易开仓时间戳——UTC 时间。 |
| `close_date` | datetime | 交易平仓时间戳 **请使用 `close_date_utc` 代替** |
| `close_date_utc` | datetime | 交易平仓时间戳——UTC 时间。 |
| `close_profit` | float | 交易关闭时的相对利润。`0.01` == 1% |
| `close_profit_abs` | float | 交易关闭时的绝对利润（以 stake 货币计）。 |
| `realized_profit` | float | 交易仍在开放期间已实现的绝对利润（以 stake 货币计）。 |
| `leverage` | float | 此交易使用的杠杆倍数——现货市场默认为 1.0。 |
| `enter_tag` | string | 通过 dataframe 中的 `enter_tag` 列在入场时提供的标签。 |
| `exit_reason` | string | 交易退出的原因。 |
| `exit_order_status` | string | 退出订单的状态。 |
| `strategy` | string | 用于此交易的策略名称。 |
| `timeframe` | int | 用于此交易的时间框架。 |
| `is_short` | boolean | 做空交易为 True，否则为 False。 |
| `orders` | Order[] | 附加到此交易的订单对象列表（包括已成交和已取消的订单）。 |
| `date_last_filled_utc` | datetime | 最后一笔已成交订单的时间。 |
| `date_entry_fill_utc` | datetime | 第一笔已成交入场订单的日期。 |
| `entry_side` | "buy" / "sell" | 交易入场的订单方向。 |
| `exit_side` | "buy" / "sell" | 将导致交易退出/减仓的订单方向。 |
| `trade_direction` | "long" / "short" | 交易方向的文本表示——long 或 short。 |
| `max_rate` | float | 此交易期间达到的最高价格。不是 100% 精确。 |
| `min_rate` | float | 此交易期间达到的最低价格。不是 100% 精确。 |
| `nr_of_successful_entries` | int | 成功（已成交）入场订单的数量。 |
| `nr_of_successful_exits` | int | 成功（已成交）退出订单的数量。 |
| `has_open_position` | boolean | 此交易是否有开放仓位（amount > 0）。仅在初始入场订单未成交时为 false。 |
| `has_open_orders` | boolean | 交易是否有未完成的订单（不包括止损订单）。 |
| `has_open_sl_orders` | boolean | 此交易是否有未完成的止损订单。 |
| `open_orders` | Order[] | 此交易的所有未完成订单（不包括止损订单）。 |
| `open_sl_orders` | Order[] | 此交易的所有未完成止损订单。 |
| `fully_canceled_entry_order_count` | int | 完全取消的入场订单数量。 |
| `canceled_exit_order_count` | int | 已取消的退出订单数量。 |

### 止损相关属性

|  属性 | 数据类型 | 描述 |
|------------|-------------|-------------|
| `stop_loss` | float | 止损的绝对值。 |
| `stop_loss_pct` | float | 止损的相对值。 |
| `initial_stop_loss` | float | 初始止损的绝对值。 |
| `initial_stop_loss_pct` | float | 初始止损的相对值。 |
| `stoploss_last_update_utc` | datetime | 最后一次交易所止损订单更新的时间戳。 |
| `stoploss_or_liquidation` | float | 返回止损价或强制平仓价中更严格的那个，对应止损将触发的价格。 |

### 期货/保证金交易属性

|  属性 | 数据类型 | 描述 |
|------------|-------------|-------------|
| `liquidation_price` | float | 杠杆交易的强制平仓价格。 |
| `interest_rate` | float | 保证金交易的利率。 |
| `funding_fees` | float | 期货交易的总资金费用。 |

## 类方法

以下是类方法——它们返回通用信息，通常会对数据库执行显式查询。
可以通过 `Trade.<method>` 使用——例如 `open_trades = Trade.get_open_trade_count()`

!!! Warning "回测/超参数优化"
    大多数方法在回测/超参数优化和实盘/模拟模式下都可使用。
    在回测期间，仅限于在[策略回调](strategy-callbacks.md)中使用。在 `populate_*()` 方法中使用不受支持，会导致错误的结果。

### get_trades_proxy

当您的策略需要有关现有（开放或关闭）交易的信息时——最好使用 `Trade.get_trades_proxy()`。

用法：

``` python
from freqtrade.persistence import Trade
from datetime import timedelta

# ...
trade_hist = Trade.get_trades_proxy(pair='ETH/USDT', is_open=False, open_date=current_date - timedelta(days=2))

```

`get_trades_proxy()` 支持以下关键字参数。所有参数都是可选的——不带参数调用 `get_trades_proxy()` 将返回数据库中所有交易的列表。

* `pair` 例如 `pair='ETH/USDT'`
* `is_open` 例如 `is_open=False`
* `open_date` 例如 `open_date=current_date - timedelta(days=2)`
* `close_date` 例如 `close_date=current_date - timedelta(days=5)`

### get_open_trade_count

获取当前开放交易的数量

``` python
from freqtrade.persistence import Trade
# ...
open_trades = Trade.get_open_trade_count()
```

### get_total_closed_profit

获取机器人迄今为止产生的总利润。
聚合所有已关闭交易的 `close_profit_abs`。

``` python
from freqtrade.persistence import Trade

# ...
profit = Trade.get_total_closed_profit()
```

### total_open_trades_stakes

获取当前在交易中的总 stake_amount。

``` python
from freqtrade.persistence import Trade

# ...
profit = Trade.total_open_trades_stakes()
```

## 不支持在回测/超参数优化中使用的类方法

以下类方法不支持在回测/超参数优化模式下使用。

### get_overall_performance

获取整体表现——类似于 `/performance` Telegram 命令。

``` python
from freqtrade.persistence import Trade

# ...
if self.config['runmode'].value in ('live', 'dry_run'):
    performance = Trade.get_overall_performance()
```

示例返回值：ETH/BTC 有 5 笔交易，总利润为 1.5%（比率为 0.015）。

``` json
{"pair": "ETH/BTC", "profit": 0.015, "count": 5}
```

### get_trading_volume

根据订单获取总交易量。

``` python
from freqtrade.persistence import Trade

# ...
volume = Trade.get_trading_volume()
```

## Order 对象

`Order` 对象代表交易所上的一个订单（或模拟交易模式下的模拟订单）。
`Order` 对象始终与其对应的 [`Trade`](#trade-对象) 关联，只有在交易的上下文中才真正有意义。

### Order - 可用属性

Order 对象通常附加到一笔交易上。
这里的大多数属性可能为 None，因为它们取决于交易所的响应。

|  属性 | 数据类型 | 描述 |
|------------|-------------|-------------|
| `trade` | Trade | 此订单所附加的 Trade 对象 |
| `ft_pair` | string | 此订单对应的交易对 |
| `ft_is_open` | boolean | 订单是否仍然开放？ |
| `ft_order_side` | string | 订单方向（'buy'、'sell' 或 'stoploss'） |
| `ft_cancel_reason` | string | 订单被取消的原因 |
| `ft_order_tag` | string | 自定义订单标签 |
| `order_id` | string | 交易所订单 ID |
| `order_type` | string | 交易所定义的订单类型——通常为 market、limit 或 stoploss |
| `status` | string | 由 [ccxt 订单结构](https://docs.ccxt.com/#/README?id=order-structure)定义的状态。通常为 open、closed、expired、canceled 或 rejected |
| `side` | string | buy 或 sell |
| `price` | float | 下单价格 |
| `average` | float | 订单成交均价 |
| `amount` | float | 以基础货币计的数量 |
| `filled` | float | 已成交数量（以基础货币计）（请使用 `safe_filled` 代替） |
| `safe_filled` | float | 已成交数量（以基础货币计）——保证不为 None |
| `safe_amount` | float | 数量——如果为 None 则回退到 ft_amount |
| `safe_price` | float | 价格——依次回退到 average、price、stop_price、ft_price |
| `safe_placement_price` | float | 下单时的价格 |
| `remaining` | float | 剩余数量（请使用 `safe_remaining` 代替） |
| `safe_remaining` | float | 剩余数量——从交易所获取或计算得出。 |
| `safe_cost` | float | 订单成本——保证不为 None |
| `safe_fee_base` | float | 以基础货币计的手续费——保证不为 None |
| `safe_amount_after_fee` | float | 扣除手续费后的数量 |
| `cost` | float | 订单成本——通常为 average * filled（*期货交易中取决于交易所，可能包含或不包含杠杆的成本，且可能以合约为单位。*） |
| `stop_price` | float | 止损订单的止损价。非止损订单为空。 |
| `stake_amount` | float | 此订单使用的 stake 金额。 |
| `stake_amount_filled` | float | 此订单已成交的 stake 金额。 |
| `order_date` | datetime | 订单创建日期 **请使用 `order_date_utc` 代替** |
| `order_date_utc` | datetime | 订单创建日期（UTC 时间） |
| `order_filled_date` | datetime | 订单成交日期 **请使用 `order_filled_utc` 代替** |
| `order_filled_utc` | datetime | 订单成交日期 |
| `order_update_date` | datetime | 最后订单更新日期 |
