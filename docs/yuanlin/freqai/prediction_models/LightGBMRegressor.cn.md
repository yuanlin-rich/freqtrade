# LightGBMRegressor.py

## 概述

基于 LightGBM 梯度提升框架实现的**单目标回归**预测模型。该类继承自 `BaseRegressionModel`，通过重写 `fit()` 方法使用 LightGBM 的 `LGBMRegressor` 来训练回归模型。这是 FreqAI 中最常用的回归模型之一，支持增量学习、评估集验证和样本权重。

## 架构图

```mermaid
classDiagram
    class IFreqaiModel {
        <<abstract>>
        +fit()*
        +train()
        +predict()
    }
    class BaseRegressionModel {
        +train()
        +predict()
    }
    class LightGBMRegressor {
        +fit(data_dictionary, dk) Any
    }
    IFreqaiModel <|-- BaseRegressionModel
    BaseRegressionModel <|-- LightGBMRegressor
    LightGBMRegressor ..> LGBMRegressor : 使用
    LightGBMRegressor ..> FreqaiDataKitchen : 使用
```

## 核心类/函数

### LightGBMRegressor

继承自 `BaseRegressionModel`，是 FreqAI 中使用 LightGBM 进行回归任务的标准实现。

#### fit(data_dictionary, dk, **kwargs) -> Any

训练 LightGBM 回归模型的核心方法。

**参数：**
- `data_dictionary: dict` — 包含所有训练/测试数据的字典，关键 key 包括：
  - `train_features` — 训练特征 DataFrame
  - `train_labels` — 训练标签 DataFrame
  - `train_weights` — 训练样本权重
  - `test_features` — 测试特征 DataFrame
  - `test_labels` — 测试标签 DataFrame
  - `test_weights` — 测试样本权重
- `dk: FreqaiDataKitchen` — 当前交易对/模型的数据处理对象

**返回值：** 训练好的 `LGBMRegressor` 模型对象

**关键逻辑：**
1. 检查配置中的 `test_size` 参数：若为 0 则不使用评估集，否则准备 `eval_set` 和 `eval_weights`
2. 与分类器版本不同，回归器直接使用 DataFrame 而不转为 numpy 数组
3. 调用 `self.get_init_model(dk.pair)` 获取增量学习的初始模型
4. 使用 `self.model_training_parameters` 配置实例化 `LGBMRegressor`
5. 调用 `model.fit()` 进行训练，传入样本权重和评估集

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.freqai.base_models.BaseRegressionModel` — 回归模型基类，提供 `train()` 和 `predict()` 方法
- `freqtrade.freqai.data_kitchen.FreqaiDataKitchen` — 数据处理工具类

### 外部依赖（第三方库）
- `lightgbm.LGBMRegressor` — LightGBM 回归器实现
- `logging` — 日志记录

### 被依赖（谁引用了本文件）
- 通过 `freqtrade.resolvers.freqaimodel_resolver` 动态加载 — 用户在配置文件中指定 `"freqaimodel": "LightGBMRegressor"` 时使用
