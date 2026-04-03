# __init__.py

## 概述

FreqAI TensorBoard 子模块的包初始化文件。它提供了 `TBLogger` 和 `TBCallback` 两个便捷别名，并实现了优雅的降级策略：当 PyTorch 未安装时（`ModuleNotFoundError`），自动降级为无操作的基类实现。

这种设计确保了非 PyTorch 用户仍然可以正常使用 FreqAI，不会因为缺少 torch 依赖而导致导入失败。

## 核心逻辑

```python
try:
    # 尝试导入完整的 TensorBoard 实现（依赖 torch）
    from freqtrade.freqai.tensorboard.tensorboard import TensorBoardCallback, TensorboardLogger
    TBLogger = TensorboardLogger
    TBCallback = TensorBoardCallback
except ModuleNotFoundError:
    # 如果 torch 未安装，降级为无操作的基类
    from freqtrade.freqai.tensorboard.base_tensorboard import (
        BaseTensorBoardCallback,
        BaseTensorboardLogger,
    )
    TBLogger = BaseTensorboardLogger
    TBCallback = BaseTensorBoardCallback
```

导出符号：
- `TBLogger` -- TensorBoard 日志记录器（`TensorboardLogger` 或 `BaseTensorboardLogger`）
- `TBCallback` -- TensorBoard 回调（`TensorBoardCallback` 或 `BaseTensorBoardCallback`）

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.freqai.tensorboard.tensorboard` -- 完整实现（可选）
- `freqtrade.freqai.tensorboard.base_tensorboard` -- 无操作降级实现

### 外部依赖（第三方库）
- `torch`（间接）-- 通过 tensorboard.py 中的 `torch.utils.tensorboard` 触发 ModuleNotFoundError

### 被依赖（谁引用了本文件）
- `freqtrade.freqai.utils` -- `get_tb_logger()` 函数导入 `TBLogger`
- `freqtrade.freqai.prediction_models.XGBoostRegressor` -- 使用 TBCallback
- `freqtrade.freqai.prediction_models.XGBoostRegressorMultiTarget` -- 使用 TBCallback
- `freqtrade.freqai.RL.BaseReinforcementLearningModel` -- 使用 TBCallback
