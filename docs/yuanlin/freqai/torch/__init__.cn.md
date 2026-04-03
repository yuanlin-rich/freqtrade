# __init__.py

## 概述

`freqtrade/freqai/torch/__init__.py` 是 FreqAI PyTorch 子模块的包初始化文件。该文件内容为空，仅用于将 `torch/` 目录标记为一个 Python 包，使得其他模块可以通过 `from freqtrade.freqai.torch import ...` 的方式导入该包下的模块。

## 依赖关系

### 内部依赖（本项目模块）
- 无

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- 该包下的所有模块（`datasets`、`PyTorchDataConvertor`、`PyTorchMLPModel`、`PyTorchModelTrainer`、`PyTorchTrainerInterface`、`PyTorchTransformerModel`）均属于此包
