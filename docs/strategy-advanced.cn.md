# 高级策略

本页面介绍策略中可用的一些高级概念。
如果你刚刚入门，请先熟悉 [Freqtrade 基础](bot-basics.md) 和 [策略自定义](strategy-customization.md) 中描述的方法。

此处描述的方法调用顺序在 [机器人执行逻辑](bot-basics.md#bot-execution-logic) 中有说明。这些文档也有助于你决定哪种方法最适合你的自定义需求。

!!! Note
    回调方法*仅*在策略使用它们时才应实现。

!!! Tip
    通过运行 `freqtrade new-strategy --strategy MyAwesomeStrategy --template advanced` 可以从包含所有可用回调方法的策略模板开始

## 存储信息（持久化）

Freqtrade 允许在数据库中存储/检索与特定交易相关的用户自定义信息。

使用 trade 对象，可以通过 `trade.set_custom_data(key='my_key', value=my_value)` 存储信息，通过 `trade.get_custom_data(key='my_key')` 检索信息。每条数据记录与一笔交易和一个用户提供的键（`string` 类型）相关联。这意味着此功能只能在同时提供 trade 对象的回调中使用。

为了能够将数据存储到数据库中，freqtrade 必须将数据序列化。这是通过将数据转换为 JSON 格式的字符串来完成的。
Freqtrade 在检索时会尝试反向执行此操作，因此从策略角度来看，这应该无关紧要。

```python
from freqtrade.persistence import Trade
from datetime import timedelta

class AwesomeStrategy(IStrategy):

    def bot_loop_start(self, **kwargs) -> None:
        for trade in Trade.get_open_order_trades():
            fills = trade.select_filled_orders(trade.entry_side)
            if trade.pair == 'ETH/USDT':
                trade_entry_type = trade.get_custom_data(key='entry_type')
                if trade_entry_type is None:
                    trade_entry_type = 'breakout' if 'entry_1' in trade.enter_tag else 'dip'
                elif len(fills) > 1:
                    trade_entry_type = 'buy_up'
                trade.set_custom_data(key='entry_type', value=trade_entry_type)
        return super().bot_loop_start(**kwargs)

    def adjust_entry_price(self, trade: Trade, order: Order | None, pair: str,
                           current_time: datetime, proposed_rate: float, current_order_rate: float,
                           entry_tag: str | None, side: str, **kwargs) -> float:
        # Limit orders to use and follow SMA200 as price target for the first 10 minutes since entry trigger for BTC/USDT pair.
        if (
            pair == 'BTC/USDT'
            and entry_tag == 'long_sma200'
            and side == 'long'
            and (current_time - timedelta(minutes=10)) > trade.open_date_utc
            and order.filled == 0.0
        ):
            dataframe, _ = self.dp.get_analyzed_dataframe(pair=pair, timeframe=self.timeframe)
            current_candle = dataframe.iloc[-1].squeeze()
            # store information about entry adjustment
            existing_count = trade.get_custom_data('num_entry_adjustments', default=0)
            if not existing_count:
                existing_count = 1
            else:
                existing_count += 1
            trade.set_custom_data(key='num_entry_adjustments', value=existing_count)

            # adjust order price
            return current_candle['sma_200']

        # default: maintain existing order
        return current_order_rate

    def custom_exit(self, pair: str, trade: Trade, current_time: datetime, current_rate: float, current_profit: float, **kwargs):

        entry_adjustment_count = trade.get_custom_data(key='num_entry_adjustments')
        trade_entry_type = trade.get_custom_data(key='entry_type')
        if entry_adjustment_count is None:
            if current_profit > 0.01 and (current_time - timedelta(minutes=100) > trade.open_date_utc):
                return True, 'exit_1'
        else
            if entry_adjustment_count > 0 and if current_profit > 0.05:
                return True, 'exit_2'
            if trade_entry_type == 'breakout' and current_profit > 0.1:
                return True, 'exit_3

        return False, None
```

以上是一个简单的示例——有更简单的方法来检索交易数据，如入场调整次数。

!!! Note
    建议使用简单数据类型 `[bool, int, float, str]` 以确保在序列化需要存储的数据时不会出现问题。
    存储大量数据可能会导致意外的副作用，例如数据库变大（从而也变慢）。

!!! Warning "不可序列化的数据"
    如果提供的数据无法序列化，将记录一条警告，并且指定 `key` 对应的条目将包含 `None` 作为数据。

??? Note "所有属性"
    custom-data 通过 Trade 对象（以下假设为 `trade`）具有以下访问器：

    * `trade.get_custom_data(key='something', default=0)` - 返回给定类型的实际值。
    * `trade.get_custom_data_entry(key='something')` - 返回条目——包括元数据。值可通过 `.value` 属性访问。
    * `trade.set_custom_data(key='something', value={'some': 'value'})` - 设置或更新此交易对应的键。值必须可序列化——我们建议保持存储的数据相对较小。

    "value" 可以是任何类型（在设置和接收时）——但必须是 JSON 可序列化的。

## 存储信息（非持久化）

!!! Warning "已弃用"
    此存储信息的方法已弃用，我们建议不要使用非持久化存储。
    请改用 [持久化存储](#storing-information-persistent)。

    因此其内容已折叠。

??? Abstract "存储信息"
    存储信息可以通过在策略类中创建一个新字典来实现。

    变量名可以任意选择，但应以 `custom_` 为前缀以避免与预定义的策略变量发生命名冲突。

    ```python
    class AwesomeStrategy(IStrategy):
        # Create custom dictionary
        custom_info = {}

        def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
            # Check if the entry already exists
            if not metadata["pair"] in self.custom_info:
                # Create empty entry for this pair
                self.custom_info[metadata["pair"]] = {}

            if "crosstime" in self.custom_info[metadata["pair"]]:
                self.custom_info[metadata["pair"]]["crosstime"] += 1
            else:
                self.custom_info[metadata["pair"]]["crosstime"] = 1
    ```

    !!! Warning
        数据在机器人重启（或配置重新加载）后不会持久化。此外，数据量应保持较小（不要存储 DataFrame 之类的大对象），否则机器人将开始消耗大量内存，最终耗尽内存并崩溃。

    !!! Note
        如果数据是特定于交易对的，请确保将交易对作为字典中的键之一。

## Dataframe 访问

你可以在各种策略函数中通过从 dataprovider 查询来访问 dataframe。

``` python
from freqtrade.exchange import timeframe_to_prev_date

class AwesomeStrategy(IStrategy):
    def confirm_trade_exit(self, pair: str, trade: 'Trade', order_type: str, amount: float,
                           rate: float, time_in_force: str, exit_reason: str,
                           current_time: 'datetime', **kwargs) -> bool:
        # Obtain pair dataframe.
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)

        # Obtain last available candle. Do not use current_time to look up latest candle, because
        # current_time points to current incomplete candle whose data is not available.
        last_candle = dataframe.iloc[-1].squeeze()
        # <...>

        # In dry/live runs trade open date will not match candle open date therefore it must be
        # rounded.
        trade_date = timeframe_to_prev_date(self.timeframe, trade.open_date_utc)
        # Look up trade candle.
        trade_candle = dataframe.loc[dataframe['date'] == trade_date]
        # trade_candle may be empty for trades that just opened as it is still incomplete.
        if not trade_candle.empty:
            trade_candle = trade_candle.squeeze()
            # <...>
```

!!! Warning "使用 .iloc[-1]"
    你可以在这里使用 `.iloc[-1]`，因为 `get_analyzed_dataframe()` 只返回回测允许看到的 K 线。
    这在 `populate_*` 方法中不起作用，所以请确保不要在那些方法中使用 `.iloc[]`。
    此外，这仅从 2021.5 版本开始有效。

***

## 入场标签

当你的策略有多个入场信号时，你可以为触发的信号命名。
然后你可以在 `custom_exit` 中访问你的入场信号。

```python
def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe["enter_tag"] = ""
    signal_rsi = (qtpylib.crossed_above(dataframe["rsi"], 35))
    signal_bblower = (dataframe["bb_lowerband"] < dataframe["close"])
    # Additional conditions
    dataframe.loc[
        (
            signal_rsi
            | signal_bblower
            # ... additional signals to enter a long position
        )
        & (dataframe["volume"] > 0)
            , "enter_long"
        ] = 1
    # Concatenate the tags so all signals are kept
    dataframe.loc[signal_rsi, "enter_tag"] += "long_signal_rsi "
    dataframe.loc[signal_bblower, "enter_tag"] += "long_signal_bblower "

    return dataframe

def custom_exit(self, pair: str, trade: Trade, current_time: datetime, current_rate: float,
                current_profit: float, **kwargs):
    dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
    last_candle = dataframe.iloc[-1].squeeze()
    if "long_signal_rsi" in trade.enter_tag and last_candle["rsi"] > 80:
        return "exit_signal_rsi"
    if "long_signal_bblower" in trade.enter_tag and last_candle["high"] > last_candle["bb_upperband"]:
        return "exit_signal_bblower"
    # ...
    return None

```

!!! Note
    `enter_tag` 限制为 255 个字符，超出部分将被截断。

!!! Warning
    只有一个 `enter_tag` 列，做多和做空交易共用。
    因此，此列必须被视为"最后写入生效"（毕竟它只是一个 dataframe 列）。
    在复杂情况下，当多个信号冲突时（或信号基于不同条件再次被停用时），这可能会导致奇怪的结果，即错误的标签被应用到入场信号上。
    这些结果是策略覆盖先前标签的后果——最后一个标签将"保留"，并且将是 freqtrade 使用的标签。

## 出场标签

与 [入场标签](#enter-tag) 类似，你也可以指定出场标签。

``` python
def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe["exit_tag"] = ""
    rsi_exit_signal = (dataframe["rsi"] > 70)
    ema_exit_signal  = (dataframe["ema20"] < dataframe["ema50"])
    # Additional conditions
    dataframe.loc[
        (
            rsi_exit_signal
            | ema_exit_signal
            # ... additional signals to exit a long position
        ) &
        (dataframe["volume"] > 0)
        ,
    "exit_long"] = 1
    # Concatenate the tags so all signals are kept
    dataframe.loc[rsi_exit_signal, "exit_tag"] += "exit_signal_rsi "
    dataframe.loc[rsi_exit_signal2, "exit_tag"] += "exit_signal_rsi "

    return dataframe
```

提供的出场标签将作为出场原因使用——并在回测结果中如此显示。

!!! Note
    `exit_reason` 限制为 100 个字符，超出部分将被截断。

## 策略版本

你可以通过使用 "version" 方法并返回你希望此策略拥有的版本号来实现自定义策略版本控制。

``` python
def version(self) -> str:
    """
    Returns version of the strategy.
    """
    return "1.1"
```

!!! Note
    你应该确保同时实现适当的版本控制（如 git 仓库），因为 freqtrade 不会保留你策略的历史版本，所以由用户负责能够最终回滚到策略的先前版本。

## 派生策略

策略可以从其他策略派生。这避免了自定义策略代码的重复。你可以使用此技术来覆盖主策略的小部分，其余部分保持不变：

``` python title="user_data/strategies/myawesomestrategy.py"
class MyAwesomeStrategy(IStrategy):
    ...
    stoploss = 0.13
    trailing_stop = False
    # All other attributes and methods are here as they
    # should be in any custom strategy...
    ...

```

``` python title="user_data/strategies/MyAwesomeStrategy2.py"
from myawesomestrategy import MyAwesomeStrategy
class MyAwesomeStrategy2(MyAwesomeStrategy):
    # Override something
    stoploss = 0.08
    trailing_stop = True
```

属性和方法都可以被覆盖，以你需要的方式改变原始策略的行为。

虽然将子类保存在同一个文件中技术上是可行的，但这可能会导致 hyperopt 参数文件的一些问题，因此我们建议使用单独的策略文件，并如上所示导入父策略。

## 嵌入策略

Freqtrade 为你提供了一种将策略嵌入配置文件的简便方法。
这是通过使用 BASE64 编码并在你选择的配置文件中的策略配置字段提供此字符串来完成的。

### 将字符串编码为 BASE64

这是一个快速示例，展示如何在 Python 中生成 BASE64 字符串

```python
from base64 import urlsafe_b64encode

with open(file, 'r') as f:
    content = f.read()
content = urlsafe_b64encode(content.encode('utf-8'))
```

变量 'content' 将包含 BASE64 编码形式的策略文件。现在可以在你的配置文件中如下设置

```json
"strategy": "NameOfStrategy:BASE64String"
```

请确保 'NameOfStrategy' 与策略名称完全一致！

## 性能警告

执行策略时，有时可能会在日志中看到以下信息

> PerformanceWarning: DataFrame is highly fragmented.

这是来自 [`pandas`](https://github.com/pandas-dev/pandas) 的警告，正如警告继续说的：
使用 `pd.concat(axis=1)`。
这可能会有轻微的性能影响，通常仅在 hyperopt 期间（优化指标时）可见。

例如：

```python
for val in self.buy_ema_short.range:
    dataframe[f'ema_short_{val}'] = ta.EMA(dataframe, timeperiod=val)
```

应该重写为

```python
frames = [dataframe]
for val in self.buy_ema_short.range:
    frames.append(DataFrame({
        f'ema_short_{val}': ta.EMA(dataframe, timeperiod=val)
    }))

# Combine all dataframes, and reassign the original dataframe column
dataframe = pd.concat(frames, axis=1)
```

然而 Freqtrade 也通过在 `populate_indicators()` 方法之后立即对 dataframe 运行 `dataframe.copy()` 来应对此问题——因此其性能影响应该很小到不存在。
