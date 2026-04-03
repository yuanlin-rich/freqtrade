# 高级回测分析

## 分析买入/入场和卖出/出场标签

了解策略根据用于标记不同买入条件的买入/入场标签的表现是很有帮助的。您可能希望查看比默认回测输出更复杂的关于每个买入和卖出条件的统计数据。您可能还希望确定导致交易开仓的信号 K 线上的指标值。

!!! Note
    以下买入原因分析仅适用于回测，*不适用于超参数优化*。

我们需要将 `--export` 选项设置为 `signals` 来运行回测，以启用信号**和**交易的导出：

``` bash
freqtrade backtesting -c <config.json> --timeframe <tf> --strategy <strategy_name> --timerange=<timerange> --export=signals
```

这将告诉 freqtrade 输出一个包含策略、交易对及其对应的导致入场和出场信号的 K 线 DataFrame 的 pickle 字典。
根据您的策略产生的入场次数，此文件可能会变得相当大，因此请定期检查您的 `user_data/backtest_results` 文件夹以删除旧的导出文件。

在运行下一次回测之前，请确保删除旧的回测结果或使用 `--cache none` 选项运行回测，以确保不使用缓存结果。

如果一切顺利，您现在应该在 `user_data/backtest_results` 文件夹中看到 `backtest-result-{timestamp}_signals.pkl` 和 `backtest-result-{timestamp}_exited.pkl` 文件。

要分析入场/出场标签，我们现在需要使用 `freqtrade backtesting-analysis` 命令，并使用 `--analysis-groups` 选项提供以空格分隔的参数：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 1 2 3 4 5
```

此命令将读取最近的回测结果。`--analysis-groups` 选项用于指定各种表格输出，显示每个分组或交易的利润，从最简单的（0）到最详细的按交易对、买入标签和卖出标签（4）：

* 0：按 enter_tag 统计的总体胜率和利润摘要
* 1：按 enter_tag 分组的利润摘要
* 2：按 enter_tag 和 exit_tag 分组的利润摘要
* 3：按交易对和 enter_tag 分组的利润摘要
* 4：按交易对、enter_tag 和 exit_tag 分组的利润摘要（可能会很大）
* 5：按 exit_tag 分组的利润摘要

通过使用 `-h` 选项运行可查看更多选项。

### 使用 backtest-filename

默认情况下，`backtesting-analysis` 处理 `user_data/backtest_results` 目录中最近的回测结果。
如果您想分析之前的回测结果，请使用 `--backtest-filename` 选项指定所需的文件。这使您可以随时通过提供相关回测结果的文件名来回顾和重新分析历史回测输出：

``` bash
freqtrade backtesting -c <config.json> --strategy <strategy_name> --timerange <timerange> --export signals --backtest-filename backtest-result-2025-03-05_20-38-34.zip
```

您应该在日志中看到类似以下的输出，其中包含已导出的带时间戳的文件名：

```
2022-06-14 16:28:32,698 - freqtrade.misc - INFO - dumping json to "mystrat_backtest-2022-06-14_16-28-32.json"
```

然后您可以在 `backtesting-analysis` 中使用该文件名：

``` bash
freqtrade backtesting-analysis -c <config.json> --backtest-filename=backtest-result-2025-03-05_20-38-34.zip
```

要使用不同结果目录中的结果，您可以使用 `--backtest-directory` 指定目录

``` bash
freqtrade backtesting-analysis -c <config.json> --backtest-directory custom_results/ --backtest-filename backtest-result-2025-03-05_20-38-34.zip
```

### 调整显示的买入标签和卖出标签

要在显示的输出中仅显示某些买入和卖出标签，请使用以下两个选项：

```
--enter-reason-list : Space-separated list of enter signals to analyse. Default: "all"
--exit-reason-list : Space-separated list of exit signals to analyse. Default: "all"
```

例如：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 2 --enter-reason-list enter_tag_a enter_tag_b --exit-reason-list roi custom_exit_tag_a stop_loss
```

### 输出信号 K 线的指标值

`freqtrade backtesting-analysis` 的真正强大之处在于能够打印信号 K 线上的指标值，从而可以对买入信号指标进行细粒度的调查和调优。要打印给定指标集的列，请使用 `--indicator-list` 选项：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 2 --enter-reason-list enter_tag_a enter_tag_b --exit-reason-list roi custom_exit_tag_a stop_loss --indicator-list rsi rsi_1h bb_lowerband ema_9 macd macdsignal
```

指标必须存在于您策略的主 DataFrame 中（无论是主时间周期还是信息时间周期），否则它们将在脚本输出中被忽略。

!!! Note "指标列表"
    指标值将同时显示入场点和出场点。如果指定了 `--indicator-list all`，则仅显示入场点的指标，以避免过大的列表（具体取决于策略）。

分析中包含了一系列 K 线和交易相关的字段，因此可以通过将它们包含在 indicator-list 中自动访问，这些字段包括：

* **open_date     :** 交易开仓日期时间
* **close_date    :** 交易平仓日期时间
* **min_rate      :** 持仓期间的最低价格
* **max_rate      :** 持仓期间的最高价格
* **open          :** 信号 K 线开盘价
* **close         :** 信号 K 线收盘价
* **high          :** 信号 K 线最高价
* **low           :** 信号 K 线最低价
* **volume        :** 信号 K 线成交量
* **profit_ratio  :** 交易利润率
* **profit_abs    :** 交易的绝对利润

#### 指标值的示例输出

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen
```

在此示例中，我们旨在显示交易入场和出场点的 `chikou_span` 和 `tenkan_sen` 指标值。

指标的示例输出可能如下所示：

| pair      | open_date                 | enter_reason | exit_reason | chikou_span (entry) | tenkan_sen (entry) | chikou_span (exit) | tenkan_sen (exit) |
|-----------|---------------------------|--------------|-------------|---------------------|--------------------|--------------------|-------------------|
| DOGE/USDT | 2024-07-06 00:35:00+00:00 |              | exit_signal | 0.105               | 0.106              | 0.105              | 0.107             |
| BTC/USDT  | 2024-08-05 14:20:00+00:00 |              | roi         | 54643.440           | 51696.400          | 54386.000          | 52072.010         |

如表所示，`chikou_span (entry)` 表示交易入场时的指标值，而 `chikou_span (exit)` 反映出场时的值。
这种指标值的详细视图增强了分析效果。

`(entry)` 和 `(exit)` 后缀被添加到指标名称中，以区分交易入场和出场点的值。

!!! Note "交易级指标"
    某些交易级指标没有 `(entry)` 或 `(exit)` 后缀。这些指标包括：`pair`、`stake_amount`、
    `max_stake_amount`、`amount`、`open_date`、`close_date`、`open_rate`、`close_rate`、`fee_open`、`fee_close`、`trade_duration`、
    `profit_ratio`、`profit_abs`、`exit_reason`、`initial_stop_loss_abs`、`initial_stop_loss_ratio`、`stop_loss_abs`、`stop_loss_ratio`、
    `min_rate`、`max_rate`、`is_open`、`enter_tag`、`leverage`、`is_short`、`open_timestamp`、`close_timestamp` 和 `orders`

#### 根据入场或出场信号过滤指标

`--indicator-list` 选项默认显示入场和出场信号的指标值。要仅过滤入场信号的指标值，可以使用 `--entry-only` 参数。类似地，要仅显示出场信号的指标值，请使用 `--exit-only` 参数。

示例：显示入场信号的指标值：

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen --entry-only
```

示例：显示出场信号的指标值：

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen --exit-only
```

!!! note
    使用这些过滤器时，指标名称不会添加 `(entry)` 或 `(exit)` 后缀。

### 按日期过滤交易输出

要仅显示回测时间范围内特定日期之间的交易，请以 `YYYYMMDD-[YYYYMMDD]` 格式提供常用的 `timerange` 选项：

```
--timerange : Timerange to filter output trades, start date inclusive, end date exclusive. e.g. 20220101-20221231
```

例如，如果您的回测时间范围是 `20220101-20221231`，但您只想输出一月份的交易：

``` bash
freqtrade backtesting-analysis -c <config.json> --timerange 20220101-20220201
```

### 打印被拒绝的信号

使用 `--rejected-signals` 选项打印被拒绝的信号。

``` bash
freqtrade backtesting-analysis -c <config.json> --rejected-signals
```

### 将表格写入 CSV

某些表格输出可能很大，因此将它们打印到终端并不理想。
使用 `--analysis-to-csv` 选项禁止将表格打印到标准输出，改为将它们写入 CSV 文件。

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv
```

默认情况下，这将为您在 `backtesting-analysis` 命令中指定的每个输出表写入一个文件，例如

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv --rejected-signals --analysis-groups 0 1
```

这将写入到 `user_data/backtest_results`：

* rejected_signals.csv
* group_0.csv
* group_1.csv

要覆盖文件写入的位置，还需指定 `--analysis-csv-path` 选项。

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv --analysis-csv-path another/data/path/
```
