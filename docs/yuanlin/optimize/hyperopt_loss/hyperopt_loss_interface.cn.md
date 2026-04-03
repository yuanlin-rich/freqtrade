# hyperopt_loss_interface.py

## 概述

`IHyperOptLoss` 抽象接口模块，定义了 hyperopt 损失函数的标准接口。所有自定义损失函数必须继承此接口并实现 `hyperopt_loss_function` 静态方法。

损失函数是 hyperopt 优化的核心——它将回测结果转化为一个标量值（损失值），优化器通过最小化此值来搜索最优参数组合。返回值越小表示结果越好。

## 架构图

```mermaid
classDiagram
    class IHyperOptLoss {
        <<abstract>>
        +str timeframe
        +hyperopt_loss_function(*, results, trade_count, min_date, max_date, config, processed, backtest_stats, starting_balance, **kwargs)$ float
    }

    IHyperOptLoss <|-- CalmarHyperOptLoss
    IHyperOptLoss <|-- SharpeHyperOptLoss
    IHyperOptLoss <|-- SortinoHyperOptLoss
    IHyperOptLoss <|-- MaxDrawDownHyperOptLoss
    IHyperOptLoss <|-- OnlyProfitHyperOptLoss
    IHyperOptLoss <|-- ShortTradeDurHyperOptLoss
    IHyperOptLoss <|-- ProfitDrawDownHyperOptLoss
    IHyperOptLoss <|-- MultiMetricHyperOptLoss
    IHyperOptLoss <|-- "... 更多实现"
```

## 核心类/函数

### IHyperOptLoss

抽象基类，继承自 `ABC`。

#### 类属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `timeframe` | `str` | 策略使用的时间帧 |

#### `hyperopt_loss_function(*, results, trade_count, min_date, max_date, config, processed, backtest_stats, starting_balance, **kwargs) -> float`
- **装饰器**: `@staticmethod`, `@abstractmethod`
- **参数**（全部为 keyword-only）:
  - `results: DataFrame` — 回测交易结果 DataFrame，包含 `profit_abs`, `profit_ratio`, `trade_duration`, `close_date` 等列
  - `trade_count: int` — 交易总数
  - `min_date: datetime` — 回测起始日期
  - `max_date: datetime` — 回测结束日期
  - `config: Config` — 全局配置字典
  - `processed: dict[str, DataFrame]` — 按交易对分组的已处理 OHLCV 数据
  - `backtest_stats: dict[str, Any]` — 回测统计摘要
  - `starting_balance: float` — 起始资金
  - `**kwargs` — 预留的扩展参数
- **返回**: `float` — 损失值，越小表示结果越好
- **说明**: 子类实现时不必使用所有参数，可以只接收需要的参数加上 `*args, **kwargs`

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.constants` — `Config` 类型

### 外部依赖（第三方库）
- `pandas` — `DataFrame` 类型
- `abc` — `ABC`, `abstractmethod`
- `datetime` — `datetime` 类型

### 被依赖（谁引用了本文件）
- `freqtrade.optimize.hyperopt.__init__` — 导出 `IHyperOptLoss`
- `freqtrade.optimize.hyperopt.hyperopt_optimizer` — 导入用于类型标注和 unpickling
- `freqtrade.resolvers.hyperopt_resolver` — 加载损失函数实现
- 所有 `hyperopt_loss_*.py` 实现文件 — 通过 `from freqtrade.optimize.hyperopt import IHyperOptLoss` 间接导入
