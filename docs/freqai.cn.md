![freqai-logo](assets/freqai_doc_logo.svg)

# FreqAI

## 简介

FreqAI 是一款旨在自动化与训练预测性机器学习模型相关的各种任务的软件，用于在给定一组输入信号的情况下生成市场预测。总体而言，FreqAI 旨在成为一个沙箱，便于在实时数据上轻松部署强大的机器学习库（[详情](#freqai-position-in-open-source-machine-learning-landscape)）。

!!! Note
    FreqAI 是且永远是一个非营利性的开源项目。FreqAI *没有*加密代币，FreqAI *不*销售信号，FreqAI 除了当前的 [freqtrade 文档](https://www.freqtrade.io/en/stable/freqai/) 之外没有其他域名。

功能包括：

* **自适应重新训练** - 在[实盘部署](freqai-running.md#live-deployments)期间以监督方式重新训练模型，使其自适应市场
* **快速特征工程** - 基于用户创建的简单策略创建大型丰富的[特征集](freqai-feature-engineering.md#feature-engineering)（10k+ 特征）
* **高性能** - 线程化允许在与模型推理（预测）和机器人交易操作分离的线程上进行自适应模型重训练（如果可用，也可在 GPU 上进行）。最新的模型和数据保存在 RAM 中以进行快速推理
* **真实的回测** - 通过[回测模块](freqai-running.md#backtesting)在历史数据上模拟自适应训练，自动化重训练过程
* **可扩展性** - 通用且健壮的架构允许整合 Python 中可用的任何[机器学习库/方法](freqai-configuration.md#using-different-prediction-models)。目前提供八个示例，包括分类器、回归器和卷积神经网络
* **智能异常值移除** - 使用多种[异常值检测技术](freqai-feature-engineering.md#outlier-detection)从训练和预测数据集中移除异常值
* **崩溃恢复能力** - 将训练好的模型存储到磁盘，使崩溃后的重新加载快速且简单，并[清除过时文件](freqai-running.md#purging-old-model-data)以支持持续的模拟/实盘运行
* **自动数据归一化** - 以智能且统计安全的方式[归一化数据](freqai-feature-engineering.md#building-the-data-pipeline)
* **自动数据下载** - 计算数据下载的时间范围并更新历史数据（在实盘部署中）
* **清洗传入数据** - 在训练和模型推理之前安全处理 NaN 值
* **降维** - 通过[主成分分析](freqai-feature-engineering.md#data-dimensionality-reduction-with-principal-component-analysis)减少训练数据的大小
* **部署机器人集群** - 设置一个机器人训练模型，而一组[消费者](producer-consumer.md)使用信号

## 快速开始

测试 FreqAI 最简单的方式是使用以下命令在模拟模式下运行：

```bash
freqtrade trade --config config_examples/config_freqai.example.json --strategy FreqaiExampleStrategy --freqaimodel LightGBMRegressor --strategy-path freqtrade/templates
```

您将看到自动数据下载的启动过程，随后是同时进行的训练和交易。

!!! danger "非生产用途"
    Freqtrade 源代码中提供的示例策略旨在展示/测试 FreqAI 的各种功能。它也设计为在小型计算机上运行，以便可以作为开发者和用户之间的基准。它*不*是为生产环境运行而设计的。

可用作起点的示例策略、预测模型和配置分别位于
`freqtrade/templates/FreqaiExampleStrategy.py`、`freqtrade/freqai/prediction_models/LightGBMRegressor.py` 和
`config_examples/config_freqai.example.json`。

## 总体方法

您向 FreqAI 提供一组自定义的*基础指标*（与[典型的 Freqtrade 策略](strategy-customization.md)中的方式相同）以及目标值（*标签*）。对于白名单中的每个交易对，FreqAI 训练一个模型来基于自定义指标的输入预测目标值。模型以预定的频率持续重新训练，以适应市场条件。FreqAI 提供了回测策略（通过在历史数据上进行周期性重训练来模拟现实）和部署模拟/实盘运行的能力。在模拟/实盘条件下，FreqAI 可以设置为在后台线程中持续重训练，以尽可能保持模型更新。

下面显示了算法概述，解释了数据处理管道和模型使用。

![freqai-algo](assets/freqai_algo.jpg)

### 重要的机器学习词汇

**Features（特征）** - 基于历史数据的参数，模型在其上进行训练。单个 K 线的所有特征存储为一个向量。在 FreqAI 中，您可以从策略中能构建的任何内容创建特征数据集。

**Labels（标签）** - 模型训练的目标值。每个特征向量与一个由您在策略中定义的单一标签相关联。这些标签有意地展望未来，是您训练模型希望能够预测的内容。

**Training（训练）** - "教"模型将特征集与关联标签匹配的过程。不同类型的模型以不同的方式"学习"，这意味着一种模型可能比另一种更适合特定应用。关于 FreqAI 中已实现的不同模型的更多信息可以在[此处](freqai-configuration.md#using-different-prediction-models)找到。

**Train data（训练数据）** - 特征数据集的一个子集，在训练期间输入模型以"教"模型如何预测目标。此数据直接影响模型中的权重连接。

**Test data（测试数据）** - 特征数据集的一个子集，用于在训练后评估模型性能。此数据不影响模型中的节点权重。

**Inferencing（推理）** - 向训练好的模型输入新的未见过的数据，模型将对其进行预测的过程。

## 安装前置依赖

正常的 Freqtrade 安装过程会询问您是否希望安装 FreqAI 依赖项。如果您希望使用 FreqAI，应回答"yes"。如果您没有回答 yes，可以在安装后使用以下命令手动安装这些依赖项：

``` bash
pip install -r requirements-freqai.txt
```

!!! Note
    Catboost 不会在低功耗的 ARM 设备（如树莓派）上安装，因为它不为该平台提供 wheel 包。

### 使用 Docker

如果您使用 Docker，提供了一个带有 FreqAI 依赖项的专用标签 `:freqai`。因此，您可以将 Docker Compose 文件中的镜像行替换为 `image: freqtradeorg/freqtrade:stable_freqai`。此镜像包含常规的 FreqAI 依赖项。与原生安装类似，Catboost 在基于 ARM 的设备上不可用。如果您想使用 PyTorch 或强化学习，应使用 torch 或 RL 标签：`image: freqtradeorg/freqtrade:stable_freqaitorch`、`image: freqtradeorg/freqtrade:stable_freqairl`。

!!! note "docker-compose-freqai.yml"
    我们在 `docker/docker-compose-freqai.yml` 中提供了一个专门的 docker-compose 文件——可以通过 `docker compose -f docker/docker-compose-freqai.yml run ...` 使用，或者复制来替换原始的 docker 文件。此 docker-compose 文件还包含一个（已禁用的）部分，用于在 Docker 容器中启用 GPU 资源。这显然假设系统有可用的 GPU 资源。

### FreqAI 在开源机器学习领域中的定位

预测基于混沌时间序列的系统（如股票/加密货币市场）需要一套广泛的工具来测试各种假说。幸运的是，最近成熟的强大机器学习库（例如 `scikit-learn`）开启了广泛的研究可能性。来自不同领域的科学家现在可以轻松地在大量成熟的机器学习算法上进行原型研究。同样，这些用户友好的库使"公民科学家"能够使用他们的基本 Python 技能进行数据探索。然而，在历史和实时混沌数据源上利用这些机器学习库在后勤上可能很困难且昂贵。此外，强大的数据收集、存储和处理也是一个不同的挑战。[`FreqAI`](#freqai) 旨在提供一个通用且可扩展的开源框架，面向市场预测的自适应建模的实时部署。`FreqAI` 框架实际上是丰富的开源机器学习库世界的沙箱。在 `FreqAI` 沙箱中，用户会发现他们可以组合各种第三方库，在免费的 24/7 实时混沌数据源——加密货币交易所数据上测试创造性的假说。

### 引用 FreqAI

FreqAI [发表在 Journal of Open Source Software](https://joss.theoj.org/papers/10.21105/joss.04864) 上。如果您在研究中发现 FreqAI 有用，请使用以下引用：

```bibtex
@article{Caulk2022,
    doi = {10.21105/joss.04864},
    url = {https://doi.org/10.21105/joss.04864},
    year = {2022}, publisher = {The Open Journal},
    volume = {7}, number = {80}, pages = {4864},
    author = {Robert A. Caulk and Elin Törnquist and Matthias Voppichler and Andrew R. Lawless and Ryan McMullan and Wagner Costa Santos and Timothy C. Pogue and Johan van der Vlugt and Stefan P. Gehring and Pascal Schmidt},
    title = {FreqAI: generalizing adaptive modeling for chaotic time-series market forecasts},
    journal = {Journal of Open Source Software} }
```

## 常见陷阱

FreqAI 不能与动态 `VolumePairlists`（或任何动态添加和删除交易对的 pairlist 过滤器）组合使用。
这是出于性能原因——FreqAI 依赖于快速预测/重训练。为了有效地做到这一点，
它需要在模拟/实盘实例开始时下载所有训练数据。FreqAI 自动存储和附加
新的 K 线数据以供未来重训练。这意味着如果由于 volume pairlist 导致新的交易对在模拟运行中稍后才出现，它将没有准备好的数据。然而，FreqAI 可以与 `ShufflePairlist` 或保持总交易对列表不变（但根据成交量重新排序交易对）的 `VolumePairlist` 一起使用。

## 额外学习材料

这里我们汇编了一些外部材料，提供了对 FreqAI 各个组件更深入的了解：

- [Real-time head-to-head: Adaptive modeling of financial market data using XGBoost and CatBoost](https://emergentmethods.medium.com/real-time-head-to-head-adaptive-modeling-of-financial-market-data-using-xgboost-and-catboost-995a115a7495)
- [FreqAI - from price to prediction](https://emergentmethods.medium.com/freqai-from-price-to-prediction-6fadac18b665)


## 支持

您可以在多个地方找到 FreqAI 的支持，包括 [Freqtrade Discord](https://discord.gg/Jd8JYeWHc4)、专用的 [FreqAI Discord](https://discord.gg/7AMWACmbjT) 以及 [GitHub issues](https://github.com/freqtrade/freqtrade/issues)。

## 致谢

FreqAI 由一群各自为项目贡献特定技能的个人开发。

构思和软件开发：
Robert Caulk @robcaulk

理论头脑风暴和数据分析：
Elin Törnquist @th0rntwig

代码审查和软件架构头脑风暴：
@xmatthias

软件开发：
Wagner Costa @wagnercosta
Emre Suzen @aemr3
Timothy Pogue @wizrds

Beta 测试和错误报告：
Stefan Gehring @bloodhunter4rc, @longyu, Andrew Lawless @paranoidandy, Pascal Schmidt @smidelis, Ryan McMullan @smarmau, Juha Nykänen @suikula, Johan van der Vlugt @jooopiert, Richárd Józsa @richardjosza
