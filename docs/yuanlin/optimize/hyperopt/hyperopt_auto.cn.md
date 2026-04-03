# hyperopt_auto.py

## 概述

`HyperOptAuto` 自动超参数优化类模块。该类作为策略（`IHyperStrategy` 接口）与 hyperopt 系统之间的桥梁，自动将策略中定义的参数空间映射到 hyperopt 的搜索维度。

核心设计思想：用户无需编写独立的 HyperOpt 类，只需在策略中使用参数装饰器（如 `IntParameter`, `DecimalParameter` 等），`HyperOptAuto` 就能自动发现并构建搜索空间。如果策略内部定义了 `HyperOpt` 内部类，则优先使用该内部类的方法。

## 架构图

```mermaid
classDiagram
    class IHyperOpt {
        <<abstract>>
        +generate_estimator()
        +generate_roi_table()
        +roi_space()
        +stoploss_space()
        +trailing_space()
        +max_open_trades_space()
    }

    class HyperOptAuto {
        +get_available_spaces() list~str~
        +get_indicator_space(space) list
        +generate_roi_table(params) dict
        +roi_space() list
        +stoploss_space() list
        +generate_trailing_params(params) dict
        +trailing_space() list
        +max_open_trades_space() list
        +generate_estimator(dimensions, **kwargs)
        -_get_func(name) Callable
    }

    IHyperOpt <|-- HyperOptAuto
    HyperOptAuto --> IStrategy : strategy 属性
    HyperOptAuto --> "Strategy.HyperOpt" : 委托调用（可选）

    note for HyperOptAuto "自动发现策略参数空间\n委托给 Strategy.HyperOpt 或使用默认实现"
```

## 核心类/函数

### `_format_exception_message(space: str, ignore_missing_space: bool) -> None`
- **模块级函数**
- **参数**: `space` — 空间名称；`ignore_missing_space` — 是否忽略缺失空间
- **职责**: 当指定的 hyperopt 空间在策略中没有对应参数时，根据配置决定是发出警告还是抛出 `OperationalException`

### HyperOptAuto

继承自 `IHyperOpt`，自动委托 hyperopt 功能到策略类。

#### `get_available_spaces(self) -> list[str]`
- **返回**: 策略中定义的所有可用参数空间列表
- **实现**: 直接返回 `self.strategy._ft_hyper_params` 的键列表

#### `_get_func(self, name) -> Callable`
- **参数**: `name` — 方法名
- **返回**: 对应的可调用对象
- **逻辑**: 优先从策略的 `HyperOpt` 内部类查找方法，如果没有则回退到父类 `IHyperOpt` 的默认实现
- 这是实现"策略可选覆盖"模式的关键方法

#### `get_indicator_space(self, space: Literal["buy", "sell", "enter", "exit", "protection"] | str) -> list`
- **参数**: `space` — 参数空间类型
- **返回**: 该空间下所有启用优化（`optimize=True`）的参数的搜索维度列表
- **逻辑**: 遍历策略的 `enumerate_parameters(space)`，收集需要优化的参数；若为空则调用 `_format_exception_message` 处理

#### `generate_roi_table(self, params: dict) -> dict[int, float]`
- 委托给 `Strategy.HyperOpt.generate_roi_table` 或默认实现

#### `roi_space(self) -> list[Dimension]`
- 委托给 `Strategy.HyperOpt.roi_space` 或默认实现

#### `stoploss_space(self) -> list[Dimension]`
- 委托给 `Strategy.HyperOpt.stoploss_space` 或默认实现

#### `generate_trailing_params(self, params: dict) -> dict`
- 委托给 `Strategy.HyperOpt.generate_trailing_params` 或默认实现

#### `trailing_space(self) -> list[Dimension]`
- 委托给 `Strategy.HyperOpt.trailing_space` 或默认实现

#### `max_open_trades_space(self) -> list[Dimension]`
- 委托给 `Strategy.HyperOpt.max_open_trades_space` 或默认实现

#### `generate_estimator(self, dimensions: list[Dimension], **kwargs) -> EstimatorType`
- 委托给 `Strategy.HyperOpt.generate_estimator` 或默认实现
- 用于配置 Optuna sampler（采样器）

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.exceptions` — `OperationalException` 异常类
- `freqtrade.optimize.space` — `Dimension` 搜索空间维度类型
- `freqtrade.optimize.hyperopt.hyperopt_interface` — `EstimatorType` 类型别名和 `IHyperOpt` 基类

### 外部依赖（第三方库）
无直接外部依赖

### 被依赖（谁引用了本文件）
- `freqtrade.optimize.hyperopt.hyperopt_optimizer` — `HyperOptimizer` 创建 `HyperOptAuto` 实例用于搜索空间构建
- `tests/optimize/test_hyperopt.py` — 测试文件
