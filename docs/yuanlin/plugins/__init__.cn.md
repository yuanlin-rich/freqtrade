# plugins/__init__.py

## 概述

`plugins` 包的初始化模块。该文件内容为空，仅作为 Python 包标识。plugins 包包含 freqtrade 的插件系统，主要包括交易对列表管理（PairListManager）和保护机制管理（ProtectionManager），以及其下的 pairlist 和 protections 子包。

## 依赖关系

### 被依赖（谁引用了本文件）
无直接引用。子模块通过各自的完整路径被导入（如 `from freqtrade.plugins.pairlistmanager import PairListManager`）。
