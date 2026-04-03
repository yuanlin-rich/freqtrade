# Freqtrade 策略 101：策略开发快速入门

本快速入门假设您已熟悉交易基础知识，并且已经阅读了 [Freqtrade 基础知识](bot-basics.md)页面。

## 必备知识

Freqtrade 中的策略是一个 Python 类，它定义了买入和卖出加密货币`资产`的逻辑。

资产被定义为`交易对`，由`币种`和`计价货币`组成。币种是您用另一种货币（计价货币）进行交易的资产。

数据由交易所以`K 线`的形式提供，每根 K 线由六个值组成：`date`（日期）、`open`（开盘价）、`high`（最高价）、`low`（最低价）、`close`（收盘价）和 `volume`（成交量）。

`技术分析`函数使用各种计算和统计公式分析 K 线数据，并生成称为`指标`的衍生值。

指标在资产交易对的 K 线上进行分析，以生成`信号`。

信号在加密货币`交易所`上转化为`订单`，即`交易`。

我们使用`入场`和`出场`这两个术语来替代`买入`和`卖出`，因为 Freqtrade 同时支持`做多`和`做空`交易。

- **做多（long）**: 您用计价货币买入币种，例如用 USDT 作为计价货币买入 BTC，然后以高于买入价的价格卖出获利。在做多交易中，利润来自币种价值相对于计价货币的上涨。
- **做空（short）**: 您从交易所借入币种形式的资本，之后以计价货币偿还币种价值。在做空交易中，利润来自币种价值相对于计价货币的下跌（您以较低的价格偿还借款）。

虽然 Freqtrade 在某些交易所支持现货和合约市场，但为简单起见，我们将仅关注现货（做多）交易。

## 基本策略结构

### 主数据框

Freqtrade 策略使用一种由行和列组成的表格数据结构，称为 `dataframe`（数据框），来生成入场和出场交易的信号。

您配置的交易对列表中的每个交易对都有自己的数据框。数据框以 `date` 列作为索引，例如 `2024-06-31 12:00`。

接下来的 5 列代表 `open`（开盘价）、`high`（最高价）、`low`（最低价）、`close`（收盘价）和 `volume`（成交量）（OHLCV）数据。

### 填充指标值

`populate_indicators` 函数向数据框中添加代表技术分析指标值的列。

常见指标的示例包括相对强弱指数（RSI）、布林带（Bollinger Bands）、资金流量指数（MFI）、移动平均线（MA）和平均真实波幅（ATR）。

通过调用技术分析函数（如 ta-lib 的 RSI 函数 `ta.RSI()`）并将其赋值给一个列名（如 `rsi`），即可向数据框添加列。

```python
dataframe['rsi'] = ta.RSI(dataframe)
```

??? Hint "技术分析库"
    不同的库以不同的方式生成指标值。请查阅每个库的文档以了解如何将其集成到您的策略中。您也可以查看 [Freqtrade 示例策略](https://github.com/freqtrade/freqtrade-strategies)来获取灵感。

### 填充入场信号

`populate_entry_trend` 函数定义入场信号的条件。

数据框中会添加 `enter_long` 列，当该列的值为 `1` 时，Freqtrade 识别到一个入场信号。

??? Hint "做空"
    要进入做空交易，请使用 `enter_short` 列。

### 填充出场信号

`populate_exit_trend` 函数定义出场信号的条件。

数据框中会添加 `exit_long` 列，当该列的值为 `1` 时，Freqtrade 识别到一个出场信号。

??? Hint "做空"
    要退出做空交易，请使用 `exit_short` 列。

## 一个简单的策略

以下是一个最简的 Freqtrade 策略示例：

```python
from freqtrade.strategy import IStrategy
from pandas import DataFrame
import talib.abstract as ta

class MyStrategy(IStrategy):

    timeframe = '15m'

    # set the initial stoploss to -10%
    stoploss = -0.10

    # exit profitable positions at any time when the profit is greater than 1%
    minimal_roi = {"0": 0.01}

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # generate values for technical analysis indicators
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)

        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # generate entry signals based on indicator values
        dataframe.loc[
            (dataframe['rsi'] < 30),
            'enter_long'] = 1

        return dataframe

    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # generate exit signals based on indicator values
        dataframe.loc[
            (dataframe['rsi'] > 70),
            'exit_long'] = 1

        return dataframe
```

## 执行交易

当发现信号（入场或出场列中的值为 `1`）时，Freqtrade 将尝试下单，即创建一笔`交易`或`持仓`。

每个新交易持仓会占用一个`槽位`。槽位代表可以同时开启的最大新交易数量。

槽位数量由 `max_open_trades` [配置](configuration.md)选项定义。

但是，在很多情况下，生成信号并不总是会创建交易订单。这些情况包括：

- 没有足够的剩余资金来买入资产，或钱包中没有足够的资金来卖出资产（包括手续费）
- 没有足够的空闲槽位来开新交易（您的持仓数量已经等于 `max_open_trades` 选项值）
- 该交易对已有一个未平仓交易（Freqtrade 不能叠加持仓——但可以[调整现有仓位](strategy-callbacks.md#adjust-trade-position)）
- 如果入场和出场信号出现在同一根 K 线上，它们被视为[冲突信号](strategy-customization.md#colliding-signals)，不会生成订单
- 策略通过使用相关的[入场](strategy-callbacks.md#trade-entry-buy-order-confirmation)或[出场](strategy-callbacks.md#trade-exit-sell-order-confirmation)回调函数中的逻辑主动拒绝了交易订单

请阅读[策略自定义](strategy-customization.md)文档以了解更多详情。

## 回测和前向测试

策略开发可能是一个漫长而令人沮丧的过程，因为将我们人类的"直觉"转化为可运行的计算机控制（"算法"）策略并不总是那么简单。

因此，策略应该经过测试以验证它是否能按预期工作。

Freqtrade 有两种测试模式：

- **回测（backtesting）**: 使用您从交易所[下载的历史数据](data-download.md)，回测是评估策略表现的快速方法。但是，很容易使结果失真，使策略看起来比实际情况更有利可图。请查看[回测文档](backtesting.md)获取更多信息。
- **模拟运行（dry run）**: 通常被称为_前向测试_，模拟运行使用来自交易所的实时数据。但是，任何会产生交易的信号都会被 Freqtrade 正常跟踪，只是不会在交易所上真正开仓。前向测试是实时运行的，虽然获取结果需要更长时间，但它比回测能更可靠地指示**潜在**的表现。

通过在[配置](configuration.md#using-dry-run-mode)中将 `dry_run` 设置为 true 来启用模拟运行。

!!! Warning "回测结果可能非常不准确"
    回测结果可能与实际情况不符的原因有很多。请查看[回测假设](backtesting.md#assumptions-made-by-backtesting)和[常见策略错误](strategy-customization.md#common-mistakes-when-developing-strategies)文档。
    一些列出和排名 Freqtrade 策略的网站展示了令人印象深刻的回测结果。不要假设这些结果是可实现的或现实的。

??? Hint "有用的命令"
    Freqtrade 包含两个用于检查策略基本缺陷的有用命令：[前瞻分析](lookahead-analysis.md)和[递归分析](recursive-analysis.md)。

### 评估回测和模拟运行结果

在回测策略之后，请始终进行模拟运行，以查看回测和模拟运行结果是否足够相似。

如果存在显著差异，请验证您的入场和出场信号是否一致，是否出现在两种模式下的相同 K 线上。但是，模拟运行和回测之间总会存在差异：

- 回测假设所有订单都会成交。在模拟运行中，如果使用限价单或交易所上没有成交量，情况可能并非如此。
- 在 K 线收盘后发现入场信号时，回测假设交易以下一根 K 线的开盘价入场（除非您的策略中有自定义定价回调）。在模拟运行中，信号和交易开仓之间通常会有延迟。
  这是因为当主时间周期上出现新 K 线时（例如每 5 分钟），Freqtrade 需要时间来分析所有交易对的数据框。因此，Freqtrade 会在 K 线开盘后几秒（理想情况下延迟尽可能小）尝试开仓。
- 由于模拟运行中的入场价格可能与回测不匹配，这意味着利润计算也会不同。因此，如果 ROI、止损、追踪止损和回调出场不完全相同是正常的。
- 新 K 线到来与信号生成和交易开仓之间的计算"延迟"越大，价格的不可预测性就越大。请确保您的计算机足够强大，能在合理时间内处理交易对列表中所有交易对的数据。如果存在显著的数据处理延迟，Freqtrade 会在日志中发出警告。

## 控制或监控运行中的机器人

一旦您的机器人以模拟或实盘模式运行，Freqtrade 有六种机制来控制或监控运行中的机器人：

- **[FreqUI](freq-ui.md)**: 最容易上手，FreqUI 是一个 Web 界面，用于查看和控制机器人的当前活动。
- **[Telegram](telegram-usage.md)**: 在移动设备上，可以使用 Telegram 集成来获取机器人活动的警报并控制某些方面。
- **[FTUI](https://github.com/freqtrade/ftui)**: FTUI 是 Freqtrade 的终端（命令行）界面，仅允许监控运行中的机器人。
- **[freqtrade-client](rest-api.md#consuming-the-api)**: REST API 的 Python 实现，使您能轻松地从 Python 应用程序或命令行发出请求并处理机器人的响应。
- **[REST API 端点](rest-api.md#available-endpoints)**: REST API 允许程序员开发自己的工具来与 Freqtrade 机器人交互。
- **[Webhooks](webhook-config.md)**: Freqtrade 可以通过 webhook 向其他服务（如 Discord）发送信息。

### 日志

Freqtrade 会生成大量的调试日志，帮助您了解正在发生什么。请熟悉机器人日志中可能出现的信息和错误消息。

默认情况下，日志输出到标准输出（命令行）。如果您想要写入文件，许多 freqtrade 命令（包括 `trade` 命令）都接受 `--logfile` 选项以写入文件。

查看 [FAQ](faq.md#how-do-i-search-the-bot-logs-for-something) 获取示例。

## 最后的思考

算法交易是困难的，大多数公开策略由于需要大量时间和精力才能使策略在多种场景下盈利，因此表现不佳。

因此，使用公开策略并通过回测来评估表现通常是有问题的。但是，Freqtrade 提供了有用的方法来帮助您做出决策和进行尽职调查。

实现盈利有很多不同的方法，没有任何一个单一的技巧、诀窍或配置选项能修复表现不佳的策略。

Freqtrade 是一个拥有庞大且乐于助人的社区的开源平台——请务必访问我们的 [Discord 频道](https://discord.gg/p7nuUNVfP7)与他人讨论您的策略！

一如既往，只投入您愿意承受损失的资金。

## 结论

在 Freqtrade 中开发策略涉及基于技术指标定义入场和出场信号。遵循上述结构和方法，您可以创建和测试自己的交易策略。

常见问题和解答可在我们的 [FAQ](faq.md) 中找到。

要继续学习，请参阅更深入的 [Freqtrade 策略自定义文档](strategy-customization.md)。
