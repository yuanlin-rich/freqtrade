# __init__.py

## 概述

`freqtrade/strategy/__init__.py` 是策略模块的包初始化文件，负责统一导出策略开发者最常用的类、函数和工具。用户在自定义策略中只需 `from freqtrade.strategy import *` 即可获取所有必要的接口。

## 导出内容

### 核心类
- `IStrategy` — 策略基类接口
- `Trade` / `Order` / `PairLocks` — 交易持久化对象

### 参数类（用于 Hyperopt 优化）
- `BooleanParameter` / `CategoricalParameter` / `DecimalParameter` / `IntParameter` / `RealParameter`

### 装饰器
- `informative` — 用于声明 informative pair 的装饰器

### 时间帧辅助函数
- `timeframe_to_minutes` / `timeframe_to_next_date` / `timeframe_to_prev_date` / `timeframe_to_seconds` / `timeframe_to_msecs`

### 策略辅助函数
- `merge_informative_pair` — 合并 informative 数据
- `stoploss_from_open` — 根据开仓价计算相对止损
- `stoploss_from_absolute` — 根据绝对价格计算相对止损

### 类型
- `AnnotationType` — 图表注解类型

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.exchange` — 导入 timeframe 转换函数
- `freqtrade.ft_types` — 导入 AnnotationType
- `freqtrade.persistence` — 导入 Order, PairLocks, Trade
- `freqtrade.strategy.informative_decorator` — 导入 informative 装饰器
- `freqtrade.strategy.interface` — 导入 IStrategy
- `freqtrade.strategy.parameters` — 导入各种参数类
- `freqtrade.strategy.strategy_helper` — 导入辅助函数

### 被依赖（谁引用了本文件）
- 所有用户自定义策略文件 — 通过 `from freqtrade.strategy import ...` 使用
- `freqtrade.plot.plotting` — 导入 IStrategy
- `freqtrade.optimize.backtesting` — 导入策略接口
