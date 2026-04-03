## 保护机制

保护机制将通过临时停止单个交易对或所有交易对的交易，来保护你的策略免受意外事件和市场条件的影响。所有保护结束时间都会向上舍入到下一根蜡烛图，以避免突然的、意外的蜡烛图内买入。

!!! Tip "使用提示"
    并非所有保护机制都适用于所有策略，参数需要针对你的策略进行调优以提高性能。

    每个保护机制可以使用不同的参数配置多次，以实现不同级别的保护（短期/长期）。

!!! Note "回测"
    保护机制在回测和超参数优化中受支持，但必须通过使用 `--enable-protections` 标志显式启用。

### 可用的保护机制

* [`StoplossGuard`](#stoploss-guard) 如果在某个时间窗口内发生了一定数量的止损，则停止交易。
* [`MaxDrawdown`](#maxdrawdown) 如果达到最大回撤，则停止交易。
* [`LowProfitPairs`](#low-profit-pairs) 锁定低利润的交易对。
* [`CooldownPeriod`](#cooldown-period) 在卖出交易后不要立即进入新交易。

### 所有保护机制的通用设置

| 参数 | 描述 |
| --------- | ---------- |
| `method` | 要使用的保护机制名称。 <br> **数据类型：** 字符串，从[可用保护机制](#available-protections)中选择 |
| `stop_duration_candles` | 锁定应持续多少根蜡烛图？ <br> **数据类型：** 正整数（以蜡烛图为单位） |
| `stop_duration` | 保护锁定应持续多少分钟。 <br>不能与 `stop_duration_candles` 同时使用。 <br> **数据类型：** 浮点数（以分钟为单位） |
| `lookback_period_candles` | 只考虑在最近 `lookback_period_candles` 根蜡烛图内完成的交易。此设置可能被某些保护机制忽略。 <br> **数据类型：** 正整数（以蜡烛图为单位）。 |
| `lookback_period` | 只考虑在 `current_time - lookback_period` 之后完成的交易。 <br>不能与 `lookback_period_candles` 同时使用。 <br>此设置可能被某些保护机制忽略。 <br> **数据类型：** 浮点数（以分钟为单位） |
| `trade_limit` | 所需的最少交易数量（并非所有保护机制都使用）。 <br> **数据类型：** 正整数 |
| `unlock_at` | 定期解锁交易的时间（并非所有保护机制都使用）。 <br> **数据类型：** 字符串 <br>**输入格式：** "HH:MM"（24 小时制） |

!!! Note "持续时间"
    持续时间（`stop_duration*` 和 `lookback_period*` 可以用分钟或蜡烛图来定义）。
    为了在测试不同时间框架时更灵活，以下所有示例将使用"蜡烛图"定义。

#### Stoploss Guard

`StoplossGuard` 选择 `lookback_period` 分钟内（或使用 `lookback_period_candles` 时以蜡烛图为单位）的所有交易。
如果有 `trade_limit` 笔或更多交易触发了止损，交易将停止 `stop_duration` 分钟（或使用 `stop_duration_candles` 时以蜡烛图为单位，或使用 `unlock_at` 时直到设定时间）。

这适用于所有交易对，除非将 `only_per_pair` 设置为 true，这将只查看单个交易对。

类似地，此保护机制默认会查看所有交易（做多和做空）。对于合约机器人，设置 `only_per_side` 将使机器人只考虑一个方向，然后只锁定该方向，例如允许在一系列做多止损后继续做空。

`required_profit` 将确定止损触发所需的相对利润（或亏损）。通常不应设置此项，默认为 0.0 - 这意味着所有亏损的止损都将触发锁定。

以下示例在机器人在最近 24 根蜡烛图内触发了 4 次止损后，停止所有交易对交易 4 根蜡烛图。

``` python
@property
def protections(self):
    return [
        {
            "method": "StoplossGuard",
            "lookback_period_candles": 24,
            "trade_limit": 4,
            "stop_duration_candles": 4,
            "required_profit": 0.0,
            "only_per_pair": False,
            "only_per_side": False
        }
    ]
```

!!! Note
    `StoplossGuard` 考虑所有结果为 `"stop_loss"`、`"stoploss_on_exchange"` 和 `"trailing_stop_loss"` 且利润为负的交易。
    `trade_limit` 和 `lookback_period` 需要针对你的策略进行调优。

#### MaxDrawdown

`MaxDrawdown` 保护机制评估在当前 `lookback_period`（或 `lookback_period_candles`）内关闭的交易。
它支持 2 种计算模式：

- `calculation_mode: "ratios"`（默认）：基于累计利润比率的旧版近似计算。
- `calculation_mode: "equity"`：基于账户权益曲线的标准峰值到谷值回撤，使用起始余额和累计绝对利润。

使用 `calculation_mode: "ratios"` 时，回撤来源于累计交易利润比率，而非账户权益曲线。这是为了向后兼容而保留的，当仓位大小随时间变化时，可能与账户级别的回撤不同。

对于新配置，建议使用 `calculation_mode: "equity"`。仅当你有意依赖旧版行为时才使用 `calculation_mode: "ratios"`，特别是在固定质押金额配置中，基于比率的行为更容易理解。

如果观察到的回撤超过 `max_allowed_drawdown`，交易将在最后一笔交易后停止 `stop_duration` - 假设机器人需要一些时间让市场恢复。

以下示例在考虑所有交易对的情况下，如果最大回撤 > 20%（在最近 48 根蜡烛图内，最少 `trade_limit` 笔交易），则停止交易 12 根蜡烛图。如果需要，可以使用 `lookback_period` 和/或 `stop_duration`。

``` python
@property
def protections(self):
    return  [
        {
            "method": "MaxDrawdown",
            "calculation_mode": "equity",
            "lookback_period_candles": 48,
            "trade_limit": 20,
            "stop_duration_candles": 12,
            "max_allowed_drawdown": 0.2
        },
    ]
```

#### Low Profit Pairs

`LowProfitPairs` 使用 `lookback_period` 分钟内（或使用 `lookback_period_candles` 时以蜡烛图为单位）某个交易对的所有交易来确定整体利润比率。
如果该比率低于 `required_profit`，该交易对将被锁定 `stop_duration` 分钟（或使用 `stop_duration_candles` 时以蜡烛图为单位，或使用 `unlock_at` 时直到设定时间）。

对于合约机器人，设置 `only_per_side` 将使机器人只考虑一个方向，然后只锁定该方向，例如允许在一系列做多亏损后继续做空。

以下示例将在某个交易对在最近 6 根蜡烛图内没有达到 2% 的所需利润（且最少 2 笔交易）时，停止该交易对交易 60 分钟。

``` python
@property
def protections(self):
    return [
        {
            "method": "LowProfitPairs",
            "lookback_period_candles": 6,
            "trade_limit": 2,
            "stop_duration": 60,
            "required_profit": 0.02,
            "only_per_pair": False,
        }
    ]
```

#### Cooldown Period

`CooldownPeriod` 在退出交易后锁定交易对 `stop_duration` 分钟（或使用 `stop_duration_candles` 时以蜡烛图为单位，或使用 `unlock_at` 时直到设定时间），避免在 `stop_duration` 分钟内重新进入该交易对。

以下示例将在关闭交易后停止该交易对交易 2 根蜡烛图，让该交易对"冷却"。

``` python
@property
def protections(self):
    return  [
        {
            "method": "CooldownPeriod",
            "stop_duration_candles": 2
        }
    ]
```

!!! Note
    此保护机制仅在交易对级别应用，永远不会全局锁定所有交易对。
    此保护机制不考虑 `lookback_period`，因为它只查看最近的交易。

### 保护机制完整示例

所有保护机制可以随意组合，也可以使用不同的参数，为表现不佳的交易对创建递增的保护墙。
所有保护机制按定义的顺序进行评估。

以下示例假设时间框架为 1 小时：

* 在卖出后锁定每个交易对额外 5 根蜡烛图（`CooldownPeriod`），给其他交易对一个成交的机会。
* 如果过去 2 天（`48 * 1h 蜡烛图`）内有 20 笔交易导致最大回撤超过 20%，则停止交易 4 小时（`4 * 1h 蜡烛图`）（`MaxDrawdown`）。
* 如果所有交易对在 1 天（`24 * 1h 蜡烛图`）限制内发生了超过 4 次止损，则停止交易（`StoplossGuard`）。
* 锁定所有在最近 6 小时（`6 * 1h 蜡烛图`）内有 2 笔交易且综合利润比率低于 0.02（<2%）的交易对（`LowProfitPairs`）。
* 锁定所有在最近 24 小时（`24 * 1h 蜡烛图`）内利润低于 0.01（<1%）、最少 4 笔交易的交易对 2 根蜡烛图。

``` python
from freqtrade.strategy import IStrategy

class AwesomeStrategy(IStrategy)
    timeframe = '1h'

    @property
    def protections(self):
        return [
            {
                "method": "CooldownPeriod",
                "stop_duration_candles": 5
            },
            {
                "method": "MaxDrawdown",
                "calculation_mode": "equity",
                "lookback_period_candles": 48,
                "trade_limit": 20,
                "stop_duration_candles": 4,
                "max_allowed_drawdown": 0.2
            },
            {
                "method": "StoplossGuard",
                "lookback_period_candles": 24,
                "trade_limit": 4,
                "stop_duration_candles": 2,
                "only_per_pair": False
            },
            {
                "method": "LowProfitPairs",
                "lookback_period_candles": 6,
                "trade_limit": 2,
                "stop_duration_candles": 60,
                "required_profit": 0.02
            },
            {
                "method": "LowProfitPairs",
                "lookback_period_candles": 24,
                "trade_limit": 4,
                "stop_duration_candles": 2,
                "required_profit": 0.01
            }
        ]
    # ...
```
