# ![freqtrade](https://raw.githubusercontent.com/freqtrade/freqtrade/develop/docs/assets/freqtrade_poweredby.svg)

[![Freqtrade CI](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml)
[![DOI](https://joss.theoj.org/papers/10.21105/joss.04864/status.svg)](https://doi.org/10.21105/joss.04864)
[![codecov](https://codecov.io/gh/freqtrade/freqtrade/branch/develop/graph/badge.svg?token=AD5BG3ATKI)](https://codecov.io/gh/freqtrade/freqtrade)
[![Documentation](https://readthedocs.org/projects/freqtrade/badge/)](https://www.freqtrade.io)
[![Discord Server](https://img.shields.io/badge/Freqtrade_Discord-4E4E4E?logo=discord)](https://discord.gg/p7nuUNVfP7)

Freqtrade 是一个免费开源的加密货币交易机器人，使用 Python 编写。它支持所有主流交易所，并可通过 Telegram 或 Web UI 进行控制。它包含回测、绘图和资金管理工具，以及通过机器学习进行策略优化的功能。

![freqtrade](https://raw.githubusercontent.com/freqtrade/freqtrade/develop/docs/assets/freqtrade-screenshot.png)

## 免责声明

本软件仅用于教育目的。请勿拿您承受不起损失的资金去冒险。使用本软件的风险由您自行承担。作者及所有关联方对您的交易结果不承担任何责任。

请始终先在模拟交易（Dry-Run）模式下运行交易机器人，在您了解其工作原理以及预期的盈亏情况之前，不要投入真实资金。

我们强烈建议您具备编程和 Python 知识。请不要犹豫，阅读源代码并理解本机器人的运行机制。

## 支持的交易所

请阅读[交易所特定说明](https://www.freqtrade.io/en/stable/exchanges/)，了解每个交易所可能需要的特殊配置。

### 支持的现货交易所

- [X] [Binance](https://www.binance.com/)
- [X] [BingX](https://bingx.com/invite/0EM9RX)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Bitmart](https://bitmart.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [HTX](https://www.htx.com/)
- [X] [Hyperliquid](https://hyperliquid.xyz/)（去中心化交易所，即 DEX）
- [X] [Kraken](https://kraken.com/)
- [X] [OKX](https://okx.com/)
- [X] [MyOKX](https://okx.com/)（OKX EEA）
- [ ] [以及更多潜在支持的交易所](https://github.com/ccxt/ccxt/)。_（我们无法保证它们都能正常工作）_

### 支持的期货交易所

- [X] [Binance](https://www.binance.com/)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [Hyperliquid](https://hyperliquid.xyz/)（去中心化交易所，即 DEX）
- [X] [OKX](https://okx.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Kraken](https://www.kraken.com/features/futures)

请务必阅读[交易所特定说明](https://www.freqtrade.io/en/stable/exchanges/)以及[杠杆交易](https://www.freqtrade.io/en/stable/leverage/)文档后再开始使用。

### 社区验证

经社区确认可正常使用的交易所：

- [X] [Bitvavo](https://bitvavo.com/)
- [X] [Kucoin](https://www.kucoin.com/)

## 文档

我们邀请您阅读机器人文档，以确保您理解机器人的工作方式。

请在 [freqtrade 官网](https://www.freqtrade.io)查阅完整文档。

## 功能特性

- [x] **基于 Python 3.11+**：可在任何操作系统上运行——Windows、macOS 和 Linux。
- [x] **数据持久化**：通过 sqlite 实现数据持久化。
- [x] **模拟交易（Dry-run）**：无需投入真实资金即可运行机器人。
- [x] **回测**：模拟运行您的买卖策略。
- [x] **机器学习策略优化**：使用机器学习和真实交易所数据来优化您的买卖策略参数。
- [X] **自适应预测建模**：使用 FreqAI 构建智能策略，通过自适应机器学习方法进行自我训练以适应市场。[了解更多](https://www.freqtrade.io/en/stable/freqai/)
- [x] **加密货币白名单**：选择您想要交易的加密货币或使用动态白名单。
- [x] **加密货币黑名单**：选择您想要避免的加密货币。
- [x] **内置 Web UI**：内置 Web 用户界面来管理您的机器人。
- [x] **Telegram 管理**：通过 Telegram 管理机器人。
- [x] **法币显示盈亏**：以法定货币显示您的盈亏。
- [x] **绩效状态报告**：提供当前交易的绩效状态报告。

## 快速开始

请参阅 [Docker 快速入门文档](https://www.freqtrade.io/en/stable/docker_quickstart/)了解如何快速开始。

如需了解其他（原生）安装方式，请参阅[安装文档页面](https://www.freqtrade.io/en/stable/installation/)。

## 基本用法

### 机器人命令

```
usage: freqtrade [-h] [-V]
                 {trade,create-userdir,new-config,show-config,new-strategy,download-data,convert-data,convert-trade-data,trades-to-ohlcv,list-data,backtesting,backtesting-show,backtesting-analysis,edge,hyperopt,hyperopt-list,hyperopt-show,list-exchanges,list-markets,list-pairs,list-strategies,list-hyperoptloss,list-freqaimodels,list-timeframes,show-trades,test-pairlist,convert-db,install-ui,plot-dataframe,plot-profit,webserver,strategy-updater,lookahead-analysis,recursive-analysis}
                 ...

Free, open source crypto trading bot

positional arguments:
  {trade,create-userdir,new-config,show-config,new-strategy,download-data,convert-data,convert-trade-data,trades-to-ohlcv,list-data,backtesting,backtesting-show,backtesting-analysis,edge,hyperopt,hyperopt-list,hyperopt-show,list-exchanges,list-markets,list-pairs,list-strategies,list-hyperoptloss,list-freqaimodels,list-timeframes,show-trades,test-pairlist,convert-db,install-ui,plot-dataframe,plot-profit,webserver,strategy-updater,lookahead-analysis,recursive-analysis}
    trade               Trade module.
    create-userdir      Create user-data directory.
    new-config          Create new config
    show-config         Show resolved config
    new-strategy        Create new strategy
    download-data       Download backtesting data.
    convert-data        Convert candle (OHLCV) data from one format to
                        another.
    convert-trade-data  Convert trade data from one format to another.
    trades-to-ohlcv     Convert trade data to OHLCV data.
    list-data           List downloaded data.
    backtesting         Backtesting module.
    backtesting-show    Show past Backtest results
    backtesting-analysis
                        Backtest Analysis module.
    hyperopt            Hyperopt module.
    hyperopt-list       List Hyperopt results
    hyperopt-show       Show details of Hyperopt results
    list-exchanges      Print available exchanges.
    list-markets        Print markets on exchange.
    list-pairs          Print pairs on exchange.
    list-strategies     Print available strategies.
    list-hyperoptloss   Print available hyperopt loss functions.
    list-freqaimodels   Print available freqAI models.
    list-timeframes     Print available timeframes for the exchange.
    show-trades         Show trades.
    test-pairlist       Test your pairlist configuration.
    convert-db          Migrate database to different system
    install-ui          Install FreqUI
    plot-dataframe      Plot candles with indicators.
    plot-profit         Generate plot showing profits.
    webserver           Webserver module.
    strategy-updater    updates outdated strategy files to the current version
    lookahead-analysis  Check for potential look ahead bias.
    recursive-analysis  Check for potential recursive formula issue.

options:
  -h, --help            show this help message and exit
  -V, --version         show program's version number and exit
```

### Telegram RPC 命令

Telegram 不是必需的。但它是控制机器人的绝佳方式。更多详情和完整命令列表请参阅[文档](https://www.freqtrade.io/en/stable/telegram-usage/)

- `/start`：启动交易。
- `/stop`：停止交易。
- `/stopentry`：停止开新仓。
- `/status <trade_id>|[table]`：列出所有或指定的未平仓交易。
- `/profit [<n>]`：列出所有已完成交易在最近 n 天的累计利润。
- `/profit_long [<n>]`：列出所有已完成多头交易在最近 n 天的累计利润。
- `/profit_short [<n>]`：列出所有已完成空头交易在最近 n 天的累计利润。
- `/forceexit <trade_id>|all`：立即退出指定交易（忽略 `minimum_roi`）。
- `/fx <trade_id>|all`：`/forceexit` 的别名
- `/performance`：显示每笔已完成交易按交易对分组的绩效
- `/balance`：显示各币种的账户余额。
- `/daily <n>`：显示最近 n 天的每日盈亏。
- `/help`：显示帮助信息。
- `/version`：显示版本号。


## 开发分支

本项目目前设置了两个主要分支：

- `develop` - 此分支经常包含新功能，但也可能包含破坏性更改。我们尽力保持此分支的稳定性。
- `stable` - 此分支包含最新的稳定版本。此分支通常经过充分测试。
- `feat/*` - 这些是功能分支，正在积极开发中。除非您想测试特定功能，否则请勿使用这些分支。

## 支持

### 帮助 / Discord

对于文档未涵盖的任何问题、关于机器人的更多信息，或者只是想与志同道合的人交流，我们欢迎您加入 Freqtrade 的 [Discord 服务器](https://discord.gg/p7nuUNVfP7)。

### [Bug / 问题](https://github.com/freqtrade/freqtrade/issues?q=is%3Aissue)

如果您在机器人中发现了 bug，请先[搜索问题追踪器](https://github.com/freqtrade/freqtrade/issues?q=is%3Aissue)。如果尚未被报告，请[创建新的 issue](https://github.com/freqtrade/freqtrade/issues/new/choose)，并确保遵循模板指南，以便团队能尽快为您提供帮助。

对于每个创建的 [issue](https://github.com/freqtrade/freqtrade/issues/new/choose)，请及时跟进，并在达成共识后标记满意或提醒关闭 issue。

--请遵守 GitHub 的[社区准则](https://docs.github.com/en/site-policy/github-terms/github-community-code-of-conduct)--

### [功能请求](https://github.com/freqtrade/freqtrade/labels/enhancement)

您有改进机器人的好想法想分享吗？请先搜索该功能是否[已经被讨论过](https://github.com/freqtrade/freqtrade/labels/enhancement)。如果尚未被请求，请[创建新的请求](https://github.com/freqtrade/freqtrade/issues/new/choose)，并确保遵循模板指南，以免被埋没在 bug 报告中。

### [Pull Requests](https://github.com/freqtrade/freqtrade/pulls)

觉得机器人缺少某个功能？我们欢迎您的 Pull Request！

请阅读[贡献文档](https://github.com/freqtrade/freqtrade/blob/develop/CONTRIBUTING.md)，了解提交 Pull Request 前的要求。

贡献不一定需要编码——也许可以从改进文档开始？标记为 [good first issue](https://github.com/freqtrade/freqtrade/labels/good%20first%20issue) 的问题是很好的首次贡献选择，有助于您熟悉代码库。

**注意**：在开始任何重大新功能开发之前，*请先创建一个 issue 描述您的计划*，或在 [Discord](https://discord.gg/p7nuUNVfP7) 上与我们讨论（请使用 #dev 频道）。这将确保感兴趣的各方能够对该功能提供有价值的反馈，并让其他人知道您正在进行此项工作。

**重要**：请始终基于 `develop` 分支创建您的 PR，而不是 `stable` 分支。

## 系统要求

### 时钟同步

系统时钟必须准确，需频繁与 NTP 服务器同步，以避免与交易所通信时出现问题。

### 最低硬件要求

运行此机器人，我们建议您使用至少满足以下配置的云服务器实例：

- 最低（建议）系统要求：2GB 内存、1GB 磁盘空间、2vCPU

### 软件要求

- [Python >= 3.11](http://docs.python-guide.org/en/latest/starting/installation/)
- [pip](https://pip.pypa.io/en/stable/installing/)
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [TA-Lib](https://ta-lib.github.io/ta-lib-python/)
- [virtualenv](https://virtualenv.pypa.io/en/stable/installation.html)（推荐）
- [Docker](https://www.docker.com/products/docker)（推荐）
