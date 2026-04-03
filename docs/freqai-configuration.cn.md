# 配置

FreqAI 通过典型的 [Freqtrade 配置文件](configuration.md) 和标准的 [Freqtrade 策略](strategy-customization.md) 进行配置。FreqAI 配置和策略文件的示例分别可以在 `config_examples/config_freqai.example.json` 和 `freqtrade/templates/FreqaiExampleStrategy.py` 中找到。

## 设置配置文件

 虽然有大量额外的参数可供选择，如[参数表](freqai-parameter-table.md#parameter-table)中所示，FreqAI 配置至少必须包含以下参数（参数值仅为示例）：

```json
    "freqai": {
        "enabled": true,
        "purge_old_models": 2,
        "train_period_days": 30,
        "backtest_period_days": 7,
        "identifier" : "unique-id",
        "feature_parameters" : {
            "include_timeframes": ["5m","15m","4h"],
            "include_corr_pairlist": [
                "ETH/USD",
                "LINK/USD",
                "BNB/USD"
            ],
            "label_period_candles": 24,
            "include_shifted_candles": 2,
            "indicator_periods_candles": [10, 20]
        },
        "data_split_parameters" : {
            "test_size": 0.25
        }
    }
```

完整的示例配置可在 `config_examples/config_freqai.example.json` 中找到。

!!! Note
    `identifier` 经常被新手忽略，然而这个值在你的配置中扮演着重要角色。这个值是你选择的用于描述某次运行的唯一 ID。保持它不变可以让你维持崩溃恢复能力以及更快的回测速度。一旦你想尝试新的运行（新特征、新模型等），你应该更改此值（或删除 `user_data/models/unique-id` 文件夹）。更多详情请参阅[参数表](freqai-parameter-table.md#feature-parameters)。

## 构建 FreqAI 策略

FreqAI 策略需要在标准 [Freqtrade 策略](strategy-customization.md) 中包含以下代码行：

```python
    # user should define the maximum startup candle count (the largest number of candles
    # passed to any single indicator)
    startup_candle_count: int = 20

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:

        # the model will return all labels created by user in `set_freqai_targets()`
        # (& appended targets), an indication of whether or not the prediction should be accepted,
        # the target mean/std values for each of the labels created by user in
        # `set_freqai_targets()` for each training period.

        dataframe = self.freqai.start(dataframe, metadata, self)

        return dataframe

    def feature_engineering_expand_all(self, dataframe: DataFrame, period, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        This function will automatically expand the defined features on the config defined
        `indicator_periods_candles`, `include_timeframes`, `include_shifted_candles`, and
        `include_corr_pairs`. In other words, a single feature defined in this function
        will automatically expand to a total of
        `indicator_periods_candles` * `include_timeframes` * `include_shifted_candles` *
        `include_corr_pairs` numbers of features added to the model.

        All features must be prepended with `%` to be recognized by FreqAI internals.

        :param df: strategy dataframe which will receive the features
        :param period: period of the indicator - usage example:
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)
        """

        dataframe["%-rsi-period"] = ta.RSI(dataframe, timeperiod=period)
        dataframe["%-mfi-period"] = ta.MFI(dataframe, timeperiod=period)
        dataframe["%-adx-period"] = ta.ADX(dataframe, timeperiod=period)
        dataframe["%-sma-period"] = ta.SMA(dataframe, timeperiod=period)
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)

        return dataframe

    def feature_engineering_expand_basic(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        This function will automatically expand the defined features on the config defined
        `include_timeframes`, `include_shifted_candles`, and `include_corr_pairs`.
        In other words, a single feature defined in this function
        will automatically expand to a total of
        `include_timeframes` * `include_shifted_candles` * `include_corr_pairs`
        numbers of features added to the model.

        Features defined here will *not* be automatically duplicated on user defined
        `indicator_periods_candles`

        All features must be prepended with `%` to be recognized by FreqAI internals.

        :param df: strategy dataframe which will receive the features
        dataframe["%-pct-change"] = dataframe["close"].pct_change()
        dataframe["%-ema-200"] = ta.EMA(dataframe, timeperiod=200)
        """
        dataframe["%-pct-change"] = dataframe["close"].pct_change()
        dataframe["%-raw_volume"] = dataframe["volume"]
        dataframe["%-raw_price"] = dataframe["close"]
        return dataframe

    def feature_engineering_standard(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        This optional function will be called once with the dataframe of the base timeframe.
        This is the final function to be called, which means that the dataframe entering this
        function will contain all the features and columns created by all other
        freqai_feature_engineering_* functions.

        This function is a good place to do custom exotic feature extractions (e.g. tsfresh).
        This function is a good place for any feature that should not be auto-expanded upon
        (e.g. day of the week).

        All features must be prepended with `%` to be recognized by FreqAI internals.

        :param df: strategy dataframe which will receive the features
        usage example: dataframe["%-day_of_week"] = (dataframe["date"].dt.dayofweek + 1) / 7
        """
        dataframe["%-day_of_week"] = (dataframe["date"].dt.dayofweek + 1) / 7
        dataframe["%-hour_of_day"] = (dataframe["date"].dt.hour + 1) / 25
        return dataframe

    def set_freqai_targets(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        Required function to set the targets for the model.
        All targets must be prepended with `&` to be recognized by the FreqAI internals.

        :param df: strategy dataframe which will receive the targets
        usage example: dataframe["&-target"] = dataframe["close"].shift(-1) / dataframe["close"]
        """
        dataframe["&-s_close"] = (
            dataframe["close"]
            .shift(-self.freqai_info["feature_parameters"]["label_period_candles"])
            .rolling(self.freqai_info["feature_parameters"]["label_period_candles"])
            .mean()
            / dataframe["close"]
            - 1
            )
        return dataframe
```

注意 `feature_engineering_*()` 是添加[特征](freqai-feature-engineering.md#feature-engineering)的地方。同时 `set_freqai_targets()` 用于添加标签/目标。完整的示例策略可在 `templates/FreqaiExampleStrategy.py` 中找到。

!!! Note
    `self.freqai.start()` 函数不能在 `populate_indicators()` 之外调用。

!!! Note
    特征**必须**在 `feature_engineering_*()` 中定义。在 `populate_indicators()` 中定义 FreqAI 特征将导致算法在实盘/模拟运行模式下失败。要添加不与特定交易对或时间框架关联的通用特征，你应该使用 `feature_engineering_standard()`（如 `freqtrade/templates/FreqaiExampleStrategy.py` 中所示）。

## 重要的 dataframe 键模式

以下是你可以在典型策略 dataframe（`df[]`）中使用/包含的值：

|  DataFrame 键 | 描述 |
|------------|-------------|
| `df['&*']` | 在 `set_freqai_targets()` 中以 `&` 为前缀的任何 dataframe 列都会被 FreqAI 内部视为训练目标（标签）（通常遵循 `&-s*` 的命名约定）。例如，要预测未来 40 根 K 线后的收盘价，你可以设置 `df['&-s_close'] = df['close'].shift(-self.freqai_info["feature_parameters"]["label_period_candles"])`，并在配置中设置 `"label_period_candles": 40`。FreqAI 进行预测并以相同的键（`df['&-s_close']`）返回结果，以便在 `populate_entry/exit_trend()` 中使用。 <br> **数据类型：** 取决于模型输出。
| `df['&*_std/mean']` | 训练期间（或使用 `fit_live_predictions_candles` 进行实时跟踪时）已定义标签的标准差和均值。通常用于了解预测的稀有程度（使用 `templates/FreqaiExampleStrategy.py` 中显示的 z-score，并在[此处](#创建动态目标阈值)解释，以评估在训练期间或使用 `fit_live_predictions_candles` 的历史数据中某个特定预测被观察到的频率）。 <br> **数据类型：** Float。
| `df['do_predict']` | 异常数据点的指示。返回值是 -2 到 2 之间的整数，让你知道预测是否可信。`do_predict==1` 表示预测可信。如果输入数据点的 Dissimilarity Index（DI，详情见[此处](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di)）高于配置中定义的阈值，FreqAI 将从 `do_predict` 中减去 1，导致 `do_predict==0`。如果 `use_SVM_to_remove_outliers` 处于激活状态，Support Vector Machine（SVM，详情见[此处](freqai-feature-engineering.md#identifying-outliers-using-a-support-vector-machine-svm)）也可能检测训练和预测数据中的异常值。在这种情况下，SVM 也会从 `do_predict` 中减去 1。如果输入数据点被 SVM 认为是异常值但不被 DI 认为是异常值，或反之亦然，结果将是 `do_predict==0`。如果 DI 和 SVM 都认为输入数据点是异常值，结果将是 `do_predict==-1`。与 SVM 类似，如果 `use_DBSCAN_to_remove_outliers` 处于激活状态，DBSCAN（详情见[此处](freqai-feature-engineering.md#identifying-outliers-with-dbscan)）也可能检测异常值并从 `do_predict` 中减去 1。因此，如果 SVM 和 DBSCAN 都处于激活状态，且识别出一个超过 DI 阈值的数据点为异常值，结果将是 `do_predict==-2`。一个特殊情况是 `do_predict == 2`，这意味着模型由于超过 `expired_hours` 而过期。 <br> **数据类型：** -2 到 2 之间的整数。
| `df['DI_values']` | Dissimilarity Index（DI）值是 FreqAI 对预测置信度的代理指标。较低的 DI 意味着预测接近训练数据，即预测置信度较高。详情见[此处](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di)。 <br> **数据类型：** Float。
| `df['%*']` | 在 `feature_engineering_*()` 中以 `%` 为前缀的任何 dataframe 列都会被视为训练特征。例如，你可以通过设置 `df['%-rsi']` 将 RSI 包含在训练特征集中（类似于 `templates/FreqaiExampleStrategy.py` 中的做法）。更多详情见[此处](freqai-feature-engineering.md)。 <br> **注意：** 由于以 `%` 为前缀的特征数量可以非常快速地增长（使用 `include_shifted_candles` 和 `include_timeframes` 等乘法功能，如[参数表](freqai-parameter-table.md)中所述，可以轻松工程化出数万个特征），这些特征会从 FreqAI 返回给策略的 dataframe 中移除。要保留某种类型的特征用于绘图目的，你需要以 `%%` 为前缀（见下方详情）。 <br> **数据类型：** 取决于用户创建的特征。
| `df['%%*']` | 在 `feature_engineering_*()` 中以 `%%` 为前缀的任何 dataframe 列都会被视为训练特征，与上面的 `%` 前缀完全相同。但在这种情况下，特征会返回给策略，用于 FreqUI/plot-dataframe 的绘图和在模拟运行/实盘/回测中的监控。 <br> **数据类型：** 取决于用户创建的特征。请注意，在 `feature_engineering_expand()` 中创建的特征将根据你配置的扩展（即 `include_timeframes`、`include_corr_pairlist`、`indicators_periods_candles`、`include_shifted_candles`）具有自动的 FreqAI 命名方案。因此，如果你想绘制来自 `feature_engineering_expand_all()` 的 `%%-rsi`，最终的绘图配置命名方案将是：`%%-rsi-period_10_ETH/USDT:USDT_1h`，表示 `period=10`、`timeframe=1h`、`pair=ETH/USDT:USDT` 的 `rsi` 特征（如果你使用合约交易对，会添加 `:USDT`）。在 `populate_indicators()` 中的 `self.freqai.start()` 之后简单添加 `print(dataframe.columns)` 即可查看返回给策略的所有可用特征的完整列表，用于绘图目的。

## 设置 `startup_candle_count`

FreqAI 策略中的 `startup_candle_count` 需要按照与标准 Freqtrade 策略相同的方式设置（详情见[此处](strategy-customization.md#strategy-startup-period)）。此值被 Freqtrade 用于确保在调用 `dataprovider` 时提供足够数量的数据，以避免在第一次训练开始时出现任何 NaN。你可以通过识别传递给指标创建函数（例如 TA-Lib 函数）的最长周期（以 K 线为单位）来轻松设置此值。在给出的示例中，`startup_candle_count` 为 20，因为这是 `indicators_periods_candles` 中的最大值。

!!! Note
    在某些情况下，TA-Lib 函数实际上需要比传递的 `period` 更多的数据，否则特征数据集会被 NaN 填充。根据经验，将 `startup_candle_count` 乘以 2 总是能得到完全没有 NaN 的训练数据集。因此，通常最安全的做法是将预期的 `startup_candle_count` 乘以 2。请留意以下日志消息来确认数据是干净的：

    ```
    2022-08-31 15:14:04 - freqtrade.freqai.data_kitchen - INFO - dropped 0 training points due to NaNs in populated dataset 4319.
    ```

## 创建动态目标阈值

何时进入或退出交易可以动态决定，以反映当前市场状况。FreqAI 允许你从模型训练中返回额外信息（更多信息见[此处](freqai-feature-engineering.md#returning-additional-info-from-training)）。例如，`&*_std/mean` 返回值描述了*最近一次训练期间*目标/标签的统计分布。将给定预测与这些值进行比较，可以让你了解预测的稀有程度。在 `templates/FreqaiExampleStrategy.py` 中，`target_roi` 和 `sell_roi` 被定义为距离均值 1.25 个 z-score，这导致接近均值的预测被过滤掉。

```python
dataframe["target_roi"] = dataframe["&-s_close_mean"] + dataframe["&-s_close_std"] * 1.25
dataframe["sell_roi"] = dataframe["&-s_close_mean"] - dataframe["&-s_close_std"] * 1.25
```

要考虑*历史预测*的总体分布来创建动态目标（而不是如上所述使用训练信息），你需要在配置中将 `fit_live_predictions_candles` 设置为你希望用于生成目标统计信息的历史预测 K 线数量。

```json
    "freqai": {
        "fit_live_predictions_candles": 300,
    }
```

如果设置了此值，FreqAI 将最初使用训练数据的预测，随后开始引入生成的实际预测数据。FreqAI 会保存这些历史数据，以便在你停止并使用相同 `identifier` 重新启动模型时可以重新加载。

## 使用不同的预测模型

FreqAI 有多个示例预测模型库，可以通过 `--freqaimodel` 标志直接使用。这些库包括 `LightGBM` 和 `XGBoost` 的回归、分类和多目标模型，可以在 `freqai/prediction_models/` 中找到。

回归模型和分类模型在预测的目标上有所不同 —— 回归模型预测连续值的目标，例如明天 BTC 的价格是多少，而分类器预测离散值的目标，例如明天 BTC 的价格是否会上涨。这意味着你需要根据使用的模型类型以不同的方式指定目标（详情见[下方](#设置模型目标)）。

上述所有模型库都实现了梯度提升决策树算法。它们都基于集成学习的原理，即将多个简单学习器的预测组合起来，得到更稳定和更具泛化能力的最终预测。这里的简单学习器就是决策树。梯度提升指的是学习方法，其中每个简单学习器按顺序构建 —— 后续学习器用于改进前一个学习器的误差。如果你想了解更多关于不同模型库的信息，可以在它们各自的文档中找到：

* LightGBM: <https://lightgbm.readthedocs.io/en/v3.3.2/#>
* XGBoost: <https://xgboost.readthedocs.io/en/stable/#>
* CatBoost: <https://catboost.ai/en/docs/>（自 2025.12 起不再积极支持）

网上也有许多描述和比较这些算法的文章。一些相对轻量的例子包括 [CatBoost vs. LightGBM vs. XGBoost — Which is the best algorithm?](https://towardsdatascience.com/catboost-vs-lightgbm-vs-xgboost-c80f40662924#:~:text=In%20CatBoost%2C%20symmetric%20trees%2C%20or,the%20same%20depth%20can%20differ.) 和 [XGBoost, LightGBM or CatBoost — which boosting algorithm should I use?](https://medium.com/riskified-technology/xgboost-lightgbm-or-catboost-which-boosting-algorithm-should-i-use-e7fda7bb36bc)。请记住，每种模型的性能在很大程度上取决于应用场景，因此任何报告的指标可能不适用于你对模型的特定使用。

除了 FreqAI 中已有的模型外，还可以使用 `IFreqaiModel` 类自定义和创建你自己的预测模型。我们鼓励你继承 `fit()`、`train()` 和 `predict()` 来自定义训练过程的各个方面。你可以将自定义 FreqAI 模型放在 `user_data/freqaimodels` 中 - freqtrade 将根据提供的 `--freqaimodel` 名称从那里加载它们 - 该名称必须与你自定义模型的类名一致。
请确保使用唯一的名称，以避免覆盖内置模型。

### 设置模型目标

#### 回归器

如果你使用回归器，你需要指定一个具有连续值的目标。FreqAI 包含多种回归器，例如通过 `--freqaimodel LightGBMRegressor` 标志使用的 `LightGBMRegressor`。设置回归目标以预测未来 100 根 K 线价格的示例如下：

```python
df['&s-close_price'] = df['close'].shift(-100)
```

如果你想预测多个目标，你需要使用与上面相同的语法定义多个标签。

#### 分类器

如果你使用分类器，你需要指定一个具有离散值的目标。FreqAI 包含多种分类器，例如通过 `--freqaimodel LightGBMClassifier` 标志使用的 `LightGBMClassifier`。如果你选择使用分类器，类别需要使用字符串设置。例如，如果你想预测未来 100 根 K 线的价格是上涨还是下跌，你可以设置：

```python
df['&s-up_or_down'] = np.where( df["close"].shift(-100) > df["close"], 'up', 'down')
```

如果你想预测多个目标，必须在同一标签列中指定所有标签。例如，你可以添加 `same` 标签来定义价格未变化的情况：

```python
df['&s-up_or_down'] = np.where( df["close"].shift(-100) > df["close"], 'up', 'down')
df['&s-up_or_down'] = np.where( df["close"].shift(-100) == df["close"], 'same', df['&s-up_or_down'])
```

## PyTorch 模块

### 快速开始

快速运行 PyTorch 模型的最简单方法是使用以下命令（用于回归任务）：

```bash
freqtrade trade --config config_examples/config_freqai.example.json --strategy FreqaiExampleStrategy --freqaimodel PyTorchMLPRegressor --strategy-path freqtrade/templates
```

!!! Note "安装/Docker"
    PyTorch 模块需要 `torch` 等大型包，应在 `./setup.sh -i` 过程中通过对"Do you also want dependencies for freqai-rl or PyTorch (~700mb additional space required) [y/N]?"回答 "y" 来显式请求。
    使用 Docker 的用户应确保使用附加了 `_freqaitorch` 后缀的 Docker 镜像。
    我们在 `docker/docker-compose-freqai.yml` 中提供了专门的 docker-compose 文件 - 可以通过 `docker compose -f docker/docker-compose-freqai.yml run ...` 使用 - 或者复制它来替换原始的 docker 文件。
    此 docker-compose 文件还包含一个（已禁用的）部分，用于在 Docker 容器中启用 GPU 资源。这显然假设系统有可用的 GPU 资源。

    PyTorch 在 2.3 版本中放弃了对 macOS x64（基于 Intel 的 Apple 设备）的支持。因此，freqtrade 也放弃了在此平台上对 PyTorch 的支持。

!!! Danger "安全提示"
    由于需要设置 `weights_only=False`，从磁盘加载已保存的模型可能会导致安全问题（如果使用远程模型文件，即从互联网下载的文件或从不受信任来源收到的文件），这可能会引起安全问题。
    只要你只加载自己训练的模型，就没有风险。

### 结构

#### 模型

你可以在 PyTorch 中通过简单地在自定义 [`IFreqaiModel` 文件](#使用不同的预测模型) 中定义 `nn.Module` 类，然后在 `def train()` 函数中使用该类来构建自己的神经网络架构。以下是使用 PyTorch 实现逻辑回归模型的示例（应与 nn.BCELoss 损失函数一起用于分类任务）。

```python

class LogisticRegression(nn.Module):
    def __init__(self, input_size: int):
        super().__init__()
        # Define your layers
        self.linear = nn.Linear(input_size, 1)
        self.activation = nn.Sigmoid()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # Define the forward pass
        out = self.linear(x)
        out = self.activation(out)
        return out

class MyCoolPyTorchClassifier(BasePyTorchClassifier):
    """
    This is a custom IFreqaiModel showing how a user might setup their own
    custom Neural Network architecture for their training.
    """

    @property
    def data_convertor(self) -> PyTorchDataConvertor:
        return DefaultPyTorchDataConvertor(target_tensor_type=torch.float)

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        config = self.freqai_info.get("model_training_parameters", {})
        self.learning_rate: float = config.get("learning_rate",  3e-4)
        self.model_kwargs: dict[str, Any] = config.get("model_kwargs",  {})
        self.trainer_kwargs: dict[str, Any] = config.get("trainer_kwargs",  {})

    def fit(self, data_dictionary: dict, dk: FreqaiDataKitchen, **kwargs) -> Any:
        """
        User sets up the training and test data to fit their desired model here
        :param data_dictionary: the dictionary holding all data for train, test,
            labels, weights
        :param dk: The datakitchen object for the current coin/model
        """

        class_names = self.get_class_names()
        self.convert_label_column_to_int(data_dictionary, dk, class_names)
        n_features = data_dictionary["train_features"].shape[-1]
        model = LogisticRegression(
            input_dim=n_features
        )
        model.to(self.device)
        optimizer = torch.optim.AdamW(model.parameters(), lr=self.learning_rate)
        criterion = torch.nn.CrossEntropyLoss()
        init_model = self.get_init_model(dk.pair)
        trainer = PyTorchModelTrainer(
            model=model,
            optimizer=optimizer,
            criterion=criterion,
            model_meta_data={"class_names": class_names},
            device=self.device,
            init_model=init_model,
            data_convertor=self.data_convertor,
            **self.trainer_kwargs,
        )
        trainer.fit(data_dictionary, self.splits)
        return trainer

```

#### 训练器

`PyTorchModelTrainer` 执行标准的 PyTorch 训练循环：
定义模型、损失函数和优化器，然后将它们移动到适当的设备（GPU 或 CPU）。在循环内部，我们遍历数据加载器中的批次，将数据移动到设备上，计算预测和损失，反向传播，并使用优化器更新模型参数。

此外，训练器还负责以下工作：
 - 保存和加载模型
 - 将数据从 `pandas.DataFrame` 转换为 `torch.Tensor`。

#### 与 FreqAI 模块的集成

与所有 FreqAI 模型一样，PyTorch 模型继承自 `IFreqaiModel`。`IFreqaiModel` 声明了三个抽象方法：`train`、`fit` 和 `predict`。我们在三个层次的继承结构中实现这些方法。
从顶层到底层：

1. `BasePyTorchModel` - 实现 `train` 方法。所有 `BasePyTorch*` 都继承它。负责通用数据准备（例如数据归一化）和调用 `fit` 方法。设置子类使用的 `device` 属性。设置父类使用的 `model_type` 属性。
2. `BasePyTorch*` - 实现 `predict` 方法。这里的 `*` 代表一组算法，如分类器或回归器。负责数据预处理、预测和必要时的后处理。
3. `PyTorch*Classifier` / `PyTorch*Regressor` - 实现 `fit` 方法。负责主要的训练流程，在这里初始化训练器和模型对象。

![image](assets/freqai_pytorch-diagram.png)

#### 完整示例

使用 MLP（多层感知器）模型、MSELoss 损失函数和 AdamW 优化器构建 PyTorch 回归器。

```python
class PyTorchMLPRegressor(BasePyTorchRegressor):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        config = self.freqai_info.get("model_training_parameters", {})
        self.learning_rate: float = config.get("learning_rate",  3e-4)
        self.model_kwargs: dict[str, Any] = config.get("model_kwargs",  {})
        self.trainer_kwargs: dict[str, Any] = config.get("trainer_kwargs",  {})

    def fit(self, data_dictionary: dict, dk: FreqaiDataKitchen, **kwargs) -> Any:
        n_features = data_dictionary["train_features"].shape[-1]
        model = PyTorchMLPModel(
            input_dim=n_features,
            output_dim=1,
            **self.model_kwargs
        )
        model.to(self.device)
        optimizer = torch.optim.AdamW(model.parameters(), lr=self.learning_rate)
        criterion = torch.nn.MSELoss()
        init_model = self.get_init_model(dk.pair)
        trainer = PyTorchModelTrainer(
            model=model,
            optimizer=optimizer,
            criterion=criterion,
            device=self.device,
            init_model=init_model,
            target_tensor_type=torch.float,
            **self.trainer_kwargs,
        )
        trainer.fit(data_dictionary)
        return trainer
```

这里我们创建了一个 `PyTorchMLPRegressor` 类，实现了 `fit` 方法。`fit` 方法指定了训练的构建块：模型、优化器、损失函数和训练器。我们同时继承了 `BasePyTorchRegressor` 和 `BasePyTorchModel`，前者实现了适合我们回归任务的 `predict` 方法，后者实现了 `train` 方法。

??? Note "为分类器设置类别名称"
    使用分类器时，用户必须通过覆盖 `IFreqaiModel.class_names` 属性来声明类别名称（或目标）。这通过在 FreqAI 策略的 `set_freqai_targets` 方法中设置 `self.freqai.class_names` 来实现。

    例如，如果你使用二分类器来预测价格变动方向（上涨或下跌），你可以这样设置类别名称：
    ```python
    def set_freqai_targets(self, dataframe: DataFrame, metadata: dict, **kwargs) -> DataFrame:
        self.freqai.class_names = ["down", "up"]
        dataframe['&s-up_or_down'] = np.where(dataframe["close"].shift(-100) >
                                                  dataframe["close"], 'up', 'down')

        return dataframe
    ```
    要查看完整示例，请参考[分类器测试策略类](https://github.com/freqtrade/freqtrade/blob/develop/tests/strategy/strats/freqai_test_classifier.py)。


#### 使用 `torch.compile()` 提升性能

Torch 提供了一个 `torch.compile()` 方法，可用于提升特定 GPU 硬件的性能。更多详情见[此处](https://pytorch.org/tutorials/intermediate/torch_compile_tutorial.html)。简而言之，你只需将 `model` 包装在 `torch.compile()` 中：


```python
        model = PyTorchMLPModel(
            input_dim=n_features,
            output_dim=1,
            **self.model_kwargs
        )
        model.to(self.device)
        model = torch.compile(model)
```

然后正常使用模型即可。请注意，这样做将移除即时执行模式，这意味着错误和回溯信息将不具有参考价值。
