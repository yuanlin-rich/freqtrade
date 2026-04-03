# 订单流数据

本指南将引导您在 Freqtrade 中利用公开交易数据进行高级订单流分析。

!!! Warning "实验性功能"
    订单流功能目前处于测试阶段，在未来版本中可能会发生变化。请在 [Freqtrade GitHub 仓库](https://github.com/freqtrade/freqtrade/issues)上报告任何问题或反馈。
    此功能目前尚未与 freqAI 进行过测试——目前将这两个功能结合使用视为超出支持范围。

!!! Warning "性能"
    订单流功能需要原始交易数据。这些数据相当大，在 freqtrade 需要下载最近 X 根 K 线的交易数据时，可能会导致初始启动较慢。此外，启用此功能将导致内存使用量增加。请确保有足够的资源可用。

## 入门

### 启用公开交易数据

在您的 `config.json` 文件中，将 `exchange` 部分下的 `use_public_trades` 选项设置为 true。

```json
"exchange": {
   ...
   "use_public_trades": true,
}
```

### 配置订单流处理

在 config.json 的 orderflow 部分中定义您期望的订单流处理设置。在此，您可以调整以下参数：

- `cache_size`：保存到缓存中的之前订单流 K 线数量，而不是每根新 K 线都重新计算
- `max_candles`：筛选您希望获取交易数据的 K 线数量。
- `scale`：控制足迹图的价格区间大小。
- `stacked_imbalance_range`：定义所需的最小连续不平衡价格级别数量。
- `imbalance_volume`：过滤掉低于此阈值的不平衡成交量。
- `imbalance_ratio`：过滤掉比率（买卖量之差）低于此值的不平衡。

```json
"orderflow": {
    "cache_size": 1000,
    "max_candles": 1500,
    "scale": 0.5,
    "stacked_imbalance_range": 3, //  needs at least this amount of imbalance next to each other
    "imbalance_volume": 1, //  filters out below
    "imbalance_ratio": 3 //  filters out ratio lower than
  },
```

## 下载回测交易数据

要下载用于回测的历史交易数据，请使用 freqtrade download-data 命令配合 --dl-trades 标志。

```bash
freqtrade download-data -p BTC/USDT:USDT --timerange 20230101- --trading-mode futures --timeframes 5m --dl-trades
```

!!! Warning "数据可用性"
    并非所有交易所都提供公开交易数据。对于支持的交易所，如果您使用 `--dl-trades` 标志开始下载数据时公开交易数据不可用，freqtrade 将发出警告。

## 访问订单流数据

一旦激活，数据框中将出现几个新的列：

``` python

dataframe["trades"] # 包含每笔个别交易的信息。
dataframe["orderflow"] # 表示足迹图字典（见下文）
dataframe["imbalances"] # 包含订单流不平衡的信息。
dataframe["bid"] # 总买入量
dataframe["ask"] # 总卖出量
dataframe["delta"] # 卖出量和买入量之差。
dataframe["min_delta"] # K 线内的最小 delta
dataframe["max_delta"] # K 线内的最大 delta
dataframe["total_trades"] # 总交易数
dataframe["stacked_imbalances_bid"] # 堆叠买入不平衡范围起始价格级别列表
dataframe["stacked_imbalances_ask"] # 堆叠卖出不平衡范围起始价格级别列表
```

您可以在策略代码中访问这些列进行进一步分析。以下是一个示例：

``` python
def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    # Calculating cumulative delta
    dataframe["cum_delta"] = cumulative_delta(dataframe["delta"])
    # Accessing total trades
    total_trades = dataframe["total_trades"]
    ...

def cumulative_delta(delta: Series):
    cumdelta = delta.cumsum()
    return cumdelta

```

### 足迹图（`dataframe["orderflow"]`）

此列提供不同价格级别的买卖订单的详细分解，为订单流动态提供有价值的洞察。配置中的 `scale` 参数决定了此表示的价格区间大小。

`orderflow` 列包含一个具有以下结构的字典：

``` output
{
    "price": {
        "bid_amount": 0.0,
        "ask_amount": 0.0,
        "bid": 0,
        "ask": 0,
        "delta": 0.0,
        "total_volume": 0.0,
        "total_trades": 0
    }
}
```

#### Orderflow 列说明

- key：价格区间 - 按 `scale` 间隔分箱
- `bid_amount`：每个价格级别的总买入量。
- `ask_amount`：每个价格级别的总卖出量。
- `bid`：每个价格级别的买入订单数。
- `ask`：每个价格级别的卖出订单数。
- `delta`：每个价格级别的卖出量和买入量之差。
- `total_volume`：每个价格级别的总成交量（卖出量 + 买入量）。
- `total_trades`：每个价格级别的总交易数（卖出 + 买入）。

通过利用这些功能，您可以基于订单流分析获取有关市场情绪和潜在交易机会的有价值洞察。

### 原始交易数据（`dataframe["trades"]`）

包含 K 线期间发生的个别交易的列表。这些数据可用于对订单流动态进行更细粒度的分析。

每个条目包含一个具有以下键的字典：

- `timestamp`：交易的时间戳。
- `date`：交易的日期。
- `price`：交易价格。
- `amount`：交易量。
- `side`：买入或卖出。
- `id`：交易的唯一标识符。
- `cost`：交易的总成本（价格 * 数量）。

### 不平衡（`dataframe["imbalances"]`）

此列提供一个包含订单流不平衡信息的字典。当给定价格级别的卖出量和买入量之间存在显著差异时，就会发生不平衡。

每行如下所示——以价格为索引，对应的买入和卖出不平衡值为列

``` output
{
    "price": {
        "bid_imbalance": False,
        "ask_imbalance": False
    }
}
```
