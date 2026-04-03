# 特征工程

## 定义特征

底层特征工程在用户策略中通过一组名为 `feature_engineering_*` 的函数执行。这些函数设置`基础特征`，如 `RSI`、`MFI`、`EMA`、`SMA`、时间、成交量等。`基础特征`可以是自定义指标，也可以从你能找到的任何技术分析库中导入。FreqAI 配备了一组函数来简化快速的大规模特征工程：

|  函数 | 描述 |
|---------------|-------------|
| `feature_engineering_expand_all()` | 此可选函数将根据配置中定义的 `indicator_periods_candles`、`include_timeframes`、`include_shifted_candles` 和 `include_corr_pairs` 自动扩展已定义的特征。
| `feature_engineering_expand_basic()` | 此可选函数将根据配置中定义的 `include_timeframes`、`include_shifted_candles` 和 `include_corr_pairs` 自动扩展已定义的特征。注意：此函数*不会*在 `indicator_periods_candles` 上进行扩展。
| `feature_engineering_standard()` | 此可选函数将使用基础时间框架的 dataframe 调用一次。这是最后被调用的函数，这意味着进入此函数的 dataframe 将包含由其他 `feature_engineering_expand` 函数从基础资产创建的所有特征和列。此函数适合执行自定义的特殊特征提取（例如 tsfresh）。此函数也适合放置任何不应被自动扩展的特征（例如星期几）。
| `set_freqai_targets()` | 用于设置模型目标的必需函数。所有目标必须以 `&` 为前缀才能被 FreqAI 内部识别。

同时，高级特征工程在 FreqAI 配置的 `"feature_parameters":{}` 中处理。在该文件中，可以决定在 `base_features` 之上进行大规模特征扩展，例如"包含相关交易对"或"包含信息性时间框架"甚至"包含近期 K 线"。

建议从源码提供的示例策略（位于 `templates/FreqaiExampleStrategy.py`）中的模板 `feature_engineering_*` 函数开始，以确保特征定义遵循正确的约定。以下是如何在策略中设置指标和标签的示例：

```python
    def feature_engineering_expand_all(self, dataframe: DataFrame, period, metadata, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        This function will automatically expand the defined features on the config defined
        `indicator_periods_candles`, `include_timeframes`, `include_shifted_candles`, and
        `include_corr_pairs`. In other words, a single feature defined in this function
        will automatically expand to a total of
        `indicator_periods_candles` * `include_timeframes` * `include_shifted_candles` *
        `include_corr_pairs` numbers of features added to the model.

        All features must be prepended with `%` to be recognized by FreqAI internals.

        Access metadata such as the current pair/timeframe/period with:

        `metadata["pair"]` `metadata["tf"]`  `metadata["period"]`

        :param df: strategy dataframe which will receive the features
        :param period: period of the indicator - usage example:
        :param metadata: metadata of current pair
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)
        """

        dataframe["%-rsi-period"] = ta.RSI(dataframe, timeperiod=period)
        dataframe["%-mfi-period"] = ta.MFI(dataframe, timeperiod=period)
        dataframe["%-adx-period"] = ta.ADX(dataframe, timeperiod=period)
        dataframe["%-sma-period"] = ta.SMA(dataframe, timeperiod=period)
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)

        bollinger = qtpylib.bollinger_bands(
            qtpylib.typical_price(dataframe), window=period, stds=2.2
        )
        dataframe["bb_lowerband-period"] = bollinger["lower"]
        dataframe["bb_middleband-period"] = bollinger["mid"]
        dataframe["bb_upperband-period"] = bollinger["upper"]

        dataframe["%-bb_width-period"] = (
            dataframe["bb_upperband-period"]
            - dataframe["bb_lowerband-period"]
        ) / dataframe["bb_middleband-period"]
        dataframe["%-close-bb_lower-period"] = (
            dataframe["close"] / dataframe["bb_lowerband-period"]
        )

        dataframe["%-roc-period"] = ta.ROC(dataframe, timeperiod=period)

        dataframe["%-relative_volume-period"] = (
            dataframe["volume"] / dataframe["volume"].rolling(period).mean()
        )

        return dataframe

    def feature_engineering_expand_basic(self, dataframe: DataFrame, metadata, **kwargs) -> DataFrame:
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

        Access metadata such as the current pair/timeframe with:

        `metadata["pair"]` `metadata["tf"]`

        All features must be prepended with `%` to be recognized by FreqAI internals.

        :param df: strategy dataframe which will receive the features
        :param metadata: metadata of current pair
        dataframe["%-pct-change"] = dataframe["close"].pct_change()
        dataframe["%-ema-200"] = ta.EMA(dataframe, timeperiod=200)
        """
        dataframe["%-pct-change"] = dataframe["close"].pct_change()
        dataframe["%-raw_volume"] = dataframe["volume"]
        dataframe["%-raw_price"] = dataframe["close"]
        return dataframe

    def feature_engineering_standard(self, dataframe: DataFrame, metadata, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        This optional function will be called once with the dataframe of the base timeframe.
        This is the final function to be called, which means that the dataframe entering this
        function will contain all the features and columns created by all other
        freqai_feature_engineering_* functions.

        This function is a good place to do custom exotic feature extractions (e.g. tsfresh).
        This function is a good place for any feature that should not be auto-expanded upon
        (e.g. day of the week).

        Access metadata such as the current pair with:

        `metadata["pair"]`

        All features must be prepended with `%` to be recognized by FreqAI internals.

        :param df: strategy dataframe which will receive the features
        :param metadata: metadata of current pair
        usage example: dataframe["%-day_of_week"] = (dataframe["date"].dt.dayofweek + 1) / 7
        """
        dataframe["%-day_of_week"] = (dataframe["date"].dt.dayofweek + 1) / 7
        dataframe["%-hour_of_day"] = (dataframe["date"].dt.hour + 1) / 25
        return dataframe

    def set_freqai_targets(self, dataframe: DataFrame, metadata, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        Required function to set the targets for the model.
        All targets must be prepended with `&` to be recognized by the FreqAI internals.

        Access metadata such as the current pair with:

        `metadata["pair"]`

        :param df: strategy dataframe which will receive the targets
        :param metadata: metadata of current pair
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

在上述示例中，用户不希望将 `bb_lowerband` 作为特征传递给模型，因此没有以 `%` 为前缀。然而，用户希望将 `bb_width` 传递给模型用于训练/预测，因此以 `%` 为前缀。

定义完 `基础特征` 后，下一步是使用配置文件中强大的 `feature_parameters` 进行扩展：

```json
    "freqai": {
        //...
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
        //...
    }
```

上面配置中的 `include_timeframes` 是策略中每次调用 `feature_engineering_expand_*()` 时使用的时间框架（`tf`）。在此示例中，用户要求将 `rsi`、`mfi`、`roc` 和 `bb_width` 的 `5m`、`15m` 和 `4h` 时间框架包含在特征集中。

你可以使用 `include_corr_pairlist` 要求将定义的每个特征也包含在信息性交易对中。这意味着特征集将包含配置中定义的每个相关交易对（在此示例中为 `ETH/USD`、`LINK/USD` 和 `BNB/USD`）在所有 `include_timeframes` 上的 `feature_engineering_expand_*()` 中的所有特征。

`include_shifted_candles` 表示要包含在特征集中的前几根 K 线的数量。例如，`include_shifted_candles: 2` 告诉 FreqAI 将特征集中每个特征的前 2 根 K 线也包含进来。

总计，上述示例策略的用户创建的特征数量为：`include_timeframes` 的长度 * `feature_engineering_expand_*()` 中的特征数 * `include_corr_pairlist` 的长度 * `include_shifted_candles` 数量 * `indicator_periods_candles` 的长度
 $= 3 * 3 * 3 * 2 * 2 = 108$。

!!! note "了解更多关于创意特征工程"
    查看我们的 [Medium 文章](https://emergentmethods.medium.com/freqai-from-price-to-prediction-6fadac18b665)，旨在帮助用户学习如何创造性地工程化特征。

### 使用 `metadata` 更精细地控制 `feature_engineering_*` 函数

所有 `feature_engineering_*` 和 `set_freqai_targets()` 函数都会接收一个 `metadata` 字典，其中包含 FreqAI 为特征构建自动化的 `pair`、`tf`（时间框架）和 `period` 信息。因此，用户可以在 `feature_engineering_*` 函数内使用 `metadata` 作为条件，为特定的时间框架、周期、交易对等限定/保留特征。

```python
def feature_engineering_expand_all(self, dataframe: DataFrame, period, metadata, **kwargs) -> DataFrame:
    if metadata["tf"] == "1h":
        dataframe["%-roc-period"] = ta.ROC(dataframe, timeperiod=period)
```

这将阻止 `ta.ROC()` 被添加到 `"1h"` 以外的任何时间框架。

### 从训练中返回额外信息

可以在每次模型训练结束时通过在自定义预测模型类内部赋值 `dk.data['extra_returns_per_train']['my_new_value'] = XYZ` 来将重要指标返回给策略。

FreqAI 将此字典中赋值的 `my_new_value` 扩展为适配返回给策略的 dataframe 的形式。然后你可以在策略中通过 `dataframe['my_new_value']` 使用返回的指标。FreqAI 中返回值如何使用的示例是 `&*_mean` 和 `&*_std` 值，它们用于[创建动态目标阈值](freqai-configuration.md#creating-a-dynamic-target-threshold)。

另一个示例是用户想要使用交易数据库中的实时指标，如下所示：

```json
    "freqai": {
        "extra_returns_per_train": {"total_profit": 4}
    }
```

你需要在配置中设置标准字典，以便 FreqAI 可以返回正确的 dataframe 形状。这些值很可能会被预测模型覆盖，但在模型尚未设置它们或需要默认初始值的情况下，预设的值将被返回。

### 根据时间重要性加权特征

FreqAI 允许你通过指数函数设置 `weight_factor` 来更强地加权近期数据：

$$ W_i = \exp(\frac{-i}{\alpha*n}) $$

其中 $W_i$ 是在总共 $n$ 个数据点集合中第 $i$ 个数据点的权重。下图显示了不同权重因子对特征集中数据点的影响。

![weight-factor](assets/freqai_weight-factor.jpg)

## 构建数据管道

默认情况下，FreqAI 根据用户配置设置构建动态管道。默认设置是健壮的，旨在适用于多种方法。这两个步骤是 `MinMaxScaler(-1,1)` 和 `VarianceThreshold`（移除方差为 0 的任何列）。用户可以通过更多配置参数激活其他步骤。例如，如果用户在 `freqai` 配置中添加 `use_SVM_to_remove_outliers: true`，FreqAI 将自动将 [`SVMOutlierExtractor`](#使用支持向量机svm识别异常值) 添加到管道中。同样，用户可以在 `freqai` 配置中添加 `principal_component_analysis: true` 来激活 PCA。[DissimilarityIndex](#使用差异性指数di识别异常值) 通过 `DI_threshold: 1` 激活。此外，还可以通过 `noise_standard_deviation: 0.1` 向数据添加噪声。最后，用户可以通过 `use_DBSCAN_to_remove_outliers: true` 添加 [DBSCAN](#使用-dbscan-识别异常值) 异常值移除。

!!! note "更多可用信息"
    请查阅[参数表](freqai-parameter-table.md)了解这些参数的更多信息。


### 自定义管道

我们鼓励用户根据需要自定义数据管道，构建自己的数据管道。这可以通过在 `IFreqaiModel` 的 `train()` 函数内部简单地将 `dk.feature_pipeline` 设置为所需的 `Pipeline` 对象来完成；或者如果他们不想修改 `train()` 函数，可以在 `IFreqaiModel` 中覆盖 `define_data_pipeline`/`define_label_pipeline` 函数：

!!! note "更多可用信息"
    FreqAI 使用 [`DataSieve`](https://github.com/emergentmethods/datasieve) 管道，它遵循 SKlearn 管道 API，但在其他功能之外增加了 X、y 和 sample_weight 向量点移除之间的一致性、特征移除、特征名称跟踪等功能。

```python
from datasieve.transforms import SKLearnWrapper, DissimilarityIndex
from datasieve.pipeline import Pipeline
from sklearn.preprocessing import QuantileTransformer, StandardScaler
from freqai.base_models import BaseRegressionModel


class MyFreqaiModel(BaseRegressionModel):
    """
    Some cool custom model
    """
    def fit(self, data_dictionary: Dict, dk: FreqaiDataKitchen, **kwargs) -> Any:
        """
        My custom fit function
        """
        model = cool_model.fit()
        return model

    def define_data_pipeline(self) -> Pipeline:
        """
        User defines their custom feature pipeline here (if they wish)
        """
        feature_pipeline = Pipeline([
            ('qt', SKLearnWrapper(QuantileTransformer(output_distribution='normal'))),
            ('di', ds.DissimilarityIndex(di_threshold=1))
        ])

        return feature_pipeline

    def define_label_pipeline(self) -> Pipeline:
        """
        User defines their custom label pipeline here (if they wish)
        """
        label_pipeline = Pipeline([
            ('qt', SKLearnWrapper(StandardScaler())),
        ])

        return label_pipeline
```

在这里，你定义了将在训练和预测期间用于特征集的确切管道。你可以通过将大多数 SKLearn 转换步骤包装在 `SKLearnWrapper` 类中来使用它们，如上所示。此外，你还可以使用 [`DataSieve` 库](https://github.com/emergentmethods/datasieve) 中可用的任何转换。

你可以通过创建一个继承自 datasieve `BaseTransform` 的类并实现 `fit()`、`transform()` 和 `inverse_transform()` 方法来轻松添加自己的转换：

```python
from datasieve.transforms.base_transform import BaseTransform
# import whatever else you need

class MyCoolTransform(BaseTransform):
    def __init__(self, **kwargs):
        self.param1 = kwargs.get('param1', 1)

    def fit(self, X, y=None, sample_weight=None, feature_list=None, **kwargs):
        # do something with X, y, sample_weight, or/and feature_list
        return X, y, sample_weight, feature_list

    def transform(self, X, y=None, sample_weight=None,
                  feature_list=None, outlier_check=False, **kwargs):
        # do something with X, y, sample_weight, or/and feature_list
        return X, y, sample_weight, feature_list

    def inverse_transform(self, X, y=None, sample_weight=None, feature_list=None, **kwargs):
        # do/dont do something with X, y, sample_weight, or/and feature_list
        return X, y, sample_weight, feature_list
```

!!! note "提示"
    你可以在与 `IFreqaiModel` 相同的文件中定义此自定义类。

### 将自定义 `IFreqaiModel` 迁移到新管道

如果你创建了自己的自定义 `IFreqaiModel`，其中包含自定义的 `train()`/`predict()` 函数，*并且*你仍然依赖 `data_cleaning_train/predict()`，那么你需要迁移到新管道。如果你的模型*不*依赖 `data_cleaning_train/predict()`，那么你不需要担心此迁移。

有关迁移的更多详情请参阅[此处](strategy_migration.md#freqai-new-data-pipeline)。

## 异常值检测

股票和加密货币市场存在大量以异常数据点形式出现的非模式噪声。FreqAI 实现了多种方法来识别此类异常值，从而降低风险。

### 使用差异性指数（DI）识别异常值

差异性指数（DI）旨在量化与模型每次预测相关的不确定性。

你可以通过在配置中包含以下语句来告诉 FreqAI 使用 DI 从训练/测试数据集中移除异常数据点：

```json
    "freqai": {
        "feature_parameters" : {
            "DI_threshold": 1
        }
    }
```

这将在你的 `feature_pipeline` 中添加 `DissimilarityIndex` 步骤，并将阈值设置为 1。DI 允许将属于异常值（不存在于模型特征空间中）的预测因为低置信度而被丢弃。为此，FreqAI 测量每个训练数据点（特征向量）$X_{a}$ 与所有其他训练数据点之间的距离：

$$ d_{ab} = \sqrt{\sum_{j=1}^p(X_{a,j}-X_{b,j})^2} $$

其中 $d_{ab}$ 是归一化点 $a$ 和 $b$ 之间的距离，$p$ 是特征数量，即向量 $X$ 的长度。一组训练数据点的特征距离 $\overline{d}$ 就是平均距离的均值：

$$ \overline{d} = \sum_{a=1}^n(\sum_{b=1}^n(d_{ab}/n)/n) $$

$\overline{d}$ 量化了训练数据的分布范围，将其与新预测特征向量 $X_k$ 和所有训练数据之间的距离进行比较：

$$ d_k = \arg \min d_{k,i} $$

由此可以估算差异性指数：

$$ DI_k = d_k/\overline{d} $$

你可以通过 `DI_threshold` 调整 DI 来增加或减少训练模型的外推范围。较高的 `DI_threshold` 意味着 DI 更宽松，允许使用距离训练数据更远的预测；较低的 `DI_threshold` 则相反，会丢弃更多预测。

下图描述了三维数据集的 DI。

![DI](assets/freqai_DI.jpg)

### 使用支持向量机（SVM）识别异常值

你可以通过在配置中包含以下语句来告诉 FreqAI 使用支持向量机（SVM）从训练/测试数据集中移除异常数据点：

```json
    "freqai": {
        "feature_parameters" : {
            "use_SVM_to_remove_outliers": true
        }
    }
```

这将在你的 `feature_pipeline` 中添加 `SVMOutlierExtractor` 步骤。SVM 将在训练数据上进行训练，任何被 SVM 认为超出特征空间的数据点都将被移除。

你可以选择通过配置中的 `feature_parameters.svm_params` 字典为 SVM 提供额外参数，例如 `shuffle` 和 `nu`。

参数 `shuffle` 默认设置为 `False` 以确保结果一致。如果设置为 `True`，由于 `max_iter` 对于算法达到要求的 `tol` 来说太低，在同一数据集上多次运行 SVM 可能会产生不同的结果。增加 `max_iter` 可以解决此问题，但会使过程花费更长时间。

参数 `nu`，*非常*粗略地说，是应被视为异常值的数据点比例，应在 0 到 1 之间。

### 使用 DBSCAN 识别异常值

你可以配置 FreqAI 使用 DBSCAN 对训练/测试数据集中的异常值进行聚类和移除，或移除预测中的传入异常值，方法是在配置中激活 `use_DBSCAN_to_remove_outliers`：

```json
    "freqai": {
        "feature_parameters" : {
            "use_DBSCAN_to_remove_outliers": true
        }
    }
```

这将在你的 `feature_pipeline` 中添加 `DataSieveDBSCAN` 步骤。这是一种无监督机器学习算法，无需知道应有多少个聚类即可对数据进行聚类。

给定 $N$ 个数据点和距离 $\varepsilon$，DBSCAN 通过将所有在距离 $\varepsilon$ 内有 $N-1$ 个其他数据点的数据点设置为*核心点*来聚类数据集。在距离*核心点* $\varepsilon$ 以内但自身在距离 $\varepsilon$ 内没有 $N-1$ 个其他数据点的数据点被视为*边界点*。聚类就是*核心点*和*边界点*的集合。在距离 $<\varepsilon$ 内没有其他数据点的数据点被视为异常值。下图显示了 $N = 3$ 的聚类。

![dbscan](assets/freqai_dbscan.jpg)

FreqAI 使用 `sklearn.cluster.DBSCAN`（详细信息可在 scikit-learn 的网页[此处](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html)找到（外部网站）），其中 `min_samples`（$N$）取特征集中时间点（K 线）数量的 1/4。`eps`（$\varepsilon$）从特征集中所有数据点的成对距离的最近邻计算的 *k-distance 图* 的拐点处自动计算。


### 使用主成分分析进行数据降维

你可以通过在配置中激活 principal_component_analysis 来降低特征的维度：

```json
    "freqai": {
        "feature_parameters" : {
            "principal_component_analysis": true
        }
    }
```

这将对特征执行 PCA 并降低其维度，使数据集的解释方差 >= 0.999。降低数据维度可以使模型训练更快，从而允许使用更及时的模型。
