# 参数表

下表将列出 FreqAI 可用的所有配置参数。其中一些参数在 `config_examples/config_freqai.example.json` 中有示例说明。

必需参数标记为 **Required**，必须以建议的方式之一进行设置。

### 通用配置参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`config.freqai` 树中的通用配置参数**
| `freqai` | **Required.** <br> 包含所有用于控制 FreqAI 的参数的父字典。<br> **数据类型：** Dictionary。
| `train_period_days` | **Required.** <br> 用于训练数据的天数（滑动窗口的宽度）。<br> **数据类型：** 正整数。
| `backtest_period_days` | **Required.** <br> 在滑动上面定义的 `train_period_days` 窗口并重新训练模型之前，从训练好的模型进行推理的天数（更多信息请参见[此处](freqai-running.md#backtesting)）。这可以是小数天数，但请注意，提供的 `timerange` 将除以此数字以得出完成回测所需的训练次数。<br> **数据类型：** Float。
| `identifier` | **Required.** <br> 当前模型的唯一 ID。如果模型保存到磁盘，`identifier` 允许重新加载特定的预训练模型/数据。<br> **数据类型：** String。
| `live_retrain_hours` | 模拟/实盘运行期间重新训练的频率。<br> **数据类型：** Float > 0。<br> 默认值：`0`（模型尽可能频繁地重新训练）。
| `expiration_hours` | 如果模型超过 `expiration_hours` 小时则避免进行预测。<br> **数据类型：** 正整数。<br> 默认值：`0`（模型永不过期）。
| `purge_old_models` | 磁盘上保留的模型数量（与回测无关）。默认值为 2，这意味着模拟/实盘运行将在磁盘上保留最新的 2 个模型。设置为 0 则保留所有模型。此参数也接受布尔值以保持向后兼容性。<br> **数据类型：** Integer。<br> 默认值：`2`。
| `save_backtest_models` | 运行回测时将模型保存到磁盘。回测最有效的运行方式是保存预测数据并在后续运行中直接重用它们（当您希望调整入场/出场参数时）。将回测模型保存到磁盘还允许使用相同的模型文件以相同的模型 `identifier` 启动模拟/实盘实例。<br> **数据类型：** Boolean。<br> 默认值：`False`（不保存模型）。
| `fit_live_predictions_candles` | 用于从预测数据（而非训练数据集）计算目标（标签）统计信息的历史蜡烛数量（更多信息请参见[此处](freqai-configuration.md#creating-a-dynamic-target-threshold)）。<br> **数据类型：** 正整数。
| `continual_learning` | 使用最近训练的模型的最终状态作为新模型的起点，允许增量学习（更多信息请参见[此处](freqai-running.md#continual-learning)）。请注意，这目前是一种朴素的增量学习方法，在市场偏离模型时，过拟合/陷入局部最小值的概率很高。我们提供这些连接主要是为了实验目的，以便为在加密市场等混沌系统中更成熟的持续学习方法做好准备。<br> **数据类型：** Boolean。<br> 默认值：`False`。
| `write_metrics_to_disk` | 在 json 文件中收集训练时间、推理时间和 CPU 使用情况。<br> **数据类型：** Boolean。<br> 默认值：`False`
| `data_kitchen_thread_count` | <br> 指定用于数据处理（异常值方法、归一化等）的线程数。这不会影响训练使用的线程数。如果用户未设置（默认），FreqAI 将使用最大线程数 - 2（为 Freqtrade 机器人和 FreqUI 保留 1 个物理核心）。<br> **数据类型：** 正整数。
| `activate_tensorboard` | <br> 指示是否为支持 tensorboard 的模块（目前包括强化学习、XGBoost、Catboost 和 PyTorch）激活 tensorboard。Tensorboard 需要安装 Torch，这意味着您需要 torch/RL docker 镜像，或者在安装时对是否安装 Torch 的问题回答"yes"。<br> **数据类型：** Boolean。<br> 默认值：`True`。
| `wait_for_training_iteration_on_reload` | <br> 使用 /reload 或 ctrl-c 时，等待当前训练迭代完成后再完成优雅关闭。如果设置为 `False`，FreqAI 将中断当前训练迭代，允许您更快地优雅关闭，但您将丢失当前的训练迭代。<br> **数据类型：** Boolean。<br> 默认值：`True`。

### 特征参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.feature_parameters` 子字典中的特征参数**
| `feature_parameters` | 包含用于特征工程的参数的字典。详细信息和示例请参见[此处](freqai-feature-engineering.md)。<br> **数据类型：** Dictionary。
| `include_timeframes` | `feature_engineering_expand_*()` 中所有指标将为其创建的时间周期列表。该列表作为特征添加到基础指标数据集中。<br> **数据类型：** 时间周期列表（字符串）。
| `include_corr_pairlist` | FreqAI 将作为附加特征添加到所有 `pair_whitelist` 币种的相关币种列表。在特征工程期间（详情请参见[此处](freqai-feature-engineering.md)），`feature_engineering_expand_*()` 中设置的所有指标都将为每个相关币种创建。相关币种特征将添加到基础指标数据集中。<br> **数据类型：** 资产列表（字符串）。
| `label_period_candles` | 为标签创建的未来蜡烛数量。这可以在 `set_freqai_targets()` 中使用（详细用法请参见 `templates/FreqaiExampleStrategy.py`）。此参数不是必需的，您可以创建自定义标签并选择是否使用此参数。请参见 `templates/FreqaiExampleStrategy.py` 以查看示例用法。<br> **数据类型：** 正整数。
| `include_shifted_candles` | 将先前蜡烛的特征添加到后续蜡烛中，目的是添加历史信息。如果使用，FreqAI 将复制并移位来自 `include_shifted_candles` 个先前蜡烛的所有特征，以便信息可用于后续蜡烛。<br> **数据类型：** 正整数。
| `weight_factor` | 根据数据点的时间近远程度对训练数据点进行加权（详情请参见[此处](freqai-feature-engineering.md#weighting-features-for-temporal-importance)）。<br> **数据类型：** 正浮点数（通常 < 1）。
| `indicator_max_period_candles` | **不再使用（#7325）**。已被在[策略](freqai-configuration.md#building-a-freqai-strategy)中设置的 `startup_candle_count` 取代。`startup_candle_count` 与时间周期无关，定义了 `feature_engineering_*()` 中用于指标创建的最大*周期*。FreqAI 将此参数与 `include_time_frames` 中的最大时间周期结合使用，以计算需要下载多少数据点，以确保第一个数据点不包含 NaN。<br> **数据类型：** 正整数。
| `indicator_periods_candles` | 计算指标的时间周期。指标将添加到基础指标数据集中。<br> **数据类型：** 正整数列表。
| `principal_component_analysis` | 使用主成分分析自动降低数据集的维度。有关工作原理的详细信息请参见[此处](freqai-feature-engineering.md#data-dimensionality-reduction-with-principal-component-analysis)。<br> **数据类型：** Boolean。<br> 默认值：`False`。
| `plot_feature_importances` | 为每个模型创建特征重要性图，显示前/后 `plot_feature_importances` 个特征。图表存储在 `user_data/models/<identifier>/sub-train-<COIN>_<timestamp>.html` 中。<br> **数据类型：** Integer。<br> 默认值：`0`。
| `DI_threshold` | 当设置为 > 0 时，激活使用差异性指数进行异常值检测。有关工作原理的详细信息请参见[此处](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di)。<br> **数据类型：** 正浮点数（通常 < 1）。
| `use_SVM_to_remove_outliers` | 训练支持向量机以检测并从训练数据集以及传入数据点中移除异常值。有关工作原理的详细信息请参见[此处](freqai-feature-engineering.md#identifying-outliers-using-a-support-vector-machine-svm)。<br> **数据类型：** Boolean。
| `svm_params` | Sklearn 的 `SGDOneClassSVM()` 中所有可用的参数。有关部分参数的详细信息请参见[此处](freqai-feature-engineering.md#identifying-outliers-using-a-support-vector-machine-svm)。<br> **数据类型：** Dictionary。
| `use_DBSCAN_to_remove_outliers` | 使用 DBSCAN 算法对数据进行聚类，以识别并从训练和预测数据中移除异常值。有关工作原理的详细信息请参见[此处](freqai-feature-engineering.md#identifying-outliers-with-dbscan)。<br> **数据类型：** Boolean。
| `noise_standard_deviation` | 如果设置，FreqAI 会向训练特征添加噪声，旨在防止过拟合。FreqAI 从标准差为 `noise_standard_deviation` 的高斯分布中生成随机偏差并将其添加到所有数据点。`noise_standard_deviation` 应保持相对于归一化空间，即在 -1 和 1 之间。换句话说，由于 FreqAI 中的数据始终归一化为 -1 到 1 之间，`noise_standard_deviation: 0.05` 将导致 32% 的数据随机增加/减少超过 2.5%（即落在第一个标准差内的数据百分比）。<br> **数据类型：** Integer。<br> 默认值：`0`。
| `outlier_protection_percentage` | 启用以防止异常值检测方法丢弃过多数据。如果 SVM 或 DBSCAN 检测到超过 `outlier_protection_percentage`% 的点为异常值，FreqAI 将记录警告消息并忽略异常值检测，即原始数据集将保持不变。如果异常值保护被触发，将不会基于训练数据集进行预测。<br> **数据类型：** Float。<br> 默认值：`30`。
| `reverse_train_test_order` | 拆分特征数据集（见下文）并使用最新数据拆分进行训练，在历史拆分数据上进行测试。这允许模型训练到最近的数据点，同时避免过拟合。但是，在使用此参数之前，您应该仔细理解其非常规性质。<br> **数据类型：** Boolean。<br> 默认值：`False`（不反转）。
| `shuffle_after_split` | 将数据拆分为训练集和测试集，然后分别对两个集合进行随机打乱。<br> **数据类型：** Boolean。<br> 默认值：`False`。
| `buffer_train_data_candles` | 在指标计算完成*之后*，从训练数据的开头和结尾各裁剪 `buffer_train_data_candles` 个蜡烛。主要使用示例是在预测最大值和最小值时，argrelextrema 函数无法知道时间范围边缘处的最大值/最小值。为了提高模型精度，最好在完整时间范围上计算 argrelextrema，然后使用此函数按内核大小裁剪掉边缘（缓冲区）。在另一种情况下，如果目标设置为移位价格变动，则不需要此缓冲区，因为时间范围末尾的移位蜡烛将为 NaN，FreqAI 将自动从训练数据集中裁剪掉这些。<br> **数据类型：** Integer。<br> 默认值：`0`。

### 数据拆分参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.data_split_parameters` 子字典中的数据拆分参数**
| `data_split_parameters` | 包含 scikit-learn `test_train_split()` 中可用的任何附加参数，具体参见[此处](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)（外部网站）。<br> **数据类型：** Dictionary。
| `test_size` | 应用于测试而非训练的数据比例。<br> **数据类型：** 正浮点数 < 1。
| `shuffle` | 训练期间随机打乱训练数据点。通常，为了不破坏时间序列预测中数据的时间顺序，此项设置为 `False`。<br> **数据类型：** Boolean。<br> 默认值：`False`。

### 模型训练参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.model_training_parameters` 子字典中的模型训练参数**
| `model_training_parameters` | 一个灵活的字典，包含所选模型库提供的所有参数。例如，如果您使用 `LightGBMRegressor`，此字典可以包含 `LightGBMRegressor` 中[此处](https://lightgbm.readthedocs.io/en/latest/pythonapi/lightgbm.LGBMRegressor.html)（外部网站）可用的任何参数。如果您选择其他模型，此字典可以包含该模型的任何参数。当前可用模型的列表可在[此处](freqai-configuration.md#using-different-prediction-models)找到。<br> **数据类型：** Dictionary。
| `n_estimators` | 模型训练中拟合的 boosted 树数量。<br> **数据类型：** Integer。
| `learning_rate` | 模型训练期间的 boosting 学习率。<br> **数据类型：** Float。
| `n_jobs`, `thread_count`, `task_type` | 设置并行处理的线程数和 `task_type`（`gpu` 或 `cpu`）。不同的模型库使用不同的参数名称。<br> **数据类型：** Float。

### 强化学习参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.rl_config` 子字典中的强化学习参数**
| `rl_config` | 包含强化学习模型控制参数的字典。<br> **数据类型：** Dictionary。
| `train_cycles` | 训练时间步数将基于 `train_cycles * 训练数据点数量` 来设置。<br> **数据类型：** Integer。
| `max_trade_duration_candles`| 引导智能体训练将交易保持在期望的长度以下。示例用法见 `prediction_models/ReinforcementLearner.py` 中的可自定义 `calculate_reward()` 函数。<br> **数据类型：** int。
| `model_type` | stable_baselines3 或 SBcontrib 中的模型字符串。可用字符串包括：`'TRPO', 'ARS', 'RecurrentPPO', 'MaskablePPO', 'PPO', 'A2C', 'DQN'`。用户应通过查阅文档确保 `model_training_parameters` 与对应的 stable_baselines3 模型匹配。[PPO 文档](https://stable-baselines3.readthedocs.io/en/master/modules/ppo.html)（外部网站）<br> **数据类型：** string。
| `policy_type` | stable_baselines3 中可用的策略类型之一。<br> **数据类型：** string。
| `max_training_drawdown_pct` | 智能体在训练期间允许经历的最大回撤。<br> **数据类型：** float。<br> 默认值：0.8
| `cpu_count` | 专用于强化学习训练过程的线程/CPU 数量（取决于是否选择了 `ReinforcementLearner_multiproc`）。建议保持不变，默认情况下此值设置为物理核心总数减 1。<br> **数据类型：** int。
| `model_reward_parameters` | 在 `ReinforcementLearner.py` 中的可自定义 `calculate_reward()` 函数内使用的参数。<br> **数据类型：** int。
| `add_state_info` | 告诉 FreqAI 在训练和推理的特征集中包含状态信息。当前状态变量包括交易持续时间、当前利润、交易仓位。这仅在模拟/实盘运行中可用，在回测中自动切换为 false。<br> **数据类型：** bool。<br> 默认值：`False`。
| `net_arch` | 网络架构，在 [`stable_baselines3` 文档](https://stable-baselines3.readthedocs.io/en/master/guide/custom_policy.html#examples)中有详细描述。简要来说：`[<共享层>, dict(vf=[<非共享价值网络层>], pi=[<非共享策略网络层>])]`。默认设置为 `[128, 128]`，即定义 2 个共享隐藏层，每层 128 个单元。
| `randomize_starting_position` | 随机化每个回合的起始点以避免过拟合。<br> **数据类型：** bool。<br> 默认值：`False`。
| `drop_ohlc_from_features` | 不要在训练期间传递给智能体的特征集中包含归一化的 ohlc 数据（ohlc 仍将在所有情况下用于驱动环境）。<br> **数据类型：** Boolean。<br> **默认值：** `False`
| `progress_bar` | 显示包含当前进度、已用时间和预计剩余时间的进度条。<br> **数据类型：** Boolean。<br> 默认值：`False`。

### PyTorch 参数

#### 通用

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.model_training_parameters` 子字典中的模型训练参数**
| `learning_rate` | 传递给优化器的学习率。<br> **数据类型：** float。<br> 默认值：`3e-4`。
| `model_kwargs` | 传递给模型类的参数。<br> **数据类型：** dict。<br> 默认值：`{}`。
| `trainer_kwargs` | 传递给训练器类的参数。<br> **数据类型：** dict。<br> 默认值：`{}`。

#### trainer_kwargs

| 参数    | 描述 |
|--------------|-------------|
|              |  **`freqai.model_training_parameters.model_kwargs` 子字典中的模型训练参数**
| `n_epochs`   | `n_epochs` 参数是 PyTorch 训练循环中的关键设置，决定了整个训练数据集将用于更新模型参数的次数。一个 epoch 代表对整个训练数据集的一次完整遍历。覆盖 `n_steps`。必须设置 `n_epochs` 或 `n_steps` 之一。<br><br> **数据类型：** int。可选。<br> 默认值：`10`。
| `n_steps`    | 设置 `n_epochs` 的替代方式 - 要运行的训练迭代次数。这里的迭代指的是调用 `optimizer.step()` 的次数。如果设置了 `n_epochs` 则忽略此项。简化版函数：<br><br> n_epochs = n_steps / (n_obs / batch_size) <br><br> 这样做的动机是 `n_steps` 在不同的 n_obs（数据点数量）之间更容易优化和保持稳定。<br> <br> **数据类型：** int。可选。<br> 默认值：`None`。
| `batch_size` | 训练期间使用的批次大小。<br><br> **数据类型：** int。<br> 默认值：`64`。
| `early_stopping_patience` | 验证损失没有改善的 epoch 数量，超过此数量后训练将提前停止。这通过在模型不再改善时停止训练来帮助防止过拟合。设置为 `0` 可禁用提前停止。需要测试/验证集拆分（`test_size > 0`）。<br><br> **数据类型：** int。<br> 默认值：`0`（已禁用）。

### 附加参数

|  参数 | 描述 |
|------------|-------------|
|  |  **额外参数**
| `freqai.keras` | 如果所选模型使用 Keras（通常用于基于 TensorFlow 的预测模型），需要激活此标志以使模型保存/加载遵循 Keras 标准。<br> **数据类型：** Boolean。<br> 默认值：`False`。
| `freqai.conv_width` | 神经网络输入张量的宽度。这通过将历史数据点作为张量的第二维传入来替代移位蜡烛（`include_shifted_candles`）的需求。从技术上讲，此参数也可用于回归器，但它只会增加计算开销而不会改变模型训练/预测。<br> **数据类型：** Integer。<br> 默认值：`2`。
| `freqai.reduce_df_footprint` | 将所有数值列重新转换为 float32/int32，目的是减少内存/磁盘使用并减少训练/推理时间。此参数设置在 Freqtrade 配置文件的主层级（不在 FreqAI 内部）。<br> **数据类型：** Boolean。<br> 默认值：`False`。
| `freqai.override_exchange_check` | 覆盖交易所检查以强制 FreqAI 使用可能没有足够历史数据的交易所。如果您知道您的 FreqAI 模型和策略不需要历史数据，请将其设置为 True。<br> **数据类型：** Boolean。<br> 默认值：`False`。
