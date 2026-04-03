# hyperopt_optimizer.py

## 概述

`HyperOptimizer` 类模块，是 hyperopt 系统的核心优化引擎。该类封装了以下职责：

1. **回测引擎管理**：持有 `Backtesting` 实例，管理回测的生命周期
2. **搜索空间构建**：通过 `HyperOptAuto` 收集策略中定义的所有参数空间，并转换为 Optuna 兼容的分布
3. **单次优化执行**：`generate_optimizer()` 方法在每个 epoch 执行一次完整的回测，并计算损失值
4. **Optuna 优化器创建**：`get_optimizer()` 方法创建并配置 Optuna Study 对象
5. **数据准备与缓存**：加载历史数据、计算指标、使用 joblib pickle 缓存数据
6. **多进程 pickle 支持**：通过 `cloudpickle` 注册策略继承链中的模块，解决跨文件策略继承的序列化问题
7. **早停机制**：通过 Optuna 的 `Terminator` 实现 best-value stagnation 检测

该类会被序列化（pickle）并发送到 joblib 的工作进程中执行。

## 架构图

```mermaid
classDiagram
    class HyperOptimizer {
        -dict spaces
        -list dimensions
        -dict o_dimensions
        -Config config
        -datetime min_date
        -datetime max_date
        -Backtesting backtesting
        -list pairlist
        -HyperOptAuto custom_hyperopt
        -IHyperOptLoss custom_hyperoptloss
        -Callable calculate_loss
        -Path data_pickle_file
        -float market_change
        -int es_epochs
        +__init__(config, data_pickle_file)
        +prepare_hyperopt()
        +get_strategy_name() str
        +init_spaces()
        +generate_optimizer(params_dict) dict
        +generate_optimizer_wrapped(params_dict) dict
        +get_optimizer(random_state) Study
        +convert_dimensions_to_optuna_space(dimensions) dict
        +advise_and_trim(data) dict
        +prepare_hyperopt_data()
        +handle_mp_logging()
        -_get_params_details(params) dict
        -_get_no_optimize_details() dict
        -hyperopt_pickle_magic(bases)
        -_setup_logging_mp_workaround()
    }

    HyperOptimizer --> Backtesting : 持有回测引擎
    HyperOptimizer --> HyperOptAuto : 搜索空间委托
    HyperOptimizer --> IHyperOptLoss : 损失函数
    HyperOptimizer --> "optuna.Study" : 创建优化器
    HyperOptimizer --> HyperoptTools : 工具函数

    class Backtesting {
        +backtest(processed, start_date, end_date)
        +strategy: IStrategy
    }
```

## 核心类/函数

### 模块级常量

| 常量 | 值 | 说明 |
|---|---|---|
| `INITIAL_POINTS` | 30 | 初始随机探索点数，用于 Optuna sampler 的 `n_startup_trials` 或 `population_size` |
| `MAX_LOSS` | 100000 | 极大损失值，用于标记无效结果（如交易次数不足时） |
| `optuna_samplers_dict` | dict | Optuna 采样器名称到类的映射字典 |

### HyperOptimizer

#### `__init__(self, config: Config, data_pickle_file: Path) -> None`
- **参数**: `config` — 全局配置；`data_pickle_file` — 数据缓存文件路径
- **职责**:
  1. 创建 `Backtesting` 实例并加载策略
  2. 创建 `HyperOptAuto` 并绑定到策略
  3. 调用 `hyperopt_pickle_magic` 注册策略继承模块
  4. 通过 `HyperOptLossResolver` 加载损失函数
  5. 配置早停参数
  6. 设置多进程日志队列

#### `prepare_hyperopt(self) -> None`
- **职责**: 完整的 hyperopt 准备流程
  1. 调用 `init_spaces()` 初始化搜索空间
  2. 调用 `prepare_hyperopt_data()` 加载并预处理数据
  3. 释放不再需要的交换所连接和 pairlist 资源

#### `init_spaces(self)`
- **职责**: 初始化所有搜索空间维度
- **逻辑**:
  1. 预定义 7 个标准空间: `buy`, `sell`, `protection`, `roi`, `stoploss`, `trailing`, `trades`
  2. 追加策略中自定义的额外空间
  3. 对每个启用的空间，调用 `custom_hyperopt` 的对应方法获取维度列表
  4. 将所有维度展平到 `self.dimensions` 列表
  5. 转换为 Optuna 兼容的分布格式 `self.o_dimensions`
  6. 如果没有找到任何维度则抛出 `OperationalException`

#### `generate_optimizer(self, params_dict: dict) -> dict`
- **核心方法**：单次优化迭代
- **参数**: `params_dict` — 由 Optuna 采样的参数字典
- **返回**: 包含 `loss`, `params_dict`, `params_details`, `results_metrics` 等的结果字典
- **关键逻辑**:
  1. 将采样参数应用到策略（更新参数值、ROI 表、stoploss、trailing 等）
  2. 从 pickle 缓存加载预处理数据（使用 mmap 只读模式提升性能）
  3. 如果 `analyze_per_epoch`，重新运行指标计算
  4. 执行回测
  5. 调用损失函数计算目标值
  6. 如果交易次数低于 `hyperopt_min_trades`，返回 `MAX_LOSS`

#### `generate_optimizer_wrapped(self, params_dict: dict) -> dict`
- 被 `@delayed` 和 `@wrap_non_picklable_objects` 装饰器包装的 `generate_optimizer`
- 在子进程中先设置日志系统，再执行优化

#### `get_optimizer(self, random_state: int) -> optuna.Study`
- **参数**: `random_state` — 随机种子
- **返回**: 配置好的 `optuna.Study` 实例
- **逻辑**:
  1. 通过 `custom_hyperopt.generate_estimator()` 获取采样器
  2. 根据采样器类型创建对应的 Optuna sampler 实例
  3. 如果启用了早停，创建 `Terminator` 和 `BestValueStagnationEvaluator`
  4. 创建最小化方向的 `optuna.Study`

#### `convert_dimensions_to_optuna_space(self, s_dimensions: list) -> dict`
- 将内部的搜索空间维度对象转换为 Optuna 兼容的分布字典
- 支持 `ft_CategoricalDistribution`, `ft_IntDistribution`, `ft_FloatDistribution`, `SKDecimal`

#### `_get_params_details(self, params: dict) -> dict`
- 将原始参数字典按空间分组，生成可读的参数详情
- 对 `roi` 空间会调用 `generate_roi_table` 生成完整 ROI 表
- 对 `trailing` 空间会调用 `generate_trailing_params` 生成完整参数

#### `_get_no_optimize_details(self) -> dict`
- 获取不在优化范围内的参数当前值（作为固定参数记录）

#### `advise_and_trim(self, data: dict) -> dict`
- 运行策略的 `advise_all_indicators` 计算指标
- 裁剪启动期数据以获取正确的日期范围
- 计算市场变化率

#### `prepare_hyperopt_data(self) -> None`
- 加载回测数据
- 如果不是 `analyze_per_epoch` 模式，预先计算指标并缓存
- 将数据 pickle 序列化到磁盘

#### `hyperopt_pickle_magic(self, bases: tuple) -> None`
- 递归遍历策略类的继承链
- 对每个非 `IStrategy` 的基类，使用 `cloudpickle.register_pickle_by_value` 注册其模块
- 解决跨文件策略继承时的 pickle 序列化问题

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.constants` — `DATETIME_PRINT_FORMAT`, `Config`
- `freqtrade.data.converter` — `trim_dataframes` 数据裁剪
- `freqtrade.data.history` — `get_timerange` 获取时间范围
- `freqtrade.data.metrics` — `calculate_market_change` 市场变化率计算
- `freqtrade.enums` — `HyperoptState` 状态枚举
- `freqtrade.exceptions` — `OperationalException`
- `freqtrade.ft_types` — `BacktestContentType` 类型定义
- `freqtrade.misc` — `deep_merge_dicts`, `round_dict`
- `freqtrade.optimize.backtesting` — `Backtesting` 回测引擎
- `freqtrade.optimize.hyperopt.hyperopt_auto` — `HyperOptAuto`
- `freqtrade.optimize.hyperopt.hyperopt_logger` — `logging_mp_setup`, `logging_mp_handle`
- `freqtrade.optimize.hyperopt_loss.hyperopt_loss_interface` — `IHyperOptLoss`
- `freqtrade.optimize.hyperopt_tools` — `HyperoptStateContainer`, `HyperoptTools`
- `freqtrade.optimize.optimize_reports` — `generate_strategy_stats`
- `freqtrade.optimize.space` — `DimensionProtocol`, `SKDecimal`, 分布类
- `freqtrade.resolvers.hyperopt_resolver` — `HyperOptLossResolver`
- `freqtrade.util` — `dt_now`
- `freqtrade.util.dry_run_wallet` — `get_dry_run_wallet`

### 外部依赖（第三方库）
- `optuna` — 优化框架核心（Study, samplers, distributions, Terminator）
- `joblib` — `delayed`, `dump`, `load`, `wrap_non_picklable_objects` 序列化和并行化
- `joblib.externals.cloudpickle` — 增强的 pickle 序列化
- `pandas` — `DataFrame` 数据处理

### 被依赖（谁引用了本文件）
- `freqtrade.optimize.hyperopt.hyperopt` — `Hyperopt` 主类导入 `HyperOptimizer` 和 `INITIAL_POINTS`
