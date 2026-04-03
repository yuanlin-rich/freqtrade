# __init__.py

## 概述

`freqtrade/leverage/__init__.py` 是杠杆交易模块的包初始化文件，仅导出一个函数。

## 导出内容

- `interest` — 保证金交易利息计算函数

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.leverage.interest` — 利息计算函数

### 被依赖（谁引用了本文件）
- `freqtrade.persistence.trade_model` — 交易模型中计算利息
