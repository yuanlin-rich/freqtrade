# 策略自定义

本页面解释如何自定义您的策略、添加新指标和设置交易规则。

如果您还没有阅读过，请先熟悉：

- [Freqtrade 策略 101](strategy-101.md)，提供策略开发的快速入门
- [Freqtrade 机器人基础](bot-basics.md)，提供机器人运作方式的整体信息

## 开发您自己的策略

机器人包含一个默认策略文件。

此外，[策略仓库](https://github.com/freqtrade/freqtrade-strategies)中还有其他几个策略可用。

然而，您很可能有自己的策略想法。

本文档旨在帮助您将想法转化为可运行的策略。

### 生成策略模板

要开始，您可以使用以下命令：

```bash
freqtrade new-strategy --strategy AwesomeStrategy
```

这将从模板创建一个名为 `AwesomeStrategy` 的新策略，文件位置为 `user_data/strategies/AwesomeStrategy.py`。

!!! Note
    策略的*名称*和文件名之间是有区别的。在大多数命令中，Freqtrade 使用策略的*名称*，*而不是文件名*。

!!! Note
    `new-strategy` 命令生成的起始示例不会立即盈利。

??? Hint "不同的模板级别"
    `freqtrade new-strategy` 有一个额外的参数 `--template`，它控制您在创建的策略中获得的预构建信息量。使用 `--template minimal` 获得一个没有任何指标示例的空策略，或使用 `--template advanced` 获得一个定义了更复杂功能的模板。

### 策略的结构

策略文件包含构建策略逻辑所需的所有信息：

- OHLCV 格式的蜡烛图数据
- 指标
- 入场逻辑
  - 信号
- 出场逻辑
  - 信号
  - 最小 ROI
  - 回调函数（"自定义函数"）
- 止损
  - 固定/绝对止损
  - 追踪止损
  - 回调函数（"自定义函数"）
- 定价 [可选]
- 仓位调整 [可选]

机器人包含一个名为 `SampleStrategy` 的示例策略，您可以将其作为基础：`user_data/strategies/sample_strategy.py`。
您可以使用参数 `--strategy SampleStrategy` 进行测试。请记住，您使用的是策略类名，而不是文件名。

此外，还有一个名为 `INTERFACE_VERSION` 的属性，它定义了机器人应使用的策略接口版本。
当前版本是 3 - 这也是未明确在策略中设置时的默认值。

您可能会看到较旧的策略设置为接口版本 2，这些需要更新到 v3 术语，因为未来版本将要求设置此项。

使用 `trade` 命令在干运行或实盘模式下启动机器人：

```bash
freqtrade trade --strategy AwesomeStrategy
```

### 机器人模式

Freqtrade 策略可以由 Freqtrade 机器人在 5 种主要模式下处理：

- backtesting（回测）
- hyperopting（超参数优化）
- dry（"前向测试"）
- live（实盘）
- FreqAI（此处不涉及）

查看[配置文档](configuration.md)了解如何将机器人设置为干运行或实盘模式。

**在测试时始终使用干运行模式，因为这可以让您了解策略在现实中的表现，而不会冒资金风险。**

## 深入了解

**对于以下部分，我们将使用 [user_data/strategies/sample_strategy.py](https://github.com/freqtrade/freqtrade/blob/develop/freqtrade/templates/sample_strategy.py)
文件作为参考。**

!!! Note "策略和回测"
    为避免回测和干运行/实盘模式之间的问题和意外差异，请注意
    在回测期间，完整的时间范围会一次性传递给 `populate_*()` 方法。
    因此，最好使用向量化操作（跨整个数据框，而不是循环）并
    避免索引引用（`df.iloc[-1]`），而是使用 `df.shift()` 来获取前一根蜡烛。

!!! Warning "警告：使用未来数据"
    由于回测将完整的时间范围传递给 `populate_*()` 方法，策略作者
    需要注意避免策略使用未来的数据。
    本文档的[常见错误](#common-mistakes-when-developing-strategies)部分列出了一些常见模式。

??? Hint "前瞻和递归分析"
    Freqtrade 包含两个有用的命令，可帮助评估常见的前瞻（使用未来数据）和
    递归偏差（指标值的方差）问题。在干运行或实盘模式下运行策略之前，
    您应该始终先使用这些命令。请查看相关文档以了解
    [前瞻](lookahead-analysis.md)和[递归](recursive-analysis.md)分析。

### 数据框

Freqtrade 使用 [pandas](https://pandas.pydata.org/) 来存储/提供蜡烛图（OHLCV）数据。
Pandas 是一个为处理表格格式的大量数据而开发的优秀库。

数据框中的每一行对应图表上的一根蜡烛，最新的完整蜡烛始终是数据框中的最后一行（按日期排序）。

如果我们使用 pandas 的 `head()` 函数查看主数据框的前几行，我们会看到：

```output
> dataframe.head()
                       date      open      high       low     close     volume
0 2021-11-09 23:25:00+00:00  67279.67  67321.84  67255.01  67300.97   44.62253
1 2021-11-09 23:30:00+00:00  67300.97  67301.34  67183.03  67187.01   61.38076
2 2021-11-09 23:35:00+00:00  67187.02  67187.02  67031.93  67123.81  113.42728
3 2021-11-09 23:40:00+00:00  67123.80  67222.40  67080.33  67160.48   78.96008
4 2021-11-09 23:45:00+00:00  67160.48  67160.48  66901.26  66943.37  111.39292
```

数据框是一个表格，其中列不是单个值，而是一系列数据值。因此，像下面这样的简单 python 比较将不起作用：

``` python
    if dataframe['rsi'] > 30:
        dataframe['enter_long'] = 1
```

上述部分将失败并显示 `The truth value of a Series is ambiguous [...]`。

这必须以 pandas 兼容的方式编写，以便在整个数据框上执行操作，即`向量化`。

``` python
    dataframe.loc[
        (dataframe['rsi'] > 30)
    , 'enter_long'] = 1
```

通过这部分，您在数据框中有了一个新列，每当 RSI 高于 30 时，该列就会被赋值为 `1`。

Freqtrade 使用这个新列作为入场信号，假设交易将在下一根开盘蜡烛上开仓。

Pandas 提供了快速计算指标的方法，即"向量化"。为了从这种速度中受益，建议不要使用循环，而是使用向量化方法。

向量化操作在整个数据范围内执行计算，因此与循环遍历每一行相比，在计算指标时要快得多。

??? Hint "信号 vs 交易"
    - 信号是在蜡烛收盘时从指标生成的，是进入交易的意图。
    - 交易是执行的订单（在实盘模式下在交易所上），然后交易将尽可能接近下一根蜡烛开盘时开仓。

!!! Warning "交易订单假设"
    在回测中，信号在蜡烛收盘时生成。然后交易立即在下一根蜡烛开盘时启动。

    在干运行和实盘中，这可能会延迟，因为需要首先分析所有交易对的数据框，然后对每个交易对进行交易处理。这意味着在干运行/实盘中，您需要注意尽可能降低计算延迟，通常通过运行少量交易对并使用具有良好时钟速度的 CPU。

#### 为什么我看不到"实时"蜡烛数据？

Freqtrade 不会在数据框中存储不完整/未完成的蜡烛。

使用不完整的数据进行策略决策称为"重绘"，您可能会看到其他平台允许这样做。

Freqtrade 不允许。数据框中只有完整/完成的蜡烛数据可用。

### 自定义指标

入场和出场信号需要指标。您可以通过扩展策略文件中 `populate_indicators()` 方法中包含的列表来添加更多指标。

您应该只添加在 `populate_entry_trend()`、`populate_exit_trend()` 中使用的指标，或用于填充另一个指标的指标，否则性能可能会受到影响。

重要的是始终从这三个函数返回数据框，而不删除/修改列 `"open", "high", "low", "close", "volume"`，否则这些字段将包含意外内容。

示例：

```python
def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    """
    向给定的 DataFrame 添加几个不同的 TA 指标

    性能说明：为了获得最佳性能，请节约使用指标的数量
    只取消注释您在策略或超参数优化配置中使用的指标，
    否则您将浪费内存和 CPU 使用率。
    :param dataframe: 来自交易所的数据的 Dataframe
    :param metadata: 附加信息，如当前交易的交易对
    :return: 包含策略所有必需指标的 Dataframe
    """
    dataframe['sar'] = ta.SAR(dataframe)
    dataframe['adx'] = ta.ADX(dataframe)
    stoch = ta.STOCHF(dataframe)
    dataframe['fastd'] = stoch['fastd']
    dataframe['fastk'] = stoch['fastk']
    dataframe['bb_lower'] = ta.BBANDS(dataframe, nbdevup=2, nbdevdn=2)['lowerband']
    dataframe['sma'] = ta.SMA(dataframe, timeperiod=40)
    dataframe['tema'] = ta.TEMA(dataframe, timeperiod=9)
    dataframe['mfi'] = ta.MFI(dataframe)
    dataframe['rsi'] = ta.RSI(dataframe)
    dataframe['ema5'] = ta.EMA(dataframe, timeperiod=5)
    dataframe['ema10'] = ta.EMA(dataframe, timeperiod=10)
    dataframe['ema50'] = ta.EMA(dataframe, timeperiod=50)
    dataframe['ema100'] = ta.EMA(dataframe, timeperiod=100)
    dataframe['ao'] = awesome_oscillator(dataframe)
    macd = ta.MACD(dataframe)
    dataframe['macd'] = macd['macd']
    dataframe['macdsignal'] = macd['macdsignal']
    dataframe['macdhist'] = macd['macdhist']
    hilbert = ta.HT_SINE(dataframe)
    dataframe['htsine'] = hilbert['sine']
    dataframe['htleadsine'] = hilbert['leadsine']
    dataframe['plus_dm'] = ta.PLUS_DM(dataframe)
    dataframe['plus_di'] = ta.PLUS_DI(dataframe)
    dataframe['minus_dm'] = ta.MINUS_DM(dataframe)
    dataframe['minus_di'] = ta.MINUS_DI(dataframe)
    return dataframe
```

