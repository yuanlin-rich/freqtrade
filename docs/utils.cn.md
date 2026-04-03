# 实用工具子命令

除了实盘交易和模拟运行模式、`backtesting` 和 `hyperopt` 优化子命令以及用于准备历史数据的 `download-data` 子命令之外，机器人还包含许多实用工具子命令。本节将对它们进行描述。

## 创建用户目录

创建用于存放 freqtrade 文件的目录结构。
还会为你创建策略和超参数优化示例以便入门。
可以多次使用 - 使用 `--reset` 将把示例策略和超参数优化文件重置为默认状态。

--8<-- "commands/create-userdir.md"

!!! Warning
    使用 `--reset` 可能导致数据丢失，因为这将覆盖所有示例文件且不会再次询问确认。

```
├── backtest_results
├── data
├── hyperopt_results
├── hyperopts
│   ├── sample_hyperopt_loss.py
├── notebooks
│   └── strategy_analysis_example.ipynb
├── plot
└── strategies
    └── sample_strategy.py
```

## 创建新配置

创建一个新的配置文件，询问一些对配置来说很重要的选项。

--8<-- "commands/new-config.md"

!!! Warning
    只会询问关键问题。Freqtrade 提供了更多的配置可能性，详见[配置文档](configuration.md#configuration-parameters)。

### 创建配置示例

```
$ freqtrade new-config --config user_data/config_binance.json

? Do you want to enable Dry-run (simulated trades)?  Yes
? Please insert your stake currency: BTC
? Please insert your stake amount: 0.05
? Please insert max_open_trades (Integer or -1 for unlimited open trades): 3
? Please insert your desired timeframe (e.g. 5m): 5m
? Please insert your display Currency (for reporting): USD
? Select exchange  binance
? Do you want to enable Telegram?  No
```

## 显示配置

显示配置文件（默认情况下敏感值会被脱敏）。
在使用[拆分配置文件](configuration.md#multiple-configuration-files)或[环境变量](configuration.md#environment-variables)时特别有用，该命令将显示合并后的配置。

![Show config output](assets/show-config-output.png)

--8<-- "commands/show-config.md"

``` output
Your combined configuration is:
{
  "exit_pricing": {
    "price_side": "other",
    "use_order_book": true,
    "order_book_top": 1
  },
  "stake_currency": "USDT",
  "exchange": {
    "name": "binance",
    "key": "REDACTED",
    "secret": "REDACTED",
    "ccxt_config": {},
    "ccxt_async_config": {},
  }
  // ...
}
```

!!! Warning "分享此命令提供的信息"
    我们尽力从默认输出中移除所有已知的敏感信息（不使用 `--show-sensitive`）。
    但是，请再次检查输出中的敏感值，确保你不会意外暴露某些隐私信息。

## 创建新策略

从类似于 SampleStrategy 的模板创建新策略。
文件将以你的类名命名，不会覆盖现有文件。

结果将位于 `user_data/strategies/<strategyclassname>.py`。

--8<-- "commands/new-strategy.md"

### new-strategy 使用示例

```bash
freqtrade new-strategy --strategy AwesomeStrategy
```

使用自定义用户目录

```bash
freqtrade new-strategy --userdir ~/.freqtrade/ --strategy AwesomeStrategy
```

使用高级模板（填充所有可选函数和方法）

```bash
freqtrade new-strategy --strategy AwesomeStrategy --template advanced
```

## 列出策略

使用 `list-strategies` 子命令查看特定目录中的所有策略。

此子命令对于发现加载策略时的环境问题很有用：包含错误且加载失败的策略模块会以红色显示（LOAD FAILED），而名称重复的策略会以黄色显示（DUPLICATE NAME）。

--8<-- "commands/list-strategies.md"

!!! Warning
    使用这些命令将尝试加载目录中的所有 Python 文件。如果该目录中存在不受信任的文件，这可能存在安全风险，因为所有模块级代码都会被执行。

示例：搜索默认策略目录（在默认用户目录内）。

``` bash
freqtrade list-strategies
```

示例：搜索用户目录内的策略目录。

``` bash
freqtrade list-strategies --userdir ~/.freqtrade/
```

示例：搜索指定的策略路径。

``` bash
freqtrade list-strategies --strategy-path ~/.freqtrade/strategies/
```

## 列出 Hyperopt 损失函数

使用 `list-hyperoptloss` 子命令查看所有可用的超参数优化损失函数。

它提供了你环境中所有可用损失函数的快速列表。

此子命令对于发现加载损失函数时的环境问题很有用：包含错误且加载失败的 Hyperopt-Loss 函数模块会以红色显示（LOAD FAILED），而名称重复的 Hyperopt-Loss 函数会以黄色显示（DUPLICATE NAME）。

--8<-- "commands/list-hyperoptloss.md"

## 列出 FreqAI 模型

使用 `list-freqaimodels` 子命令查看所有可用的 FreqAI 模型。

此子命令对于发现加载 FreqAI 模型时的环境问题很有用：包含错误且加载失败的模型模块会以红色显示（LOAD FAILED），而名称重复的模型会以黄色显示（DUPLICATE NAME）。

--8<-- "commands/list-freqaimodels.md"

## 列出交易所

使用 `list-exchanges` 子命令查看机器人可用的交易所。

--8<-- "commands/list-exchanges.md"

示例：查看机器人可用的交易所：

```
$ freqtrade list-exchanges
Exchanges available for Freqtrade:
Exchange name       Supported    Markets                 Reason
------------------  -----------  ----------------------  ------------------------------------------------------------------------
binance             Official     spot, isolated futures
bitmart             Official     spot
bybit                            spot, isolated futures
gate                Official     spot, isolated futures
htx                 Official     spot
huobi                            spot
kraken              Official     spot
okx                 Official     spot, isolated futures
```

!!! info ""
    为清晰起见输出已缩减 - 支持的和可用的交易所可能会随时间变化。

!!! Note "缺少可选功能的交易所"
    带有 "missing opt:" 的值可能需要特殊配置（例如，如果 `fetchTickers` 缺失，则使用 orderbook）- 但理论上应该可以工作（尽管我们不能保证它们一定可以）。

示例：查看 ccxt 库支持的所有交易所（包括"不良"交易所，即已知无法与 Freqtrade 配合使用的交易所）

```
$ freqtrade list-exchanges -a
All exchanges supported by the ccxt library:
Exchange name       Valid    Supported    Markets                 Reason
------------------  -------  -----------  ----------------------  ---------------------------------------------------------------------------------
binance             True     Official     spot, isolated futures
bitflyer            False                 spot                    missing: fetchOrder. missing opt: fetchTickers.
bitmart             True     Official     spot
bybit               True                  spot, isolated futures
gate                True     Official     spot, isolated futures
htx                 True     Official     spot
kraken              True     Official     spot
okx                 True     Official     spot, isolated futures
```

!!! info ""
    输出已缩减 - 支持的和可用的交易所可能会随时间变化。

## 列出时间框架

使用 `list-timeframes` 子命令查看交易所可用的时间框架列表。

--8<-- "commands/list-timeframes.md"

* 示例：查看在配置文件中设置的 'binance' 交易所的时间框架：

```
$ freqtrade list-timeframes -c config_binance.json
...
Timeframes available for the exchange `binance`: 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M
```

* 示例：列出 Freqtrade 可用的交易所并打印每个交易所支持的时间框架：
```
$ for i in `freqtrade list-exchanges -1`; do freqtrade list-timeframes --exchange $i; done
```

## 列出交易对/列出市场

`list-pairs` 和 `list-markets` 子命令允许查看交易所上可用的交易对/市场。

交易对是市场符号中基础货币部分和报价货币部分之间带有 '/' 字符的市场。
例如，在 'ETH/BTC' 交易对中，'ETH' 是基础货币，而 'BTC' 是报价货币。

对于 Freqtrade 交易的交易对，其报价货币由 `stake_currency` 配置项的值定义。

你可以使用这些子命令打印任何交易对/市场的信息 - 并且可以使用 `--quote BTC` 按报价货币过滤输出，或使用 `--base ETH` 选项按基础货币过滤输出。

这些子命令具有相同的用法和相同的可用选项集：

--8<-- "commands/list-pairs.md"

默认情况下，只显示活跃的交易对/市场。活跃的交易对/市场是那些当前可以在交易所上交易的。
你可以使用 `-a`/`-all` 选项查看所有交易对/市场的列表，包括非活跃的。
如果市场的最小可交易价格非常小，即小于 `1e-11`（`0.00000000001`），交易对可能被列为不可交易。

打印输出中的交易对/市场按其符号字符串排序。

### 示例

* 打印交易所上报价货币为 USD 的活跃交易对列表，使用默认配置文件中指定的交易所（即 "Binance" 交易所），以 JSON 格式输出：

```
$ freqtrade list-pairs --quote USD --print-json
```

* 打印 `config_binance.json` 配置文件中指定的交易所（即 "Binance" 交易所）上的所有交易对，基础货币为 BTC 或 ETH，报价货币为 USDT 或 USD，以人类可读列表形式输出并附带摘要：

```
$ freqtrade list-pairs -c config_binance.json --all --base BTC ETH --quote USDT USD --print-list
```

* 以表格格式打印 "Kraken" 交易所上的所有市场：

```
$ freqtrade list-markets --exchange kraken --all
```

## 测试交易对列表

使用 `test-pairlist` 子命令测试[动态交易对列表](plugins.md#pairlists)的配置。

需要在配置中指定 `pairlists` 属性。
可用于生成在回测/超参数优化期间使用的静态交易对列表。

--8<-- "commands/test-pairlist.md"

### 示例

使用[动态交易对列表](plugins.md#pairlists)时显示白名单。

```
freqtrade test-pairlist --config config.json --quote USDT BTC
```

## 转换数据库

`freqtrade convert-db` 可用于将你的数据库从一个系统转换到另一个系统（sqlite -> postgres、postgres -> 其他 postgres），迁移所有交易、订单和交易对锁。

请参阅[相关文档](advanced-setup.md#use-a-different-database-system)了解不同数据库系统的要求。

--8<-- "commands/convert-db.md"

!!! Warning
    请确保只在空的目标数据库上使用此命令。Freqtrade 将执行常规迁移，但如果已存在条目则可能会失败。

## 网络服务器模式

!!! Warning "实验性"
    网络服务器模式是一个实验性模式，旨在提高回测和策略开发的生产力。
    可能仍存在 bug - 如果你碰巧遇到这些问题，请将它们作为 GitHub issue 报告，谢谢。

以网络服务器模式运行 freqtrade。
Freqtrade 将启动网络服务器，允许 FreqUI 启动和控制回测过程。
这样做的优势是在回测运行之间不需要重新加载数据（只要时间框架和时间范围保持不变）。
FreqUI 还会显示回测结果。

--8<-- "commands/webserver.md"

### 网络服务器模式 - Docker

你也可以通过 Docker 使用网络服务器模式。
启动一次性容器需要显式配置端口，因为默认情况下端口不会被暴露。
你可以使用 `docker compose run --rm -p 127.0.0.1:8080:8080 freqtrade webserver` 来启动一个一次性容器，停止后会被自动删除。这假设端口 8080 仍然可用且没有其他机器人在该端口上运行。

或者，你可以重新配置 docker-compose 文件来更新命令：

``` yml
    command: >
      webserver
      --config /freqtrade/user_data/config.json
```

现在你可以使用 `docker compose up` 来启动网络服务器。
这假设配置已启用并为 Docker 配置了网络服务器（监听端口 = `0.0.0.0`）。

!!! Tip
    如果你想启动实盘或模拟运行机器人，不要忘记将命令重置回 trade 命令。

## 显示之前的回测结果

允许你显示之前的回测结果。
添加 `--show-pair-list` 会输出一个已排序的交易对列表，你可以轻松复制/粘贴到配置中（排除表现不好的交易对）。

??? Warning "策略过拟合"
    仅使用盈利的交易对可能导致策略过拟合，这在未来数据上可能表现不佳。请确保在使用真金白银之前在模拟运行中充分测试你的策略。

--8<-- "commands/backtesting-show.md"

## 详细回测分析

高级回测结果分析。

更多详情请参阅[回测分析](advanced-backtesting.md#analyze-the-buyentry-and-sellexit-tags)章节。

--8<-- "commands/backtesting-analysis.md"

## 列出 Hyperopt 结果

你可以使用 `hyperopt-list` 子命令列出 Hyperopt 模块之前评估过的超参数优化轮次。

--8<-- "commands/hyperopt-list.md"

!!! Note
    `hyperopt-list` 将自动使用最新的可用超参数优化结果文件。
    你可以使用 `--hyperopt-filename` 参数覆盖此行为，并指定另一个可用的文件名（不含路径！）。

### 示例

列出所有结果，在最后打印最佳结果的详情：
```
freqtrade hyperopt-list
```

仅列出盈利的轮次。不打印最佳轮次的详情，以便列表可以在脚本中迭代：
```
freqtrade hyperopt-list --profitable --no-details
```

## 显示 Hyperopt 结果详情

你可以使用 `hyperopt-show` 子命令显示 Hyperopt 模块之前评估过的任何超参数优化轮次的详情。

--8<-- "commands/hyperopt-show.md"

!!! Note
    `hyperopt-show` 将自动使用最新的可用超参数优化结果文件。
    你可以使用 `--hyperopt-filename` 参数覆盖此行为，并指定另一个可用的文件名（不含路径！）。

### 示例

打印第 168 轮的详情（轮次编号由 `hyperopt-list` 子命令或 Hyperopt 本身在超参数优化运行期间显示）：

```
freqtrade hyperopt-show -n 168
```

以 JSON 格式打印最后一个最佳轮次的详情（即所有轮次中最好的）：

```
freqtrade hyperopt-show --best -n -1 --print-json --no-header
```

## 显示交易

将选定的（或所有的）数据库中的交易打印到屏幕上。

--8<-- "commands/show-trades.md"

### 示例

以 JSON 格式打印 ID 为 2 和 3 的交易

``` bash
freqtrade show-trades --db-url sqlite:///tradesv3.sqlite --trade-ids 2 3 --print-json
```

## 策略更新器

将列出的策略或策略文件夹中的所有策略更新为 v3 兼容版本。
如果命令运行时不带 --strategy-list，则策略文件夹内的所有策略都将被转换。
你的原始策略将保留在 `user_data/strategies_orig_updater/` 目录中。

!!! Warning "转换结果"
    策略更新器将采用"尽力而为"的方式工作。请自行做好尽职调查，验证转换结果。
    我们还建议运行 Python 格式化工具（例如 `ruff format`）来合理地格式化结果。

--8<-- "commands/strategy-updater.md"
