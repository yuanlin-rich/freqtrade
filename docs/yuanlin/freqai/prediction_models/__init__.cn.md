# __init__.py

## 概述

FreqAI 预测模型包的初始化文件。该文件为空，仅用于将 `prediction_models` 目录标记为 Python 包，使其中的预测模型类可以被其他模块导入。

## 说明

该包包含了 FreqAI 框架中所有内置的预测模型实现，涵盖以下几类：

- **LightGBM 系列**：基于 LightGBM 的回归和分类模型（单目标/多目标）
- **XGBoost 系列**：基于 XGBoost 的回归和分类模型（包含 Random Forest 变体）
- **PyTorch 系列**：基于 PyTorch 的 MLP 和 Transformer 模型
- **SKLearn 系列**：基于 scikit-learn 的随机森林分类器
- **强化学习系列**：基于 Stable Baselines3 的强化学习模型（单进程/多进程）

这些模型通过 `freqtrade.resolvers.freqaimodel_resolver` 动态加载，用户在配置文件中指定模型名称即可使用。
