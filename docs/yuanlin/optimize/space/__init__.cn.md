# space/__init__.py

## 概述

`freqtrade/optimize/space/__init__.py` 是 space 子包的初始化文件，负责导出 Hyperopt 优化空间相关的类，并提供兼容性别名。该文件从子模块导入具体实现，统一对外暴露接口。

## 导出内容

| 导出名称 | 来源 | 说明 |
|---------|------|------|
| `SKDecimal` | `decimalspace.SKDecimal` | 带小数精度的浮点分布 |
| `Dimension` | `optunaspaces.DimensionProtocol` | 维度协议（Protocol） |
| `Categorical` | `optunaspaces.ft_CategoricalDistribution` | 分类分布 |
| `Integer` | `optunaspaces.ft_IntDistribution` | 整数分布 |
| `Real` | `optunaspaces.ft_FloatDistribution` | 浮点分布 |

这些别名使得外部代码可以通过 `from freqtrade.optimize.space import Categorical, Integer, Real` 等简洁方式引用。

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.optimize.space.decimalspace` — `SKDecimal` 类
- `freqtrade.optimize.space.optunaspaces` — `DimensionProtocol`、`ft_CategoricalDistribution`、`ft_IntDistribution`、`ft_FloatDistribution`

### 被依赖（谁引用了本文件）
- `freqtrade.strategy.parameters` — 策略参数定义中使用这些分布类
- `freqtrade.optimize.hyperopt.hyperopt_auto` — Hyperopt 自动空间生成
- `freqtrade.optimize.hyperopt.hyperopt_interface` — Hyperopt 接口
- `freqtrade.optimize.hyperopt.hyperopt_optimizer` — Hyperopt 优化器
