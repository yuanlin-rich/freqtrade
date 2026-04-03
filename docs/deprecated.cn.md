# 已弃用的功能

本页面包含已被机器人开发团队声明为已弃用（DEPRECATED）且不再支持的命令行参数、配置参数和机器人功能的说明。请避免在您的配置中使用它们。

## 已移除的功能

### `--refresh-pairs-cached` 命令行选项

`--refresh-pairs-cached` 在回测、超参数优化和 edge 的场景中允许刷新 K 线数据以用于回测。
由于这容易造成混淆，并且会拖慢回测速度（同时也不是回测本身的一部分），因此已将其独立为一个单独的 freqtrade 子命令 `freqtrade download-data`。

此命令行选项在 2019.7-dev（develop 分支）中被弃用，并在 2019.9 中被移除。

### **--dynamic-whitelist** 命令行选项

此命令行选项在 2018 年被弃用，并在 freqtrade 2019.6-dev（develop 分支）和 freqtrade 2019.7 中被移除。
请参阅 [pairlists](plugins.md#pairlists-and-pairlist-handlers) 替代方案。

### `--live` 命令行选项

`--live` 在回测场景中允许下载最新的 tick 数据用于回测。
该选项仅下载最近 500 根 K 线，因此在获取良好的回测数据方面效果不佳。
在 2019-7-dev（develop 分支）和 freqtrade 2019.8 中被移除。

### `ticker_interval`（现为 `timeframe`）

对 `ticker_interval` 术语的支持在 2020.6 中被弃用，改为使用 `timeframe`——兼容代码在 2022.3 中被移除。

### 允许按顺序运行多个 pairlist

配置中原来的 `"pairlist"` 部分已被移除，替换为 `"pairlists"`——它是一个列表，用于指定 pairlist 的执行顺序。

旧的配置参数部分（`"pairlist"`）在 2019.11 中被弃用，并在 2020.4 中被移除。

### 从 volume-pairlist 中弃用 bidVolume 和 askVolume

由于只有 quoteVolume 可以在不同资产之间进行比较，其他选项（bidVolume、askVolume）在 2020.4 中被弃用，并在 2020.9 中被移除。

### 使用订单簿步进作为卖出价格

使用 `order_book_min` 和 `order_book_max` 曾经允许遍历订单簿并尝试找到下一个 ROI 档位——尝试提前下卖单。
然而由于这会增加风险且没有收益，出于可维护性目的在 2021.7 中被移除。

### 旧版 Hyperopt 模式

使用单独的 hyperopt 文件在 2021.4 中被弃用，并在 2021.9 中被移除。
请切换到新的[参数化策略](hyperopt.md)以使用新的 hyperopt 接口。

## 策略从 V2 到 V3 的变更

独立期货 / 做空交易在 2022.4 中引入。这需要对配置设置、策略接口等进行重大更改。

我们已尽最大努力保持与现有策略的兼容性，因此如果您只是想在现货市场继续使用 freqtrade，则无需进行任何更改。
虽然我们可能会在未来某个时候停止对当前接口的支持，但我们会另行通知并提供适当的过渡期。

请按照[策略迁移](strategy_migration.md)指南将您的策略迁移到新格式，以开始使用新功能。

### webhooks - 2022.4 的变更

#### `buy_tag` 已重命名为 `enter_tag`

这应该只影响您的策略和可能的 webhooks。
我们将保留 1-2 个版本的兼容层（因此 `buy_tag` 和 `enter_tag` 都将继续工作），但 webhooks 中对此的支持将在之后消失。

#### 命名变更

Webhook 术语从 "sell" 变更为 "exit"，从 "buy" 变更为 "entry"，并在此过程中移除了 "webhook" 前缀。

* `webhookbuy`, `webhookentry` -> `entry`
* `webhookbuyfill`, `webhookentryfill` -> `entry_fill`
* `webhookbuycancel`, `webhookentrycancel` -> `entry_cancel`
* `webhooksell`, `webhookexit` -> `exit`
* `webhooksellfill`, `webhookexitfill` -> `exit_fill`
* `webhooksellcancel`, `webhookexitcancel` -> `exit_cancel`

## 移除 `populate_any_indicators`

2023.3 版本移除了 `populate_any_indicators`，改为使用拆分的特征工程和目标方法。请阅读[迁移文档](strategy_migration.md#freqai-strategy)获取完整详情。

## 从配置中移除 `protections`

通过配置中的 `"protections": [],` 设置保护措施的方式已在 2024.10 中被移除，此前已发出弃用警告超过 3 年。

## hdf5 数据存储

使用 hdf5 作为数据存储在 2024.12 中被弃用，并在 2025.1 中被移除。我们建议切换到 feather 数据格式。

请在更新之前使用 [`convert-data` 子命令](data-download.md#sub-command-convert-data) 将现有数据转换为支持的格式之一。

## 通过配置进行高级日志设置

通过 `--logfile systemd` 和 `--logfile journald` 分别配置 syslog 和 journald 的方式已在 2025.3 中被弃用。
请改用基于配置的[日志设置](advanced-setup.md#advanced-logging)。

## 移除 edge 模块

edge 模块在 2023.9 中被弃用，并在 2025.6 中被移除。
edge 的所有功能已被移除，配置了 edge 将导致错误。

## 动态资金费率处理的调整

在 2025.12 版本中，动态资金费率的处理方式已调整，以支持低至 1 小时资金费率间隔的动态资金费率。
因此，所有支持的期货交易所的标记价格和资金费率时间框架已更改为 1 小时。

由于标记价格和 funding_fee K 线的时间框架已更改（通常从 8 小时变为 1 小时），已下载的数据需要调整或部分重新下载。
您可以重新下载所有数据（`freqtrade download-data [...] --erase` - :warning: 可能需要很长时间）——或者有选择地下载更新的数据。

### 策略

大多数策略无需调整即可继续正常工作——但是，使用 `@informative("8h", candle_type="funding_rate")` 或类似写法的策略需要将时间框架切换为 1h。
同样，`dp.get_pair_dataframe(metadata["pair"], "8h", candle_type="funding_rate")` 也需要切换为 1h。

freqtrade 会自动调整时间框架并返回 `funding_rates`，尽管给定了错误的时间框架。它会发出警告——并且可能仍会导致您的策略出错。

### 选择性重新下载数据

下面的脚本作为示例——您可能需要根据自己的需求调整时间框架和交易所！

``` bash
# 清理不再需要的数据
rm user_data/data/<exchange>/futures/*-mark*
rm user_data/data/<exchange>/futures/*-funding_rate*

# 下载新数据（仅需执行一次以修复标记价格和资金费率数据）
freqtrade download-data -t 1h --trading-mode futures --candle-types funding_rate mark [...] --timerange <full timerange you've got other data for>

```

上述操作的结果是您的 funding_rates 和 mark 数据将使用 1h 时间框架。
您可以通过 `freqtrade list-data --exchange <yourexchange> --show` 来验证。

!!! Note "附加参数"
    上述命令可能需要附加参数，例如配置文件或与默认值不同的显式 user_data 路径。

**Hyperliquid** 是一个特殊情况——它将不再需要 1h 标记价格数据，而是使用常规 K 线代替（此数据以前从未存在过，并且与 1h 期货 K 线相同）。由于我们不支持 Hyperliquid 的 download-data（他们不提供历史数据），因此 Hyperliquid 用户无需执行任何操作。

## freqAI 中的 Catboost 模型

CatBoost 模型已在 2025.12 版本中被移除，不再得到积极支持。
如果您有使用 CatBoost 模型的现有机器人，您仍然可以通过从 git 历史记录中复制粘贴（如下方链接所示）并手动安装 Catboost 库来在自定义模型中使用它们。
但我们建议切换到其他支持的模型库，如 LightGBM 或 XGBoost，以获得更好的支持和未来的兼容性。

* [CatboostRegressor](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostRegressor.py)
* [CatboostClassifier](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostClassifier.py)
* [CatboostClassifierMultiTarget](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostClassifierMultiTarget.py)
