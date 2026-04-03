# Freqtrade 常见问题

## 支持的市场

Freqtrade 支持现货交易，以及部分选定交易所的期货交易。请参阅 [文档首页](index.md#supported-futures-exchanges) 获取最新的支持交易所列表。

### 我的机器人可以开空头仓位吗？

Freqtrade 可以在期货市场开空头仓位。
这需要策略针对此进行编写——并且在配置中设置 `"trading_mode": "futures"`。
请务必先阅读 [相关文档页面](leverage.md)。

在现货市场中，某些情况下你可以使用杠杆代币，它们反映了反向交易对（例如 BTCUP/USD、BTCDOWN/USD、ETHBULL/USD、ETHBEAR/USD 等），这些代币可以用 Freqtrade 交易。

### 我的机器人可以交易期权或期货吗？

部分选定交易所支持期货交易。请参阅 [文档首页](index.md#supported-futures-exchanges) 获取最新的支持交易所列表。

## 新手提示与技巧

* 当你编写策略和 hyperopt 文件时，应该使用合适的代码编辑器，如 VSCode 或 PyCharm。好的代码编辑器会提供语法高亮和行号，方便找到语法错误（Freqtrade 在启动时很可能会指出这些错误）。

## Freqtrade 常见问题

### Freqtrade 能否同时在同一交易对上开多个仓位？

不能。Freqtrade 一次只会在每个交易对上开一个仓位。
但你可以使用 [`adjust_trade_position()` 回调](strategy-callbacks.md#adjust-trade-position) 来调整已开仓位。

回测提供了 `--eps` 选项来实现此功能——但这仅用于突出"隐藏"信号，在实盘中不起作用。

### Freqtrade 支持沙盒账户吗？

不支持，但你可以使用模拟运行模式来模拟交易而不冒真实资金的风险。

沙盒市场是独立的模拟市场——不适合在真实环境中测试你的策略。
这些市场通常具有不同的订单簿、流动性和交易行为（通常参与者很少）——这使得它们不适合对你的策略进行真实测试。

### 机器人无法启动

使用 `freqtrade trade --config config.json` 运行机器人时显示输出 `freqtrade: command not found`。

这可能由以下原因引起：

* 虚拟环境未激活。
  * 运行 `source .venv/bin/activate` 来激活虚拟环境。
* 安装未成功完成。
  * 请查看 [安装文档](installation.md)。

### 机器人启动了，但处于 STOPPED 模式

确保你在 config.json 中将 `initial_state` 配置选项设置为 `"running"`。

### 我已经等了 5 分钟，为什么机器人还没有进行任何交易？

* 根据入场策略、白名单币种数量、市场状况等因素，可能需要数小时甚至数天才能找到一个好的交易入场位置。请耐心等待！

* 回测会大致告诉你预期有多少笔交易——但不保证它们会均匀分布——所以你可能一天有 20 笔交易，而接下来的一周一笔都没有。

* 这可能是配置错误导致的。最好检查日志，日志通常会告诉你机器人是否只是没有收到买入信号（只有心跳消息），或者是否出了问题（日志中有错误/异常）。

### 我已经进行了 12 笔交易，为什么总利润是负的？

我理解你的失望，但遗憾的是 12 笔交易远远不够说明任何问题。如果你运行回测，你可以看到当前算法确实让你处于盈利状态，但那是在数千笔交易之后，即使如此，你仍然会在某些你交易了数十甚至数百次的特定币种上亏损。我们当然一直致力于改进机器人，但它*始终*是一种赌博，应该让你每月获得适度的盈利，但从少数几笔交易中你无法得出什么结论。

### 我想修改配置。我能在不关闭机器人的情况下修改吗？

可以。你可以编辑配置并使用 `/reload_config` 命令重新加载配置。机器人将停止、重新加载配置和策略，然后使用新的配置和策略重新启动。

### 为什么我的机器人没有卖出它买入的全部数量？

这被称为"币尘"，在所有交易所都可能发生。
这是因为许多交易所从"接收货币"中扣除手续费——所以你买了 100 COIN，但你只得到 99.9 COIN。
由于 COIN 以整数手交易（1 COIN 步进），你无法卖出 0.9 COIN（或 99.9 COIN）——你需要向下取整到 99 COIN。

这不是机器人的问题，手动交易时也会发生。

虽然 freqtrade 可以处理这个问题（它会卖出 99 COIN），但手续费通常低于最小可交易手数（你只能交易整数个 COIN，不能交易 0.9 COIN）。
将币尘（0.9 COIN）留在交易所通常是合理的，因为下次 freqtrade 买入 COIN 时，它会消耗掉剩余的小额余额，这次卖出它买入的全部，从而慢慢减少币尘余额（尽管很可能永远不会精确到 0）。

在可能的情况下（例如在 Binance 上），使用交易所专用的手续费货币可以解决这个问题。
在 Binance 上，只需在你的账户中持有 BNB，并在个人资料中启用"使用 BNB 支付手续费"即可。你的 BNB 余额会慢慢减少（因为用来支付手续费）——但你将不再遇到币尘问题（Freqtrade 会在利润计算中包含手续费）。
其他交易所不提供这种功能，这只是你必须接受的事情，或者转移到其他交易所。

### 我向交易所充值了更多资金，但机器人没有识别到

Freqtrade 会在需要时更新交易所余额（下单之前）。
RPC 调用（Telegram 的 `/balance`，API 调用 `/balance`）最多每小时触发一次更新。

如果启用了 `adjust_trade_position`（且机器人有符合仓位调整条件的未平仓交易）——那么钱包将每小时刷新一次。
要强制立即更新，你可以使用 `/reload_config`——这将重启机器人。

### 我想使用未完成的 K 线

Freqtrade 不会向策略提供未完成的 K 线。使用未完成的 K 线会导致重绘，从而导致具有"幽灵"买入的策略，这些买入无法在回测中验证，也无法在发生后进行验证。

你可以通过使用 [dataprovider](strategy-customization.md#orderbookpair-maximum) 的订单簿或行情方法来使用"当前"市场数据——但这些在回测期间不可用。

### 有没有设置可以只退出持有的交易而不进行新的入场？

你可以在 Telegram 中使用 `/stopentry` 命令来阻止未来的交易入场，然后使用 `/forceexit all`（卖出所有未平仓交易）。

### 我卖掉了机器人的资金，现在日志中出现了错误

Freqtrade 假设它开立的交易仅通过机器人管理。
如果你（意外地）卖掉了机器人的资金，freqtrade 将尝试通过在交易所上重新查找订单来恢复。

这是一种尽力而为的方法，并不适用于所有情况，特别是当使用 freqtrade 不支持的订单类型（OCO、冰山订单等）时，或者处理旧交易（交易所不再提供完整订单信息）时。
确切的限制因交易所而异——详细信息通常记录在交易所的 API 文档中。

### 我想在同一台机器上运行多个机器人

请查看 [高级设置文档页面](advanced-setup.md#running-multiple-instances-of-freqtrade)。

### 启动机器人时出现 "Impossible to load Strategy" 错误

当机器人无法加载策略时会显示此错误消息。
通常，你可以使用 `freqtrade list-strategies` 列出所有可用策略。
此命令的输出还将包含一个状态列，显示策略是否可以被加载。

请检查以下内容：

* 你使用的策略名称是否正确？策略名称区分大小写，必须对应策略类名（不是文件名！）。
* 策略是否在 `user_data/strategies` 目录中，且文件扩展名为 `.py`？
* 在此错误之前机器人是否显示了其他警告？也许你缺少策略所需的某些依赖——这会在日志中高亮显示。
* 使用 Docker 的情况下——策略目录是否正确挂载（检查 docker-compose 文件的 volumes 部分）？

### 日志中出现 "Missing data fillup" 消息

此消息只是一个警告，表示最新的 K 线中有缺失的 K 线。
根据交易所的不同，这可能表示该交易对在你使用的时间周期内没有交易——交易所仅返回有成交量的 K 线。
在低成交量的交易对上，这是相当常见的情况。

如果这发生在交易对列表中的所有交易对上，这可能表示交易所最近出现了宕机。请检查你的交易所的公共渠道了解详情。

无论原因如何，Freqtrade 都会用"空" K 线来填充这些 K 线，其中开盘价、最高价、最低价和收盘价设置为前一根 K 线的收盘价——成交量为空。在图表中，这看起来像一个 `_`——与交易所通常表示 0 成交量 K 线的方式一致。

### 出现 "Price jump between 2 candles detected" 消息

此消息是一个警告，表示 K 线之间出现了 > 30% 的价格跳跃。
这可能是交易对停止交易，并发生了某种代币转换的迹象（例如 2021 年的 COCOS——价格从 0.0000154 跳到 0.01621）。
此消息通常伴随着 ["Missing data fillup"](#im-getting-missing-data-fillup-messages-in-the-log)——因为这类交易对的交易通常会暂停一段时间。

### 我想重置机器人的数据库

要重置机器人的数据库，你可以删除数据库（默认为 `tradesv3.sqlite` 或 `tradesv3.dryrun.sqlite`），或者通过 `--db-url` 使用不同的数据库 URL（例如 `sqlite:///mynewdatabase.sqlite`）。

### 日志中出现 "Outdated history for pair xxx"

机器人试图告诉你它获取到了一根过时的最新 K 线（不是最后一根完整的 K 线）。
因此，Freqtrade 不会为此交易对入场交易——因为基于旧信息交易通常不是期望的行为。

此警告可能指向以下问题之一：

* 交易所宕机 -> 检查你的交易所状态页面/博客/推特了解详情。
* 系统时间错误 -> 确保你的系统时间正确。
* 交易量极低的交易对 -> 在交易所网页上查看该交易对，查看你策略使用的时间周期。如果该交易对在某些 K 线中没有成交量（通常显示为"成交量 0"的柱子和一个 "_" 作为 K 线），则该交易对在此时间周期内没有任何交易。应尽量避免使用这些交易对，因为它们可能导致订单成交问题。
* API 问题 -> API 返回错误数据（这里只是为了完整性，支持的交易所不应该发生这种情况）。

### 日志中出现 "Couldn't reuse watch for xxx" 消息

这是一条信息性消息，表示机器人尝试使用来自 websocket 的 K 线，但交易所没有提供正确的信息。
如果 websocket 连接中断，或者该交易对在你使用的时间周期内没有发生任何交易，都可能发生这种情况。

Freqtrade 会优雅地处理这种情况，回退到 REST API。
虽然这会使迭代稍微慢一些（由于 REST API 调用）——但不会对机器人的运行造成任何问题。

### 出现 "Exchange XXX does not support market orders." 消息且无法运行策略

正如消息所说，你的交易所不支持市价单，而你将某个 [订单类型](configuration.md/#understand-order_types) 设置为了 "market"。你的策略可能是针对其他交易所编写的，将 "stoploss" 订单设置为 "market"，这对于大多数支持市价单的交易所来说是正确的、可取的（但不适用于 Gate.io）。

要修复此问题，在策略中重新定义订单类型，使用 "limit" 替代 "market"：

``` python
    order_types = {
        ...
        "stoploss": "limit",
        ...
    }
```

如果订单类型定义在你的自定义配置中而非策略中，则应在配置文件中应用同样的修改。

### 我尝试启动实盘机器人，但出现 API 权限错误

像 `Invalid API-key, IP, or permissions for action` 这样的错误意味着确实如其所述。
你的 API 密钥可能无效（复制/粘贴错误？检查配置中的前导/尾随空格）、已过期，或者你运行机器人的 IP 未在交易所的 API 控制台中启用。
通常需要 "Spot Trading"（或你使用的交易所中的等效权限）权限。
期货通常需要专门启用。

### 如何在机器人日志中搜索内容？

默认情况下，机器人将其日志写入 stderr 流。之所以这样实现，是为了让你可以轻松地将机器人的诊断消息与回测、Edge 和 Hyperopt 结果、其他各种 Freqtrade 实用子命令的输出以及你可能在策略中插入的自定义 `print()` 输出分开。因此，如果你需要使用 grep 实用工具搜索日志消息，你需要将 stderr 重定向到 stdout 并忽略 stdout。

* 在 Unix shell 中，这通常可以简单地执行如下：
```shell
$ freqtrade --some-options 2>&1 >/dev/null | grep 'something'
```
（注意，`2>&1` 和 `>/dev/null` 应按此顺序编写）

* Bash 解释器还支持所谓的进程替换语法，你可以用它在日志中搜索字符串：
```shell
$ freqtrade --some-options 2> >(grep 'something') >/dev/null
```
或
```shell
$ freqtrade --some-options 2> >(grep -v 'something' 1>&2)
```

* 你也可以使用 `--logfile` 选项将 Freqtrade 日志消息的副本写入文件：
```shell
$ freqtrade --logfile /path/to/mylogfile.log --some-options
```
然后搜索：
```shell
$ cat /path/to/mylogfile.log | grep 'something'
```
或者在机器人运行和日志文件增长时实时搜索：
```shell
$ tail -f /path/to/mylogfile.log | grep 'something'
```
从单独的终端窗口执行。

在 Windows 上，Freqtrade 也支持 `--logfile` 选项，你可以使用 `findstr` 命令在日志中搜索感兴趣的字符串：
```
> type \path\to\mylogfile.log | findstr "something"
```

## Hyperopt 模块

### 为什么 Freqtrade 没有 GPU 支持？

首先，大多数指标库没有 GPU 支持——因此 GPU 对指标计算的益处很小。
GPU 改进仅适用于 pandas 原生计算——或你自己编写的计算。

GPU 仅擅长处理数字（浮点运算）。
对于 hyperopt，我们既需要数字运算（找到下一组参数）也需要运行 Python 代码（运行回测）。
因此，GPU 对于 hyperopt 的大部分工作并不太适合。

使用 GPU 的收益因此会非常小——不值得为添加 GPU 支持引入的复杂性。

然而，如果你认为必须使用 GPU 加速指标，没有什么能阻止你在策略中使用 GPU 加速指标——不过你可能会对其带来的微小收益（与复杂性相比）感到失望。

### 我需要多少个 epoch 才能获得好的 Hyperopt 结果？

默认情况下，不带 `-e`/`--epochs` 命令行选项调用的 Hyperopt 只会运行 100 个 epoch，即对你的触发器、守卫等进行 100 次评估。这太少了，不足以找到出色的结果（除非你非常幸运），所以你可能需要运行 10000 次甚至更多。但计算将花费很长时间。

由于 hyperopt 使用贝叶斯搜索，运行太多 epoch 可能不会产生更好的结果。

因此建议每次运行 500-1000 个 epoch，反复运行直到总计达到至少 10000 个 epoch（或你对结果满意为止）。你可以通过查看结果来判断——如果机器人持续发现更好的策略，最好继续运行。

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --strategy SampleStrategy -e 1000
```

### 为什么运行 hyperopt 需要很长时间？

* 使用 Hyperopt 发现出色的策略需要时间。学习 www.freqtrade.io，阅读 Freqtrade 文档页面，加入 Freqtrade [Discord 社区](https://discord.gg/p7nuUNVfP7)。在你耐心等待世界上最先进、免费的加密货币机器人为你量身定制可能的黄金策略时。

* 如果你想知道为什么做 1000 个 epoch 可能需要从 20 分钟到几天，以下是一些解答：

此回答写于版本 0.15.1 发布期间，当时我们有：

* 8 个触发器
* 9 个守卫：假设我们从每个守卫中评估 10 个值
* 1 个止损计算：假设我们也想评估 10 个值

以下计算仍然非常粗略且不太精确，但会给出大致概念。仅凭这些触发器和守卫就已经有 8\*10^9\*10 次评估。大约总共 800 亿次评估。你运行了 100,000 次评估？恭喜，你大约完成了搜索空间的 1/100,000，假设机器人从不重复测试相同的参数。

* 运行 1000 个 hyperopt epoch 所需的时间取决于：可用的 CPU、硬盘、内存、时间周期、时间范围、指标设置、指标数量、hyperopt 测试策略的币种数量以及由此产生的交易次数——可能一年 650 笔交易或 100000 笔交易，取决于策略是追求通过少量交易获取大额利润还是多次低利润交易。

示例：一年中 4% 利润 650 次 vs 0.3% 利润每笔交易 10000 次。假设你设置 --timerange 为 365 天。

示例：
`freqtrade --config config.json --strategy SampleStrategy --hyperopt SampleHyperopt -e 1000 --timerange 20190601-20200601`

## 官方渠道

Freqtrade 仅使用以下官方渠道：

* [Freqtrade Discord 服务器](https://discord.gg/p7nuUNVfP7)
* [Freqtrade 文档 (https://freqtrade.io)](https://freqtrade.io)
* [Freqtrade GitHub 组织](https://github.com/freqtrade)

任何与 freqtrade 项目相关的人都不会向你询问你的交易所密钥或任何其他可能让你的资金遭受利用的信息。
如果有人要求你提供你的交易所密钥或向某个随机钱包发送资金，请不要按照这些指示操作。

不遵守这些准则将不是 freqtrade 的责任。

## 支持政策

我们在 [Discord 服务器](https://discord.gg/p7nuUNVfP7) 和 GitHub issues 上为 Freqtrade 提供免费支持。
我们仅支持最新发布版本（例如 2025.8）和当前开发分支（例如 2025.9-dev）。

如果你使用的是旧版本，请按照 [升级说明](updating.md) 操作，看看你的问题是否已经被解决。

## "Freqtrade 代币"

Freqtrade 没有加密货币代币发行。

你在互联网上找到的引用 Freqtrade、FreqAI 或 freqUI 的代币发行必须被视为骗局，它们试图利用 freqtrade 的知名度来为自己谋取不正当利益。
