# __init__.py

## 概述

`freqtrade/data/__init__.py` 是 `freqtrade.data` 包的初始化文件。它的功能非常简单，仅导入并暴露 `converter` 子模块，通过 `__all__` 限制 `from freqtrade.data import *` 的导入范围。

## 核心内容

```python
from freqtrade.data import converter
__all__ = ["converter"]
```

该文件为 data 包的入口，确保外部使用 `*` 导入时仅暴露 `converter` 模块。

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.data.converter` -- 数据格式转换子模块

### 外部依赖（第三方库）
无

### 被依赖（谁引用了本文件）
- 作为 `freqtrade.data` 包的入口，所有通过 `from freqtrade.data import ...` 导入的模块间接依赖此文件
