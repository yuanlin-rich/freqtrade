# 绘图

本页面介绍如何绘制价格、指标和利润图表。

!!! Warning "已弃用"
    本页面中描述的命令（`plot-dataframe`、`plot-profit`）应被视为已弃用，处于维护模式。
    这主要是因为即使是中等大小的图表也可能导致性能问题，同时也因为"存储文件并在浏览器中打开"从 UI 角度来看不太直观。

    虽然目前没有立即删除它们的计划，但它们并没有被积极维护——如果需要重大更改才能保持其正常工作，可能会在短期内被移除。

    请使用 [FreqUI](freq-ui.md) 满足绘图需求，它不会遇到相同的性能问题。

## 安装 / 设置

绘图模块使用 Plotly 库。您可以通过运行以下命令来安装/升级：

``` bash
pip install -U -r requirements-plot.txt
```

## 绘制价格和指标

`freqtrade plot-dataframe` 子命令显示一个交互式图表，包含三个子图：

* 主图，包含 K 线和跟随价格的指标（sma/ema）
* 成交量柱状图
* 通过 `--indicators2` 指定的其他指标

![plot-dataframe](assets/plot-dataframe.png)

可用参数：

--8<-- "commands/plot-dataframe.md"

示例：

``` bash
freqtrade plot-dataframe -p BTC/ETH --strategy AwesomeStrategy
```

`-p/--pairs` 参数可用于指定您想要绘制的交易对。

!!! Note
    `freqtrade plot-dataframe` 子命令为每个交易对生成一个图表文件。

指定自定义指标。
使用 `--indicators1` 用于主图，使用 `--indicators2` 用于下方的子图（如果值与价格范围不同）。

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --indicators1 sma ema --indicators2 macd
```

### 更多用法示例

要绘制多个交易对，用空格分隔：

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH XRP/ETH
```

要绘制某个时间范围（用于放大查看）

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --timerange=20180801-20180805
```

要绘制存储在数据库中的交易记录，请将 `--db-url` 与 `--trade-source DB` 结合使用：

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy --db-url sqlite:///tradesv3.dry_run.sqlite -p BTC/ETH --trade-source DB
```

要绘制回测结果中的交易记录，请使用 `--export-filename <filename>`

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy --export-filename user_data/backtest_results/backtest-result.json -p BTC/ETH
```

### plot-dataframe 基础

![plot-dataframe2](assets/plot-dataframe2.png)

`plot-dataframe` 子命令需要回测数据、一个策略以及一个回测结果文件或包含与策略对应的交易记录的数据库。

生成的图表将包含以下元素：

* 绿色三角形：策略发出的买入信号。（注意：并非每个买入信号都会生成交易，请与青色圆圈对比。）
* 红色三角形：策略发出的卖出信号。（同样，并非每个卖出信号都会终止交易，请与红色和绿色方块对比。）
* 青色圆圈：交易入场点。
* 红色方块：亏损或 0% 利润交易的出场点。
* 绿色方块：盈利交易的出场点。
* 与 K 线比例对应的指标（例如 SMA/EMA），通过 `--indicators1` 指定。
* 成交量（主图底部的柱状图）。
* 不同比例的指标（例如 MACD、RSI）在成交量柱状图下方，通过 `--indicators2` 指定。

!!! Note "布林带"
    如果存在 `bb_lowerband` 和 `bb_upperband` 列，布林带将自动添加到图表中，并绘制为从下轨到上轨的浅蓝色区域。

#### 高级绘图配置

可以在策略的 `plot_config` 参数中指定高级绘图配置。

使用 `plot_config` 时的额外功能包括：

* 为每个指标指定颜色
* 指定额外的子图
* 指定指标对以填充它们之间的区域

下面的示例绘图配置为指标指定了固定颜色。否则，连续的绘图可能每次产生不同的配色方案，使比较变得困难。
它还允许多个子图同时显示 MACD 和 RSI。

绘图类型可以使用 `type` 键进行配置。可选类型有：

* `scatter` 对应 `plotly.graph_objects.Scatter` 类（默认）。
* `bar` 对应 `plotly.graph_objects.Bar` 类。

可以在 `plotly` 字典中指定传递给 `plotly.graph_objects.*` 构造函数的额外参数。

带有内联注释的示例配置，解释了处理过程：

``` python
@property
def plot_config(self):
    """
        There are a lot of solutions how to build the return dictionary.
        The only important point is the return value.
        Example:
            plot_config = {'main_plot': {}, 'subplots': {}}

    """
    plot_config = {}
    plot_config['main_plot'] = {
        # Configuration for main plot indicators.
        # Assumes 2 parameters, emashort and emalong to be specified.
        f'ema_{self.emashort.value}': {'color': 'red'},
        f'ema_{self.emalong.value}': {'color': '#CCCCCC'},
        # By omitting color, a random color is selected.
        'sar': {},
        # fill area between senkou_a and senkou_b
        'senkou_a': {
            'color': 'green', #optional
            'fill_to': 'senkou_b',
            'fill_label': 'Ichimoku Cloud', #optional
            'fill_color': 'rgba(255,76,46,0.2)', #optional
        },
        # plot senkou_b, too. Not only the area to it.
        'senkou_b': {}
    }
    plot_config['subplots'] = {
         # Create subplot MACD
        "MACD": {
            'macd': {'color': 'blue', 'fill_to': 'macdhist'},
            'macdsignal': {'color': 'orange'},
            'macdhist': {'type': 'bar', 'plotly': {'opacity': 0.9}}
        },
        # Additional subplot RSI
        "RSI": {
            'rsi': {'color': 'red'}
        }
    }

    return plot_config
```

??? Note "作为属性（旧方法）"
    也可以将 plot_config 作为属性赋值（这曾经是默认方式）。
    这样做的缺点是策略参数不可用，导致某些配置无法工作。

    ``` python
        plot_config = {
            'main_plot': {
                # Configuration for main plot indicators.
                # Specifies `ema10` to be red, and `ema50` to be a shade of gray
                'ema10': {'color': 'red'},
                'ema50': {'color': '#CCCCCC'},
                # By omitting color, a random color is selected.
                'sar': {},
            # fill area between senkou_a and senkou_b
            'senkou_a': {
                'color': 'green', #optional
                'fill_to': 'senkou_b',
                'fill_label': 'Ichimoku Cloud', #optional
                'fill_color': 'rgba(255,76,46,0.2)', #optional
            },
            # plot senkou_b, too. Not only the area to it.
            'senkou_b': {}
            },
            'subplots': {
                # Create subplot MACD
                "MACD": {
                    'macd': {'color': 'blue', 'fill_to': 'macdhist'},
                    'macdsignal': {'color': 'orange'},
                    'macdhist': {'type': 'bar', 'plotly': {'opacity': 0.9}}
                },
                # Additional subplot RSI
                "RSI": {
                    'rsi': {'color': 'red'}
                }
            }
        }

    ```


!!! Note
    上述配置假设 `ema10`、`ema50`、`senkou_a`、`senkou_b`、
    `macd`、`macdsignal`、`macdhist` 和 `rsi` 是策略创建的 DataFrame 中的列。

!!! Warning
    `plotly` 参数仅在 Plotly 库中受支持，在 FreqUI 中不起作用。

!!! Note "交易仓位调整"
    如果使用了 `position_adjustment_enable` / `adjust_trade_position()`，交易的初始买入价格会在多个订单之间取平均值，交易起始价格很可能出现在 K 线范围之外。

## 绘制利润图

![plot-profit](assets/plot-profit.png)

`plot-profit` 子命令显示一个交互式图表，包含三个图：

* 所有交易对的平均收盘价。
* 回测的汇总利润。
注意，这不是真实世界的利润，而更多是一个估算值。
* 每个单独交易对的利润。
* 交易并行度。
* 水下期（回撤期间）。

第一个图有助于了解整体市场的走势。

第二个图将显示您的算法是否有效。
也许您想要一个稳步赚取小利润的算法，或者一个操作频率较低但波动较大的算法。
此图还会突出显示最大回撤期间的开始（和结束）。

第三个图可用于发现异常值，即导致利润峰值的交易对事件。

第四个图可以帮助您分析交易并行度，显示 max_open_trades 达到上限的频率。

`freqtrade plot-profit` 子命令的可用选项：

--8<-- "commands/plot-profit.md"

`-p/--pairs` 参数可用于限制此计算中考虑的交易对。

示例：

使用自定义回测导出文件

``` bash
freqtrade plot-profit  -p LTC/BTC --export-filename user_data/backtest_results/backtest-result.json
```

使用自定义数据库

``` bash
freqtrade plot-profit  -p LTC/BTC --db-url sqlite:///tradesv3.sqlite --trade-source DB
```

``` bash
freqtrade --datadir user_data/data/binance_save/ plot-profit -p LTC/BTC
```
