# Hyperopt

本页介绍如何通过寻找最优参数来调优你的策略，这一过程称为超参数优化。机器人使用 `optuna` 包中的算法来完成此任务。
搜索过程会占满你所有的 CPU 核心，让你的笔记本听起来像战斗机一样嗡嗡作响，而且仍然需要很长时间。

通常，最优参数的搜索从一些随机组合开始（详见[下文](#reproducible-results)），然后使用 optuna 的采样算法之一（目前为 NSGAIIISampler）在搜索超空间中快速找到能最小化[损失函数](#loss-functions)值的参数组合。

Hyperopt 需要可用的历史数据，就像回测一样（hyperopt 会使用不同的参数多次运行回测）。
要了解如何获取你感兴趣的交易对和交易所的数据，请前往文档的[数据下载](data-download.md)部分。

!!! Bug
    当仅使用 1 个 CPU 核心时，Hyperopt 可能会崩溃，详见 [Issue #1133](https://github.com/freqtrade/freqtrade/issues/1133)

!!! Note
    自 2021.4 版本起，你不再需要编写单独的 hyperopt 类，可以直接在策略中配置参数。
    旧方法支持到 2021.8，并在 2021.9 中被移除。

## 安装 hyperopt 依赖

由于 Hyperopt 依赖不是运行机器人本身所必需的，而且体积较大，在某些平台（如 Raspberry PI）上不易构建，因此默认不会安装。在运行 Hyperopt 之前，你需要按照以下部分的说明安装相应的依赖。

!!! Note
    由于 Hyperopt 是一个资源密集型过程，不建议也不支持在 Raspberry Pi 上运行。

### Docker

Docker 镜像已包含 hyperopt 依赖，无需额外操作。

### 简易安装脚本 (setup.sh) / 手动安装

```bash
source .venv/bin/activate
pip install -r requirements-hyperopt.txt
```

## Hyperopt 命令参考

--8<-- "commands/hyperopt.md"

### Hyperopt 检查清单

Hyperopt 中所有任务/可能性的检查清单

根据你想要优化的空间，只需要以下部分内容：

* 定义带有 `space='buy'` 的参数 - 用于入场信号优化
* 定义带有 `space='sell'` 的参数 - 用于出场信号优化
* 定义带有 `space='enter'` 的参数 - 用于入场信号优化
* 定义带有 `space='exit'` 的参数 - 用于出场信号优化
* 定义带有 `space='protection'` 的参数 - 用于保护机制优化
* 定义带有 `space='random_spacename'` 的参数 - 用于更好地控制哪些参数一起优化

选择最适合参数的空间名称。我们建议使用 `buy` / `sell` 或 `enter` / `exit` 以保持清晰（不过在技术上没有限制）。

!!! Note
    `populate_indicators` 需要创建任何空间可能使用的所有指标，否则 hyperopt 将无法工作。


极少数情况下，你可能还需要创建一个名为 `HyperOpt` 的[嵌套类](advanced-hyperopt.md#overriding-pre-defined-spaces)并实现以下方法：

* `roi_space` - 用于自定义 ROI 优化（如果你需要的 ROI 参数范围与默认值不同）
* `generate_roi_table` - 用于自定义 ROI 优化（如果你需要的 ROI 表中的值范围与默认值不同，或者 ROI 表中的条目数（步数）与默认的 4 步不同）
* `stoploss_space` - 用于自定义止损优化（如果你需要的止损参数范围与默认值不同）
* `trailing_space` - 用于自定义追踪止损优化（如果你需要的追踪止损参数范围与默认值不同）
* `max_open_trades_space` - 用于自定义 max_open_trades 优化（如果你需要的 max_open_trades 参数范围与默认值不同）

!!! Tip "快速优化 ROI、stoploss 和 trailing stoploss"
    你可以在不修改策略的情况下快速优化 `roi`、`stoploss` 和 `trailing` 空间。

    ``` bash
    # Have a working strategy at hand.
    freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --spaces roi stoploss trailing --strategy MyWorkingStrategy --config config.json -e 100
    ```

### Hyperopt 执行逻辑

Hyperopt 首先会将你的数据加载到内存中，然后对每个交易对运行一次 `populate_indicators()` 来生成所有指标，除非指定了 `--analyze-per-epoch`。

Hyperopt 随后会启动不同的进程（处理器数量，或 `-j <n>`），反复运行回测，更改属于 `--spaces` 定义的参数。

对于每组新参数，freqtrade 会先运行 `populate_entry_trend()`，然后运行 `populate_exit_trend()`，最后运行常规回测流程来模拟交易。

回测完成后，结果会传入[损失函数](#loss-functions)，评估本次结果是否优于之前的结果。
根据损失函数的结果，hyperopt 将决定下一轮回测中尝试的下一组参数。

### 配置你的守卫条件和触发条件

你需要在策略文件中修改两个地方来添加新的 hyperopt 优化参数：

* 在类级别定义 hyperopt 需要优化的参数。
* 在 `populate_entry_trend()` 中 - 使用定义的参数值而非硬编码常量。

这里有两种不同类型的指标：1. `守卫条件（guards）` 和 2. `触发条件（triggers）`。

1. 守卫条件是类似"如果 ADX < 10 则永不入场"或"如果当前价格高于 EMA10 则永不入场"的条件。
2. 触发条件是在特定时刻实际触发入场的条件，如"当 EMA5 上穿 EMA10 时入场"或"当收盘价触及布林带下轨时入场"。

!!! Hint "守卫条件和触发条件"
    从技术上讲，守卫条件和触发条件没有区别。
    但是，本指南会做出这种区分，以明确信号不应"持续存在"。
    持续存在的信号是指在多个 K 线上保持活跃的信号。这可能导致延迟入场（在信号即将消失时才入场 - 这意味着成功的机会比刚开始时低得多）。

超参数优化将在每个 epoch 轮次中选择一个触发条件和可能的多个守卫条件。

#### 出场信号优化

与上面的入场信号类似，出场信号也可以被优化。
将相应的设置放入以下方法中：

* 在类级别定义 hyperopt 需要优化的参数，命名为 `sell_*`，或通过显式定义 `space='sell'`。
* 在 `populate_exit_trend()` 中 - 使用定义的参数值而非硬编码常量。

配置和规则与买入信号相同。

## 解密

假设你很好奇：应该使用 MACD 交叉还是布林带下轨来触发做多入场。
你还想知道应该使用 RSI 还是 ADX 来辅助这些决策。
如果决定使用 RSI 或 ADX，应该使用什么值？

那么让我们使用超参数优化来解答这个谜题。

### 定义要使用的指标

我们从计算策略将要使用的指标开始。

``` python
class MyAwesomeStrategy(IStrategy):

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        Generate all indicators used by the strategy
        """
        dataframe['adx'] = ta.ADX(dataframe)
        dataframe['rsi'] = ta.RSI(dataframe)
        macd = ta.MACD(dataframe)
        dataframe['macd'] = macd['macd']
        dataframe['macdsignal'] = macd['macdsignal']
        dataframe['macdhist'] = macd['macdhist']

        bollinger = ta.BBANDS(dataframe, timeperiod=20, nbdevup=2.0, nbdevdn=2.0)
        dataframe['bb_lowerband'] = bollinger['lowerband']
        dataframe['bb_middleband'] = bollinger['middleband']
        dataframe['bb_upperband'] = bollinger['upperband']
        return dataframe
```

### 可超参数优化的参数

接下来我们定义可超参数优化的参数：

```python
class MyAwesomeStrategy(IStrategy):
    buy_adx = DecimalParameter(20, 40, decimals=1, default=30.1, space="buy")
    buy_rsi = IntParameter(20, 40, default=30, space="buy")
    buy_adx_enabled = BooleanParameter(default=True, space="buy")
    buy_rsi_enabled = CategoricalParameter([True, False], default=False, space="buy")
    buy_trigger = CategoricalParameter(["bb_lower", "macd_cross_signal"], default="bb_lower", space="buy")
```

上面的定义表示：我有五个参数想要随机组合以找到最佳组合。
`buy_rsi` 是一个整数参数，将在 20 到 40 之间测试。此空间大小为 20。
`buy_adx` 是一个小数参数，将在 20 到 40 之间以 1 位小数精度评估（因此值为 20.1、20.2、...）。此空间大小为 200。
然后我们有三个分类变量。前两个为 `True` 或 `False`。
我们用它们来启用或禁用 ADX 和 RSI 守卫条件。
最后一个我们称为 `trigger`，用于决定使用哪个买入触发条件。

!!! Note "参数空间分配"
    - 参数必须被赋值给名为 `buy_*`、`sell_*`、`enter_*` 或 `exit_*` 或 `protection_*` 的变量 - 或者通过参数显式指定空间（`space='buy'`、`space='sell'`、`space='protection'`）。
    - 分配冲突的参数（例如 `buy_adx = IntParameter(4, 24, default=14, space='sell')`）将使用显式空间分配。
    - 如果某个空间没有可用参数，运行 hyperopt 时你会收到找不到空间的错误。
    空间不明确的参数（例如 `adx_period = IntParameter(4, 24, default=14)` - 既没有显式也没有隐式空间）将不会被检测到，因此会被忽略。
    空间也可以自定义命名（例如 `space='my_custom_space'`），唯一的限制是空间名称不能是 `all`、`default` - 且必须是有效的 Python 标识符。

那么让我们使用这些值编写买入策略：

```python
    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        conditions = []
        # GUARDS AND TRENDS
        if self.buy_adx_enabled.value:
            conditions.append(dataframe['adx'] > self.buy_adx.value)
        if self.buy_rsi_enabled.value:
            conditions.append(dataframe['rsi'] < self.buy_rsi.value)

        # TRIGGERS
        if self.buy_trigger.value == 'bb_lower':
            conditions.append(dataframe['close'] < dataframe['bb_lowerband'])
        if self.buy_trigger.value == 'macd_cross_signal':
            conditions.append(qtpylib.crossed_above(
                dataframe['macd'], dataframe['macdsignal']
            ))

        # Check that volume is not 0
        conditions.append(dataframe['volume'] > 0)

        if conditions:
            dataframe.loc[
                reduce(lambda x, y: x & y, conditions),
                'enter_long'] = 1

        return dataframe
```

Hyperopt 现在会使用不同的值组合多次（epochs）调用 `populate_entry_trend()`。
它将使用给定的历史数据，根据上述函数生成的买入信号模拟交易。
根据结果，hyperopt 会告诉你哪个参数组合产生了最佳结果（基于配置的[损失函数](#loss-functions)）。

!!! Note
    上述设置期望在填充的指标中找到 ADX、RSI 和布林带。
    当你想测试机器人当前未使用的指标时，请记得
    将其添加到策略或 hyperopt 文件中的 `populate_indicators()` 方法中。

## 参数类型

有四种参数类型，各自适用于不同的目的。

* `IntParameter` - 定义一个具有搜索空间上下界的整数参数。
* `DecimalParameter` - 定义一个具有有限小数位数（默认 3 位）的浮点参数。在大多数情况下应优先于 `RealParameter` 使用。
* `RealParameter` - 定义一个具有上下界但无精度限制的浮点参数。很少使用，因为它创建了一个近乎无限可能性的空间。
* `CategoricalParameter` - 定义一个具有预定选项数量的参数。
* `BooleanParameter` - `CategoricalParameter([True, False])` 的简写 - 非常适合"启用"类参数。

### 参数选项

有两个参数选项可以帮助你快速测试各种想法：

* `optimize` - 设置为 `False` 时，该参数将不会包含在优化过程中。（默认值：True）
* `load` - 设置为 `False` 时，之前 hyperopt 运行的结果（在策略或 JSON 输出文件中的 `buy_params` 和 `sell_params`）将不会作为后续 hyperopt 的起始值使用。将使用参数中指定的默认值代替。（默认值：True）

!!! Tip "`load=False` 对回测的影响"
    请注意，将 `load` 选项设置为 `False` 意味着回测也将使用参数中指定的默认值，而*不是*通过超参数优化找到的值。

!!! Warning
    可超参数优化的参数不能在 `populate_indicators` 中使用 - 因为 hyperopt 不会在每个 epoch 重新计算指标，所以在这种情况下会使用起始值。

## 优化指标参数

假设你有一个简单的策略构想 - 一个 EMA 交叉策略（2 条移动平均线交叉） - 你想找到此策略的理想参数。
默认情况下，我们假设止损为 5% - 止盈（`minimal_roi`）为 10% - 这意味着 freqtrade 将在达到 10% 利润时卖出。

``` python
from pandas import DataFrame
from functools import reduce

import talib.abstract as ta

from freqtrade.strategy import (BooleanParameter, CategoricalParameter, DecimalParameter,
                                IStrategy, IntParameter)
import freqtrade.vendor.qtpylib.indicators as qtpylib

class MyAwesomeStrategy(IStrategy):
    stoploss = -0.05
    timeframe = '15m'
    minimal_roi = {
        "0":  0.10
    }
    # Define the parameter spaces
    buy_ema_short = IntParameter(3, 50, default=5)
    buy_ema_long = IntParameter(15, 200, default=50)


    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """Generate all indicators used by the strategy"""

        # Calculate all ema_short values
        for val in self.buy_ema_short.range:
            dataframe[f'ema_short_{val}'] = ta.EMA(dataframe, timeperiod=val)

        # Calculate all ema_long values
        for val in self.buy_ema_long.range:
            dataframe[f'ema_long_{val}'] = ta.EMA(dataframe, timeperiod=val)

        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        conditions = []
        conditions.append(qtpylib.crossed_above(
                dataframe[f'ema_short_{self.buy_ema_short.value}'], dataframe[f'ema_long_{self.buy_ema_long.value}']
            ))

        # Check that volume is not 0
        conditions.append(dataframe['volume'] > 0)

        if conditions:
            dataframe.loc[
                reduce(lambda x, y: x & y, conditions),
                'enter_long'] = 1
        return dataframe

    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        conditions = []
        conditions.append(qtpylib.crossed_above(
                dataframe[f'ema_long_{self.buy_ema_long.value}'], dataframe[f'ema_short_{self.buy_ema_short.value}']
            ))

        # Check that volume is not 0
        conditions.append(dataframe['volume'] > 0)

        if conditions:
            dataframe.loc[
                reduce(lambda x, y: x & y, conditions),
                'exit_long'] = 1
        return dataframe
```

详细解释：

使用 `self.buy_ema_short.range` 将返回一个包含参数最小值和最大值之间所有条目的 range 对象。
在本例中（`IntParameter(3, 50, default=5)`），循环将遍历 3 到 50 之间的所有数字（`[3, 4, 5, ... 49, 50]`）。
通过在循环中使用此功能，hyperopt 将生成 48 个新列（`['buy_ema_3', 'buy_ema_4', ... , 'buy_ema_50']`）。

Hyperopt 本身随后会使用选定的值来创建买入和卖出信号。

虽然此策略很可能过于简单而无法提供持续盈利，但它可以作为优化指标参数的示例。

!!! Note
    `self.buy_ema_short.range` 在 hyperopt 和其他模式下的行为不同。对于 hyperopt，上面的示例可能会生成 48 个新列，但对于所有其他模式（回测、模拟/实盘），它只会为所选值生成一列。因此你应该避免使用显式值（`self.buy_ema_short.value` 以外的值）来引用结果列。

!!! Note
    `range` 属性也可用于 `DecimalParameter` 和 `CategoricalParameter`。由于搜索空间无限，`RealParameter` 不提供此属性。

??? Hint "性能提示"
    在正常的超参数优化过程中，指标只计算一次并提供给每个 epoch，内存使用量随核心数增加而线性增长。由于这也有性能影响，有两种替代方案可以减少内存使用：

    * 将 `ema_short` 和 `ema_long` 的计算从 `populate_indicators()` 移到 `populate_entry_trend()`。由于 `populate_entry_trend()` 每个 epoch 都会被计算，你不需要使用 `.range` 功能。
    * hyperopt 提供了 `--analyze-per-epoch` 选项，它会将 `populate_indicators()` 的执行移到 epoch 进程中，每个参数每个 epoch 只计算一个值，而不是使用 `.range` 功能。在这种情况下，`.range` 功能只会返回实际使用的值。

    这些替代方案会减少内存使用，但会增加 CPU 使用。不过，你的超参数优化运行将不太可能因为内存不足（OOM）问题而失败。

    无论你使用的是 `.range` 功能还是上述替代方案，你都应该尽量使用尽可能小的空间范围，因为这会改善 CPU/内存使用。

## 优化保护机制

Freqtrade 也可以优化保护机制。如何优化保护机制取决于你，以下仅作为示例。

策略只需将"protections"条目定义为返回保护配置列表的属性。

``` python
from pandas import DataFrame
from functools import reduce

import talib.abstract as ta

from freqtrade.strategy import (BooleanParameter, CategoricalParameter, DecimalParameter,
                                IStrategy, IntParameter)
import freqtrade.vendor.qtpylib.indicators as qtpylib

class MyAwesomeStrategy(IStrategy):
    stoploss = -0.05
    timeframe = '15m'
    # Define the parameter spaces
    cooldown_lookback = IntParameter(2, 48, default=5, space="protection", optimize=True)
    stop_duration = IntParameter(12, 200, default=5, space="protection", optimize=True)
    use_stop_protection = BooleanParameter(default=True, space="protection", optimize=True)


    @property
    def protections(self):
        prot = []

        prot.append({
            "method": "CooldownPeriod",
            "stop_duration_candles": self.cooldown_lookback.value
        })
        if self.use_stop_protection.value:
            prot.append({
                "method": "StoplossGuard",
                "lookback_period_candles": 24 * 3,
                "trade_limit": 4,
                "stop_duration_candles": self.stop_duration.value,
                "only_per_pair": False
            })

        return prot

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # ...

```

然后你可以按如下方式运行 hyperopt：
`freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --strategy MyAwesomeStrategy --spaces protection`

!!! Note
    protection 空间不是默认空间的一部分，只能通过参数 Hyperopt 接口使用，不能通过旧版 hyperopt 接口（需要单独的 hyperopt 文件）使用。
    如果选择了 protection 空间，Freqtrade 也会自动更改"--enable-protections"标志。

!!! Warning
    如果 protections 被定义为属性，配置文件中的条目将被忽略。
    因此建议不要在配置文件中定义 protections。

### 从之前的属性设置迁移

从之前的设置迁移非常简单，只需将 protections 条目转换为属性即可完成。
简而言之，以下配置将被转换为下面的形式。

``` python
class MyAwesomeStrategy(IStrategy):
    protections = [
        {
            "method": "CooldownPeriod",
            "stop_duration_candles": 4
        }
    ]
```

结果

``` python
class MyAwesomeStrategy(IStrategy):

    @property
    def protections(self):
        return [
            {
                "method": "CooldownPeriod",
                "stop_duration_candles": 4
            }
        ]
```

然后你显然还会将可能有趣的条目更改为参数，以允许超参数优化。

### 优化 `max_entry_position_adjustment`

虽然 `max_entry_position_adjustment` 不是一个独立的空间，但仍然可以通过上面展示的属性方法在 hyperopt 中使用。

``` python
from pandas import DataFrame
from functools import reduce

import talib.abstract as ta

from freqtrade.strategy import (BooleanParameter, CategoricalParameter, DecimalParameter,
                                IStrategy, IntParameter)
import freqtrade.vendor.qtpylib.indicators as qtpylib

class MyAwesomeStrategy(IStrategy):
    stoploss = -0.05
    timeframe = '15m'

    # Define the parameter spaces
    max_epa = CategoricalParameter([-1, 0, 1, 3, 5, 10], default=1, space="buy", optimize=True)

    @property
    def max_entry_position_adjustment(self):
        return self.max_epa.value


    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # ...
```

??? Tip "使用 `IntParameter`"
    你也可以使用 `IntParameter` 进行此优化，但必须显式返回整数：
    ``` python
    max_epa = IntParameter(-1, 10, default=1, space="buy", optimize=True)

    @property
    def max_entry_position_adjustment(self):
        return int(self.max_epa.value)
    ```

## 损失函数

每次超参数调优都需要一个目标。这通常定义为损失函数（有时也称为目标函数），对于更理想的结果应该减小，对于不好的结果应该增大。

必须通过 `--hyperopt-loss <Class-name>` 参数指定损失函数（或者可以通过配置文件中的 `"hyperopt_loss"` 键来指定）。
该类应该在 `user_data/hyperopts/` 目录中的独立文件中。

目前，以下损失函数是内置的：

* `ShortTradeDurHyperOptLoss` -（默认的旧版 Freqtrade 超参数优化损失函数）- 主要针对短交易持续时间和避免亏损。
* `OnlyProfitHyperOptLoss` - 仅考虑利润金额。
* `SharpeHyperOptLoss` - 优化基于交易回报相对于标准差计算的 Sharpe Ratio。
* `SharpeHyperOptLossDaily` - 优化基于**每日**交易回报相对于标准差计算的 Sharpe Ratio。
* `SortinoHyperOptLoss` - 优化基于交易回报相对于**下行**标准差计算的 Sortino Ratio。
* `SortinoHyperOptLossDaily` - 优化基于**每日**交易回报相对于**下行**标准差计算的 Sortino Ratio。
* `MaxDrawDownHyperOptLoss` - 优化最大绝对回撤。
* `MaxDrawDownRelativeHyperOptLoss` - 同时优化最大绝对回撤和最大相对回撤。
* `MaxDrawDownPerPairHyperOptLoss` - 计算每个交易对的利润/回撤比率并返回最差结果作为目标，强制 hyperopt 为交易对列表中的所有交易对优化参数。这样可以防止一个或多个结果良好的交易对膨胀指标，而结果不佳的交易对未被代表因此未被优化。
* `CalmarHyperOptLoss` - 优化基于交易回报相对于最大回撤计算的 Calmar Ratio。
* `ProfitDrawDownHyperOptLoss` - 通过最大利润和最小回撤目标进行优化。hyperoptloss 文件中的 `DRAWDOWN_MULT` 变量可以调整，以对回撤目的更严格或更灵活。
* `MultiMetricHyperOptLoss` - 通过多个关键指标进行优化以实现均衡性能。主要关注最大化利润和最小化回撤，同时也考虑 Profit Factor、Expectancy Ratio 和 Winrate 等额外指标。此外，它对交易次数较少的 epoch 施加惩罚，鼓励具有足够交易频率的策略。

自定义损失函数的创建在文档的[高级 Hyperopt](advanced-hyperopt.md)部分中介绍。

## 执行 Hyperopt

更新 hyperopt 配置后，你就可以运行它了。
由于 hyperopt 会尝试大量组合来找到最佳参数，获得好的结果需要时间。

我们强烈建议使用 `screen` 或 `tmux` 来防止连接断开。

```bash
freqtrade hyperopt --config config.json --hyperopt-loss <hyperoptlossname> --strategy <strategyname> -e 500 --spaces all
```

`-e` 选项将设置 hyperopt 执行的评估次数。由于 hyperopt 使用贝叶斯搜索，一次运行太多 epoch 可能不会产生更好的结果。经验表明，在 500-1000 个 epoch 后，最佳结果通常不会有太大改善。
`--early-stop` 选项将设置在多少个 epoch 没有改善后 hyperopt 将停止。一个好的值是总 epoch 数的 20-30%。任何大于 0 且小于 20 的值将被替换为 20。早停默认禁用（`--early-stop=0`）

进行多次运行（执行），每次几千个 epoch 并使用不同的随机状态，很可能会产生不同的结果。

`--spaces all` 选项决定所有可能的参数都应被优化。可能的选项列在下面。

!!! Note
    Hyperopt 将以 hyperopt 开始时间的时间戳存储 hyperopt 结果。
    读取命令（`hyperopt-list`、`hyperopt-show`）可以使用 `--hyperopt-filename <filename>` 来读取和显示旧的 hyperopt 结果。
    你可以通过 `ls -l user_data/hyperopt_results/` 找到文件名列表。

### 使用不同的历史数据源执行 Hyperopt

如果你想使用磁盘上的备用历史数据集进行 hyperopt 参数优化，请使用 `--datadir PATH` 选项。默认情况下，hyperopt 使用 `user_data/data` 目录中的数据。

### 使用较小的测试集运行 Hyperopt

使用 `--timerange` 参数来更改你想使用多少测试集数据。
例如，要使用一个月的数据，请在 hyperopt 调用中传入 `--timerange 20210101-20210201`（从 2021 年 1 月到 2021 年 2 月）。

完整命令：

```bash
freqtrade hyperopt --strategy <strategyname> --timerange 20210101-20210201
```

### 使用较小的搜索空间运行 Hyperopt

使用 `--spaces` 选项来限制 hyperopt 使用的搜索空间。
让 Hyperopt 优化所有内容通常是一个巨大的搜索空间。
通常，先只搜索初始入场算法可能更有意义。
或者你只是想为那个很棒的新策略优化 stoploss 或 roi 表。

合法值为：

* `all`：优化所有内容（包括自定义空间）
* `buy`：仅搜索新的买入策略
* `sell`：仅搜索新的卖出策略
* `enter`：仅搜索新的入场逻辑
* `exit`：仅搜索新的出场逻辑
* `roi`：仅优化策略的最小利润表
* `stoploss`：搜索最佳止损值
* `trailing`：搜索最佳追踪止损值
* `trades`：搜索最佳最大开仓数值
* `protection`：搜索最佳保护参数（请阅读[保护机制部分](#optimizing-protections)了解如何正确定义这些参数）
* `default`：除 `trailing`、`trades` 和 `protection` 之外的 `all`
* `custom_space_name`：策略中任何参数使用的自定义空间
* 以上任意值的空格分隔列表，例如 `--spaces roi stoploss`

未指定 `--space` 命令行选项时使用的默认 Hyperopt 搜索空间不包括 `trailing` 超空间。我们建议你在找到、验证并粘贴了其他超空间的最佳参数到自定义策略后，单独运行 `trailing` 超空间的优化。

## 理解 Hyperopt 结果

Hyperopt 完成后，你可以使用结果来更新你的策略。
假设 hyperopt 给出以下结果：

```
Best result:

    44/100:    135 trades. Avg profit  0.57%. Total profit  0.03871918 BTC (0.7722%). Avg duration 180.4 mins. Objective: 1.94367

    # Buy hyperspace params:
    buy_params = {
        'buy_adx': 44,
        'buy_rsi': 29,
        'buy_adx_enabled': False,
        'buy_rsi_enabled': True,
        'buy_trigger': 'bb_lower'
    }
```

你应该这样理解这个结果：

* 效果最好的买入触发条件是 `bb_lower`。
* 你不应该使用 ADX，因为 `'buy_adx_enabled': False`。
* 你应该**考虑**使用 RSI 指标（`'buy_rsi_enabled': True`），最佳值为 `29.0`（`'buy_rsi': 29.0`）

### 自动将参数应用到策略

使用可超参数优化的参数时，hyperopt 运行的结果将写入策略旁边的 json 文件中（因此对于 `MyAwesomeStrategy.py`，文件将是 `MyAwesomeStrategy.json`）。
使用 `hyperopt-show` 子命令时也会更新此文件，除非向这两个命令中的任何一个提供了 `--disable-param-export`。


你的策略类也可以显式包含这些结果。只需复制 hyperopt 结果块并将其粘贴到类级别，替换旧参数（如果有）。下次执行策略时将自动加载新参数。

将整个 hyperopt 结果转移到策略中看起来像这样：

```python
class MyAwesomeStrategy(IStrategy):
    # Buy hyperspace params:
    buy_params = {
        'buy_adx': 44,
        'buy_rsi': 29,
        'buy_adx_enabled': False,
        'buy_rsi_enabled': True,
        'buy_trigger': 'bb_lower'
    }
```

!!! Note
    配置文件中的值将覆盖参数文件级别的参数 - 两者都将覆盖策略中的参数。
    因此优先级为：config > 参数文件 > 策略 `*_params` > 参数默认值

### 理解 Hyperopt ROI 结果

如果你正在优化 ROI（即优化搜索空间包含 'all'、'default' 或 'roi'），你的结果将如下所示并包含一个 ROI 表：

```
Best result:

    44/100:    135 trades. Avg profit  0.57%. Total profit  0.03871918 BTC (0.7722%). Avg duration 180.4 mins. Objective: 1.94367

    # ROI table:
    minimal_roi = {
        0: 0.10674,
        21: 0.09158,
        78: 0.03634,
        118: 0
    }
```

要在回测和实盘/模拟交易中使用 Hyperopt 找到的最佳 ROI 表，请将其复制粘贴为自定义策略中 `minimal_roi` 属性的值：

```
    # Minimal ROI designed for the strategy.
    # This attribute will be overridden if the config file contains "minimal_roi"
    minimal_roi = {
        0: 0.10674,
        21: 0.09158,
        78: 0.03634,
        118: 0
    }
```

如注释中所述，你也可以将其用作配置文件中 `minimal_roi` 设置的值。

#### 默认 ROI 搜索空间

如果你正在优化 ROI，Freqtrade 会为你创建 'roi' 优化超空间 -- 它是 ROI 表组件的超空间。默认情况下，Freqtrade 生成的每个 ROI 表由 4 行（步）组成。Hyperopt 为 ROI 表实现了自适应范围，ROI 步骤中的值范围取决于所使用的时间框架。默认情况下，值在以下范围内变化（对于一些最常用的时间框架，值四舍五入到小数点后 3 位）：

| # step | 1m     |               | 5m       |             | 1h         |               | 1d           |               |
| ------ | ------ | ------------- | -------- | ----------- | ---------- | ------------- | ------------ | ------------- |
| 1      | 0      | 0.011...0.119 | 0        | 0.03...0.31 | 0          | 0.068...0.711 | 0            | 0.121...1.258 |
| 2      | 2...8  | 0.007...0.042 | 10...40  | 0.02...0.11 | 120...480  | 0.045...0.252 | 2880...11520 | 0.081...0.446 |
| 3      | 4...20 | 0.003...0.015 | 20...100 | 0.01...0.04 | 240...1200 | 0.022...0.091 | 5760...28800 | 0.040...0.162 |
| 4      | 6...44 | 0.0           | 30...220 | 0.0         | 360...2640 | 0.0           | 8640...63360 | 0.0           |

这些范围在大多数情况下应该足够了。步骤中的分钟数（ROI 字典键）根据使用的时间框架线性缩放。步骤中的 ROI 值（ROI 字典值）根据使用的时间框架对数缩放。

如果你的自定义 hyperopt 中有 `generate_roi_table()` 和 `roi_space()` 方法，请删除它们以使用 Freqtrade 默认生成的自适应 ROI 表和 ROI 超参数优化空间。

如果你需要 ROI 表的组件在其他范围内变化，请覆盖 `roi_space()` 方法。如果你需要不同结构的 ROI 表或不同数量的行（步），请覆盖 `generate_roi_table()` 和 `roi_space()` 方法并实现你自己的自定义方法。

这些方法的示例可以在[覆盖预定义空间部分](advanced-hyperopt.md#overriding-pre-defined-spaces)中找到。

!!! Note "缩小搜索空间"
    为了进一步限制搜索空间，小数被限制为 3 位小数（精度为 0.001）。这通常足够了，比这更精确的值通常会导致过拟合结果。不过你可以[覆盖预定义空间](advanced-hyperopt.md#overriding-pre-defined-spaces)来根据需要更改此设置。

### 理解 Hyperopt Stoploss 结果

如果你正在优化 stoploss 值（即优化搜索空间包含 'all'、'default' 或 'stoploss'），你的结果将如下所示并包含 stoploss：

```
Best result:

    44/100:    135 trades. Avg profit  0.57%. Total profit  0.03871918 BTC (0.7722%). Avg duration 180.4 mins. Objective: 1.94367

    # Buy hyperspace params:
    buy_params = {
        'buy_adx': 44,
        'buy_rsi': 29,
        'buy_adx_enabled': False,
        'buy_rsi_enabled': True,
        'buy_trigger': 'bb_lower'
    }

    stoploss: -0.27996
```

要在回测和实盘/模拟交易中使用 Hyperopt 找到的最佳 stoploss 值，请将其复制粘贴为自定义策略中 `stoploss` 属性的值：

``` python
    # Optimal stoploss designed for the strategy
    # This attribute will be overridden if the config file contains "stoploss"
    stoploss = -0.27996
```

如注释中所述，你也可以将其用作配置文件中 `stoploss` 设置的值。

#### 默认 Stoploss 搜索空间

如果你正在优化 stoploss 值，Freqtrade 会为你创建 'stoploss' 优化超空间。默认情况下，该超空间中的 stoploss 值在 -0.35...-0.02 范围内变化，这在大多数情况下足够了。

如果你的自定义 hyperopt 文件中有 `stoploss_space()` 方法，请删除它以使用 Freqtrade 默认生成的 Stoploss 超参数优化空间。

如果你需要 stoploss 值在超参数优化期间在其他范围内变化，请覆盖 `stoploss_space()` 方法并在其中定义所需的范围。此方法的示例可以在[覆盖预定义空间部分](advanced-hyperopt.md#overriding-pre-defined-spaces)中找到。

!!! Note "缩小搜索空间"
    为了进一步限制搜索空间，小数被限制为 3 位小数（精度为 0.001）。这通常足够了，比这更精确的值通常会导致过拟合结果。不过你可以[覆盖预定义空间](advanced-hyperopt.md#overriding-pre-defined-spaces)来根据需要更改此设置。

### 理解 Hyperopt 追踪止损结果

如果你正在优化追踪止损值（即优化搜索空间包含 'all' 或 'trailing'），你的结果将如下所示并包含追踪止损参数：

```
Best result:

    45/100:    606 trades. Avg profit  1.04%. Total profit  0.31555614 BTC ( 630.48%). Avg duration 150.3 mins. Objective: -1.10161

    # Trailing stop:
    trailing_stop = True
    trailing_stop_positive = 0.02001
    trailing_stop_positive_offset = 0.06038
    trailing_only_offset_is_reached = True
```

要在回测和实盘/模拟交易中使用 Hyperopt 找到的最佳追踪止损参数，请将它们复制粘贴为自定义策略中相应属性的值：

``` python
    # Trailing stop
    # These attributes will be overridden if the config file contains corresponding values.
    trailing_stop = True
    trailing_stop_positive = 0.02001
    trailing_stop_positive_offset = 0.06038
    trailing_only_offset_is_reached = True
```

如注释中所述，你也可以将其用作配置文件中相应设置的值。

#### 默认追踪止损搜索空间

如果你正在优化追踪止损值，Freqtrade 会为你创建 'trailing' 优化超空间。默认情况下，该超空间中的 `trailing_stop` 参数始终设置为 True，`trailing_only_offset_is_reached` 的值在 True 和 False 之间变化，`trailing_stop_positive` 和 `trailing_stop_positive_offset` 参数的值分别在 0.02...0.35 和 0.01...0.1 范围内变化，这在大多数情况下足够了。

如果你需要追踪止损参数的值在超参数优化期间在其他范围内变化，请覆盖 `trailing_space()` 方法并在其中定义所需的范围。此方法的示例可以在[覆盖预定义空间部分](advanced-hyperopt.md#overriding-pre-defined-spaces)中找到。

!!! Note "缩小搜索空间"
    为了进一步限制搜索空间，小数被限制为 3 位小数（精度为 0.001）。这通常足够了，比这更精确的值通常会导致过拟合结果。不过你可以[覆盖预定义空间](advanced-hyperopt.md#overriding-pre-defined-spaces)来根据需要更改此设置。

### 可复现的结果

最优参数的搜索从超参数空间中的一些（目前 30 个）随机组合开始，即随机 Hyperopt epoch。这些随机 epoch 在 Hyperopt 输出中的第一列用星号字符（`*`）标记。

这些随机值的初始状态（随机状态）由 `--random-state` 命令行选项的值控制。你可以将其设置为任意值以获得可复现的结果。

如果你没有在命令行选项中显式设置此值，Hyperopt 会为你使用某个随机值作为随机状态种子。每次 Hyperopt 运行的随机状态值都会显示在日志中，因此你可以将其复制粘贴到 `--random-state` 命令行选项中，以重复使用的初始随机 epoch 集合。

如果你没有更改命令行选项、配置、时间范围、策略和 Hyperopt 类、历史数据和损失函数中的任何内容 -- 使用相同的随机状态值应该能获得相同的超参数优化结果。

## 输出格式

默认情况下，hyperopt 打印彩色结果 -- 利润为正的 epoch 以绿色打印。这种高亮有助于你找到值得进一步分析的 epoch。总利润为零或利润为负（亏损）的 epoch 以正常颜色打印。如果你不需要结果着色（例如，当你将 hyperopt 输出重定向到文件时），你可以通过在命令行中指定 `--no-color` 选项来关闭着色。

如果你想查看 hyperopt 输出中的所有结果，而不仅仅是最佳结果，可以使用 `--print-all` 命令行选项。使用 `--print-all` 时，当前最佳结果默认也会被着色 -- 以粗体（高亮）样式打印。这也可以通过 `--no-color` 命令行选项关闭。

!!! Note "Windows 和彩色输出"
    Windows 原生不支持彩色输出，因此会自动禁用。要在 Windows 下运行 hyperopt 时获得彩色输出，请考虑使用 WSL。

## 仓位叠加和禁用最大市场持仓

在某些情况下，你可能需要使用 `--eps`/`--enable-position-staking` 参数运行 Hyperopt（和回测），或者你可能需要将 `max_open_trades` 设置为一个非常大的数字以禁用开仓数量限制。

默认情况下，hyperopt 模拟 Freqtrade 实盘/模拟运行的行为，每个交易对只允许一个开仓交易。所有交易对的总开仓交易数也受 `max_open_trades` 设置的限制。在 Hyperopt/回测期间，这可能导致潜在交易被已有的开仓交易隐藏（或遮蔽）。

`--eps`/`--enable-position-stacking` 参数允许模拟多次购买同一交易对。
使用 `--max-open-trades` 设置一个非常大的数字将禁用开仓数量限制。

!!! Note
    模拟/实盘运行**不会**使用仓位叠加 - 因此在没有仓位叠加的情况下验证策略是有意义的，因为这更接近现实。

你也可以通过在配置文件中显式设置 `"position_stacking"=true` 来启用仓位叠加。

## 内存不足错误

由于 hyperopt 消耗大量内存（完整数据需要在每个并行回测进程中加载到内存中），你很可能会遇到"内存不足"错误。
要解决这些问题，你有多种选择：

* 减少交易对数量。
* 缩短使用的时间范围（`--timerange <timerange>`）。
* 避免使用 `--timeframe-detail`（这会加载大量额外数据到内存中）。
* 减少并行进程数（`-j <n>`）。
* 增加机器的内存。
* 如果你使用了大量带有 `.range` 功能的参数，请使用 `--analyze-per-epoch`。


## 此点已被之前评估过

如果你看到 `The objective has been evaluated at this point before.` - 那么这表明你的空间已经耗尽，或接近耗尽。
基本上你空间中的所有点都已被命中（或命中了局部最小值） - hyperopt 不再能在多维空间中找到尚未尝试的点。
Freqtrade 在这种情况下尝试通过使用新的随机点来对抗"局部最小值"问题。

示例：

``` python
buy_ema_short = IntParameter(5, 20, default=10, space="buy", optimize=True)
# This is the only parameter in the buy space
```

`buy_ema_short` 空间有 15 个可能的值（`5, 6, ... 19, 20`）。如果你现在对 buy 空间运行 hyperopt，hyperopt 在用完选项之前只有 15 个值可以尝试。
因此你的 epoch 数应该与可能的值对齐 - 或者当你注意到大量 `The objective has been evaluated at this point before.` 警告时，应该准备好中断运行。

## 显示 Hyperopt 结果详情

在运行所需数量的 epoch 后，你可以稍后列出所有结果进行分析，仅选择最佳或盈利的结果，并显示之前评估的任何 epoch 的详情。这可以通过 `hyperopt-list` 和 `hyperopt-show` 子命令完成。这些子命令的用法在 [Utils](utils.md#list-hyperopt-results) 章节中描述。

## 从策略输出调试信息

如果你想从策略输出调试信息，可以使用 `logging` 模块。默认情况下，Freqtrade 会输出所有 `INFO` 级别或更高级别的消息。


``` python
import logging


logger = logging.getLogger(__name__)


class MyAwesomeStrategy(IStrategy):
    ...

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        logger.info("This is a debug message")
        ...

```

!!! Note "使用 print"
    通过 `print()` 打印的消息不会显示在 hyperopt 输出中，除非禁用了并行处理（`-j 1`）。
    建议使用 `logging` 模块代替。

## 验证回测结果

一旦优化后的策略已实现到你的策略中，你应该回测此策略以确保一切按预期工作。

要获得与 Hyperopt 期间相同的结果（交易数量、持续时间、利润等），请使用与 hyperopt 相同的配置和参数（时间范围、时间框架等）进行回测。

### 为什么我的回测结果与 hyperopt 结果不匹配？

如果结果不匹配，请检查以下因素：

* 你可能在 `populate_indicators()` 中添加了 hyperopt 参数，在那里它们只会**为所有 epoch**计算一次。例如，如果你试图优化多个 SMA 时间周期值，可超参数优化的时间周期参数应该放在每个 epoch 都会计算的 `populate_entry_trend()` 中。参见[优化指标参数](https://www.freqtrade.io/en/stable/hyperopt/#optimizing-an-indicator-parameter)。
* 如果你禁用了 hyperopt 参数自动导出到 JSON 参数文件的功能，请仔细检查确保你正确地将所有超参数优化的值转移到了策略中。
* 检查日志以验证正在设置哪些参数以及使用了什么值。
* 特别注意 stoploss、max_open_trades 和 trailing stoploss 参数，因为这些通常在配置文件中设置，会覆盖策略中的更改。检查回测日志以确保没有参数被配置文件意外设置（如 `stoploss`、`max_open_trades` 或 `trailing_stop`）。
* 验证你没有意外的参数 JSON 文件覆盖了策略中的参数或默认 hyperopt 设置。
* 验证回测中启用的任何保护机制在超参数优化时也已启用，反之亦然。使用 `--space protection` 时，超参数优化会自动启用保护机制。
