# 运行 FreqAI

有两种方式来训练和部署自适应机器学习模型——实盘部署和历史回测。在这两种情况下，FreqAI 都会运行/模拟周期性模型重新训练，如下图所示：

![freqai-window](assets/freqai_moving-window.jpg)

## 实盘部署

FreqAI 可以使用以下命令进行模拟/实盘运行：

```bash
freqtrade trade --strategy FreqaiExampleStrategy --config config_freqai.example.json --freqaimodel LightGBMRegressor
```

启动后，FreqAI 将根据配置设置开始训练一个新模型，使用新的 `identifier`。训练完成后，该模型将用于对传入的 K 线进行预测，直到有新模型可用。新模型通常会尽可能频繁地生成，FreqAI 管理着一个币种对的内部队列，试图使所有模型保持同等最新状态。FreqAI 将始终使用最近训练的模型对传入的实时数据进行预测。如果你不希望 FreqAI 尽可能频繁地重新训练新模型，可以设置 `live_retrain_hours` 告诉 FreqAI 至少等待指定小时数后再训练新模型。此外，你可以设置 `expired_hours` 告诉 FreqAI 避免在超过指定小时数的模型上进行预测。

训练好的模型默认保存到磁盘，以便在回测期间或崩溃后重用。你可以通过在配置中设置 `"purge_old_models": true` 来选择 [清除旧模型](#purging-old-model-data) 以节省磁盘空间。

要从已保存的回测模型（或从之前崩溃的模拟/实盘会话）开始模拟/实盘运行，你只需要指定特定模型的 `identifier`：

```json
    "freqai": {
        "identifier": "example",
        "live_retrain_hours": 0.5
    }
```

在这种情况下，虽然 FreqAI 将使用预训练模型初始化，但它仍会检查自模型训练以来经过了多少时间。如果自加载模型结束以来已经过了完整的 `live_retrain_hours`，FreqAI 将开始训练新模型。

### 自动数据下载

FreqAI 会自动下载确保通过定义的 `train_period_days` 和 `startup_candle_count` 进行模型训练所需的适当数据量（有关这些参数的详细描述，请参阅 [参数表](freqai-parameter-table.md)）。

### 保存预测数据

在特定 `identifier` 模型的生命周期内做出的所有预测都存储在 `historic_predictions.pkl` 中，以便在崩溃或配置更改后重新加载。

### 清除旧模型数据

FreqAI 在每次成功训练后存储新的模型文件。随着新模型的生成以适应新的市场条件，这些文件会变得过时。如果你计划让 FreqAI 长时间高频率重新训练运行，应在配置中启用 `purge_old_models`：

```json
    "freqai": {
        "purge_old_models": 4,
    }
```

这将自动清除除最近四个训练模型之外的所有旧模型以节省磁盘空间。输入 "0" 将永远不会清除任何模型。

## 回测

FreqAI 回测模块可以使用以下命令执行：

```bash
freqtrade backtesting --strategy FreqaiExampleStrategy --strategy-path freqtrade/templates --config config_examples/config_freqai.example.json --freqaimodel LightGBMRegressor --timerange 20210501-20210701
```

如果此命令从未使用现有配置文件执行过，FreqAI 将为每个交易对、扩展的 `--timerange` 内的每个回测窗口训练一个新模型。

回测模式需要在部署前 [下载必要的数据](#downloading-data-to-cover-the-full-backtest-period)（与模拟/实盘模式不同，在模拟/实盘模式下 FreqAI 自动处理数据下载）。你应该注意考虑下载数据的时间范围应大于回测时间范围。这是因为 FreqAI 需要在期望的回测时间范围之前的数据来训练模型，以便准备好在设定的回测时间范围的第一根 K 线上进行预测。有关如何计算需要下载的数据的更多详情可在 [此处](#deciding-the-size-of-the-sliding-training-window-and-backtesting-duration) 找到。

!!! Note "模型重用"
    训练完成后，你可以使用相同的配置文件再次执行回测，FreqAI 将找到训练好的模型并加载它们，而不是花时间重新训练。如果你想调整（甚至 hyperopt）策略中的买卖条件，这很有用。如果你*想*使用相同的配置文件重新训练新模型，你只需更改 `identifier`。
    这样，你可以通过简单地指定 `identifier` 来返回使用任何你希望的模型。

!!! Note
    回测为每个回测窗口调用一次 `set_freqai_targets()`（窗口数量是完整回测时间范围除以 `backtest_period_days` 参数）。这样做意味着目标模拟了模拟/实盘行为，没有前瞻偏差。然而，`feature_engineering_*()` 中特征的定义在整个训练时间范围上只执行一次。这意味着你应该确保特征不会前瞻到未来。
    更多关于前瞻偏差的详情可在 [常见错误](strategy-customization.md#common-mistakes-when-developing-strategies) 中找到。

---

### 保存回测预测数据

为了允许你调整策略（**不是**特征！），FreqAI 将在回测期间自动保存预测，以便可以在使用相同 `identifier` 模型的未来回测和实盘运行中重用。这提供了面向启用**高级 hyperopt** 入场/出场条件的性能增强。

将在 `unique-id` 文件夹中创建一个名为 `backtesting_predictions` 的额外目录，其中包含所有以 `feather` 格式存储的预测。

要更改你的**特征**，你**必须**在配置中设置一个新的 `identifier` 以通知 FreqAI 训练新模型。

要保存在特定回测期间生成的模型，以便你可以从其中一个开始实盘部署而不是训练新模型，你必须在配置中将 `save_backtest_models` 设置为 `True`。

!!! Note
    为确保模型可以重用，FreqAI 将使用长度为 1 的 dataframe 调用你的策略。
    如果你的策略需要更多数据才能生成相同的特征，则无法将回测预测重用于实盘部署，需要为每次新的回测更新 `identifier`。

!!! Danger "安全通知"
    从磁盘加载保存的模型如果使用远程模型文件（从互联网下载的文件或从不可信来源接收的文件）可能会导致安全问题，因为需要将 `weights_only=False`，这可能引发安全问题。
    只要你只加载自己训练的模型，就没有风险。

### 回测实盘收集的预测

FreqAI 允许你通过回测参数 `--freqai-backtest-live-models` 重用实盘历史预测。当你想重用在模拟/实盘运行中生成的预测进行比较或其他研究时，这很有用。

不需要指定 `--timerange` 参数，因为它将通过历史预测文件中的数据自动计算。

### 下载覆盖完整回测期间的数据

对于实盘/模拟部署，FreqAI 将自动下载必要的数据。但是，要使用回测功能，你需要使用 `download-data` 下载必要的数据（详情 [见此](data-download.md#data-downloading)）。你需要仔细注意理解需要下载多少*额外*数据，以确保在回测时间范围开始*之前*有足够的训练数据量。额外数据量可以通过从期望的回测时间范围起点向前推算 `train_period_days` 和 `startup_candle_count`（有关这些参数的详细描述，请参阅 [参数表](freqai-parameter-table.md)）来大致估算。

例如，要回测 `--timerange 20210501-20210701`，使用将 `train_period_days` 设置为 30 的 [示例配置](freqai-configuration.md#setting-up-the-configuration-file)，以及最大 `include_timeframes` 为 1h 的 `startup_candle_count: 40`，下载数据的起始日期需要为 `20210501` - 30 天 - 40 * 1h / 24 小时 = 20210330（比期望训练时间范围起始早 31.7 天）。

### 决定滑动训练窗口和回测持续时间的大小

回测时间范围使用配置文件中典型的 `--timerange` 参数定义。滑动训练窗口的持续时间由 `train_period_days` 设置，而 `backtest_period_days` 是滑动回测窗口，两者都以天数为单位（`backtest_period_days` 可以是浮点数以表示实盘/模拟模式下的亚日重新训练）。在展示的 [示例配置](freqai-configuration.md#setting-up-the-configuration-file)（位于 `config_examples/config_freqai.example.json`）中，用户要求 FreqAI 使用 30 天的训练期并在随后的 7 天上回测。模型训练后，FreqAI 将回测随后的 7 天。然后"滑动窗口"向前移动一周（模拟 FreqAI 在实盘模式下每周重新训练一次），新模型使用前 30 天（包括前一个模型用于回测的 7 天）进行训练。这将重复直到 `--timerange` 结束。这意味着如果你设置 `--timerange 20210501-20210701`，FreqAI 将在 `--timerange` 结束时训练了 8 个独立模型（因为完整范围包含 8 周）。

!!! Note
    虽然允许小数的 `backtest_period_days`，但你应该注意 `--timerange` 除以此值来确定 FreqAI 需要训练多少个模型才能完成完整范围的回测。例如，设置 10 天的 `--timerange` 和 0.1 的 `backtest_period_days`，FreqAI 将需要为每个交易对训练 100 个模型才能完成完整的回测。因此，真正的 FreqAI 自适应训练回测将花费*很长*时间。完全测试模型的最佳方式是让它模拟运行并持续训练。在这种情况下，回测将花费与模拟运行完全相同的时间。

## 定义模型过期

在模拟/实盘模式下，FreqAI 按顺序训练每个币种对（在与主 Freqtrade 机器人分开的线程/GPU 上）。这意味着模型之间始终存在时间差异。如果你在 50 个交易对上训练，且每个交易对需要 5 分钟来训练，则最旧的模型将超过 4 小时。如果策略的特征时间尺度（目标交易持续时间）小于 4 小时，这可能是不理想的。你可以通过在配置文件中设置 `expiration_hours` 来决定仅在模型小于特定小时数时才进行交易入场：

```json
    "freqai": {
        "expiration_hours": 0.5,
    }
```

在展示的示例配置中，用户将仅允许在小于 1/2 小时的模型上进行预测。

## 控制模型学习过程

模型训练参数对于所选的机器学习库是唯一的。FreqAI 允许你使用配置中的 `model_training_parameters` 字典为任何库设置任何参数。示例配置（位于 `config_examples/config_freqai.example.json`）展示了与 `Catboost` 和 `LightGBM` 相关的一些示例参数，但你可以添加这些库或你选择实现的任何其他机器学习库中可用的任何参数。

数据拆分参数在 `data_split_parameters` 中定义，可以是与 scikit-learn 的 `train_test_split()` 函数关联的任何参数。`train_test_split()` 有一个名为 `shuffle` 的参数，允许打乱数据或保持不打乱。这对于避免用时间自相关数据偏置训练特别有用。有关这些参数的更多详情可在 [scikit-learn 网站](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)（外部网站）找到。

FreqAI 特有的参数 `label_period_candles` 定义了用于 `labels` 的偏移量（未来的 K 线数量）。在展示的 [示例配置](freqai-configuration.md#setting-up-the-configuration-file) 中，用户请求未来 24 根 K 线的 `labels`。

## 持续学习

你可以通过在配置中设置 `"continual_learning": true` 来选择采用持续学习方案。通过启用 `continual_learning`，在从头开始训练初始模型后，后续训练将从前一次训练的最终模型状态开始。这给新模型一个前一状态的"记忆"。默认情况下，此值设置为 `False`，这意味着所有新模型都从头开始训练，不接收来自先前模型的输入。

???+ danger "持续学习强制固定参数空间"
    由于 `continual_learning` 意味着模型参数空间在训练之间*不能*改变，当启用 `continual_learning` 时，`principal_component_analysis` 会自动禁用。提示：PCA 会改变参数空间和特征数量，在 [此处](freqai-feature-engineering.md#data-dimensionality-reduction-with-principal-component-analysis) 了解更多关于 PCA 的信息。

???+ danger "实验性功能"
    请注意，这目前是增量学习的一种简单方法，在市场偏离你的模型时有很高的过拟合/陷入局部最优的概率。我们在 FreqAI 中提供这些机制主要是出于实验目的，并为在加密市场等混沌系统中更成熟的持续学习方法做好准备。

## Hyperopt

你可以使用与 [典型 Freqtrade hyperopt](hyperopt.md) 相同的命令进行 hyperopt：

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLoss --strategy FreqaiExampleStrategy --freqaimodel LightGBMRegressor --strategy-path freqtrade/templates --config config_examples/config_freqai.example.json --timerange 20220428-20220507
```

`hyperopt` 要求你以与进行 [回测](#backtesting) 相同的方式预先下载数据。此外，在尝试 hyperopt FreqAI 策略时，你必须考虑一些限制：

- `--analyze-per-epoch` hyperopt 参数与 FreqAI 不兼容。
- 无法 hyperopt `feature_engineering_*()` 和 `set_freqai_targets()` 函数中的指标。这意味着你无法使用 hyperopt 优化模型参数。除此例外之外，可以优化所有其他 [空间](hyperopt.md#running-hyperopt-with-smaller-search-space)。
- 回测说明同样适用于 hyperopt。

将 hyperopt 与 FreqAI 结合的最佳方法是专注于 hyperopt 入场/出场阈值/条件。你需要专注于 hyperopt 不在特征中使用的参数。例如，你不应该尝试 hyperopt 特征创建中的滚动窗口长度，或 FreqAI 配置中任何改变预测的部分。为了高效地 hyperopt FreqAI 策略，FreqAI 将预测存储为 dataframe 并重用它们。因此要求仅 hyperopt 入场/出场阈值/条件。

一个好的 FreqAI 中可 hyperopt 参数的例子是 [不相似指数 (DI)](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di) 的阈值 `DI_values`，超过此值我们认为数据点为异常值：

```python
di_max = IntParameter(low=1, high=20, default=10, space='buy', optimize=True, load=True)
dataframe['outlier'] = np.where(dataframe['DI_values'] > self.di_max.value/10, 1, 0)
```

此特定 hyperopt 将帮助你了解适合你特定参数空间的 `DI_values`。

## 使用 Tensorboard

!!! note "可用性"
    FreqAI 为多种模型包含了 Tensorboard，包括 XGBoost、所有 PyTorch 模型、强化学习和 Catboost。如果你希望在其他模型类型中集成 Tensorboard，请在 [Freqtrade GitHub](https://github.com/freqtrade/freqtrade/issues) 上提交 issue。

!!! danger "要求"
    Tensorboard 日志记录需要 FreqAI torch 安装/Docker 镜像。


使用 Tensorboard 的最简单方法是确保配置文件中 `freqai.activate_tensorboard` 设置为 `True`（默认设置），运行 FreqAI，然后打开一个单独的 shell 并运行：

```bash
cd freqtrade
tensorboard --logdir user_data/models/unique-id
```

其中 `unique-id` 是 `freqai` 配置文件中设置的 `identifier`。如果你希望在浏览器中通过 127.0.0.1:6060 查看输出，此命令必须在单独的 shell 中运行（6060 是 Tensorboard 使用的默认端口）。

![tensorboard](assets/tensorboard.jpg)


!!! note "禁用以提高性能"
    Tensorboard 日志记录可能会减慢训练速度，应在生产使用中停用。
