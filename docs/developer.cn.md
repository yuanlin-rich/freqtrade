# 开发帮助

本页面面向 Freqtrade 的开发者、希望为 Freqtrade 代码库或文档做出贡献的人，以及希望了解所运行应用程序源代码的人。

我们欢迎所有的贡献、错误报告、错误修复、文档改进、功能增强和新想法。我们在 [GitHub](https://github.com) 上[跟踪问题](https://github.com/freqtrade/freqtrade/issues)，同时在 [discord](https://discord.gg/p7nuUNVfP7) 上设有开发频道，您可以在那里提问。

## 文档

文档可在 [https://freqtrade.io](https://www.freqtrade.io/) 获取，每个新功能的 PR 都需要提供相应的文档。

文档的特殊字段（如提示框等）可以在[这里](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)找到。

要在本地测试文档，请使用以下命令：

``` bash
pip install -r docs/requirements-docs.txt
mkdocs serve
```

这将启动一个本地服务器（通常在端口 8000 上），以便您可以查看效果是否符合预期。

## 开发者环境设置

要配置开发环境，您可以使用提供的 [DevContainer](#devcontainer-setup)，或者使用 `setup.sh` 脚本并在提示 "Do you want to install dependencies for dev [y/N]?" 时回答 "y"。
或者（例如，如果您的系统不被 setup.sh 脚本支持），请按照手动安装流程操作，并运行 `pip3 install -r requirements-dev.txt`，然后运行 `pip3 install -e .[all]`。

这将安装所有开发所需的工具，包括 `pytest`、`ruff`、`mypy` 和 `coveralls`。

运行以下命令安装 git hook 脚本：

``` bash
pre-commit install
```

这些 pre-commit 脚本会在每次提交前自动检查您的更改。
如果发现任何格式问题，提交将失败并提示修复。
这减少了不必要的 CI 失败，降低了维护负担，并提高了代码质量。

您可以在需要时使用 `pre-commit run -a` 手动运行检查。

在创建 Pull Request 之前，请同时熟悉我们的[贡献指南](https://github.com/freqtrade/freqtrade/blob/develop/CONTRIBUTING.md)。

### Devcontainer 设置

最快、最简单的入门方式是使用 [VSCode](https://code.visualstudio.com/) 配合 Remote container 扩展。
这使开发者能够在不需要在本地机器上安装任何 freqtrade 特定依赖的情况下，使用所有必需的依赖来启动机器人。

#### Devcontainer 依赖

* [VSCode](https://code.visualstudio.com/)
* [docker](https://docs.docker.com/install/)
* [Remote container 扩展文档](https://code.visualstudio.com/docs/remote)

有关 [Remote container 扩展](https://code.visualstudio.com/docs/remote)的更多信息，请参阅文档。

### 测试

新代码应该有基本的单元测试覆盖。根据功能的复杂程度，审查者可能会要求更深入的单元测试。
如有必要，Freqtrade 团队可以协助并提供编写良好测试的指导（但请不要期望别人为您编写测试）。

#### 如何运行测试

在根目录使用 `pytest` 运行所有可用的测试用例，确认您的本地环境配置正确。

!!! Note "feature branches"
    测试预期在 `develop` 和 `stable` 分支上通过。其他分支可能是正在进行中的工作，测试可能尚未通过。

#### 在测试中检查日志内容

Freqtrade 使用两种主要方法在测试中检查日志内容：`log_has()` 和 `log_has_re()`（用于正则表达式检查，适用于动态日志消息的情况）。
这些方法可从 `conftest.py` 中获取，并可在任何测试模块中导入。

示例检查如下：

``` python
from tests.conftest import log_has, log_has_re

def test_method_to_test(caplog):
    method_to_test()

    assert log_has("This event happened", caplog)
    # Check regex with trailing number ...
    assert log_has_re(r"This dynamic event happened and produced \d+", caplog)

```

### 调试配置

要调试 freqtrade，我们推荐使用 VSCode（配合 Python 扩展），使用以下启动配置（位于 `.vscode/launch.json`）。
具体细节显然会因设置不同而异——但这应该足以帮助您入门。

``` json
{
    "name": "freqtrade trade",
    "type": "debugpy",
    "request": "launch",
    "module": "freqtrade",
    "console": "integratedTerminal",
    "args": [
        "trade",
        // Optional:
        // "--userdir", "user_data",
        "--strategy",
        "MyAwesomeStrategy",
    ]
},
```

命令行参数可以添加在 `"args"` 数组中。
此方法也可以用于调试策略，只需在策略中设置断点即可。

Pycharm 也可以进行类似的设置——使用 `freqtrade` 作为模块名，并将命令行参数设置为 "parameters"。

??? Tip "正确使用虚拟环境"
    当使用虚拟环境时（这是推荐做法），请确保您的编辑器使用正确的虚拟环境，以避免出现问题或 "unknown import" 错误。

    #### Vscode

    您可以在 VSCode 中使用 "Python: Select Interpreter" 命令选择正确的环境——它会显示扩展检测到的环境。
    如果您的环境未被检测到，您也可以手动选择路径。

    #### Pycharm

    在 Pycharm 中，您可以在 "Run/Debug Configurations" 窗口中选择合适的环境。
    ![Pycharm debug configuration](assets/pycharm_debug.png)

!!! Note "启动目录"
    这假设您已检出仓库，并且编辑器在仓库根目录级别启动（即 pyproject.toml 位于仓库的顶层目录）。

## 错误处理

Freqtrade 的所有异常都继承自 `FreqtradeException`。
但不应直接使用这个通用错误类。相反，存在多个专门的子异常。

以下是异常继承层次结构的概览：

```
+ FreqtradeException
|
+---+ OperationalException
|   |
|   +---+ ConfigurationError
|
+---+ DependencyException
|   |
|   +---+ PricingError
|   |
|   +---+ ExchangeError
|       |
|       +---+ TemporaryError
|       |
|       +---+ DDosProtection
|       |
|       +---+ InvalidOrderException
|           |
|           +---+ RetryableOrderError
|           |
|           +---+ InsufficientFundsError
|
+---+ StrategyError
```

---

## 插件

### Pairlists

您有一个很好的新交易对选择算法的想法，想要尝试一下？太好了。
希望您也愿意将其贡献回上游。

无论您的动机是什么——以下内容应该能帮助您开始开发新的 Pairlist Handler。

首先，看一下 [VolumePairList](https://github.com/freqtrade/freqtrade/blob/develop/freqtrade/plugins/pairlist/VolumePairList.py) Handler，最好将此文件复制一份并用您新的 Pairlist Handler 名称命名。

这是一个简单的 Handler，但它可以作为开始开发的良好示例。

接下来，修改 Handler 的类名（最好与模块文件名保持一致）。

基类提供了交易所实例 (`self._exchange`)、pairlist 管理器 (`self._pairlistmanager`)、主配置 (`self._config`)、pairlist 专用配置 (`self._pairlistconfig`) 以及在 pairlist 列表中的绝对位置。

```python
        self._exchange = exchange
        self._pairlistmanager = pairlistmanager
        self._config = config
        self._pairlistconfig = pairlistconfig
        self._pairlist_pos = pairlist_pos
```

!!! Tip
    不要忘记在 `constants.py` 中的 `AVAILABLE_PAIRLISTS` 变量下注册您的 pairlist——否则它将无法被选择。

现在，让我们逐步了解需要实现的方法：

#### Pairlist 配置

Pairlist Handler 链的配置在机器人配置文件中的 `"pairlists"` 元素中完成，它是一个数组，包含链中每个 Pairlist Handler 的配置参数。

按照惯例，`"number_assets"` 用于指定 pairlist 中保留的最大交易对数量。请遵循此约定以确保一致的用户体验。

可以根据需要配置其他参数。例如，`VolumePairList` 使用 `"sort_key"` 来指定排序值——但您可以随意指定任何对您的算法成功运行所必需的参数。

#### short_desc

返回用于 Telegram 消息的描述。

这应该包含 Pairlist Handler 的名称，以及包含资产数量的简短描述。请遵循 `"PairlistName - top/bottom X pairs"` 的格式。

#### gen_pairlist

如果 Pairlist Handler 可以作为链中的首个 Pairlist Handler 使用（定义初始 pairlist，然后由链中所有 Pairlist Handler 处理），则重写此方法。例如 `StaticPairList` 和 `VolumePairList`。

此方法在机器人的每次迭代中调用（仅当 Pairlist Handler 位于第一个位置时）——因此请考虑对计算/网络密集型计算实现缓存。

它必须返回结果 pairlist（然后可能传入 Pairlist Handler 链）。

验证是可选的，父类提供了 `verify_blacklist(pairlist)` 和 `_whitelist_for_active_markets(pairlist)` 来执行默认过滤。如果您将结果限制为一定数量的交易对，请使用这些方法——这样最终结果就不会比预期的短。

#### filter_pairlist

此方法由 pairlist 管理器为链中的每个 Pairlist Handler 调用。

此方法在机器人的每次迭代中调用——因此请考虑对计算/网络密集型计算实现缓存。

它接收一个 pairlist（可以是之前 pairlist 的结果）以及 `tickers`（`get_tickers()` 的预获取版本）。

基类中的默认实现只是对 pairlist 中的每个交易对调用 `_validate_pair()` 方法，但您可以重写它。因此，您应该在 Pairlist Handler 中实现 `_validate_pair()`，或重写 `filter_pairlist()` 来执行其他操作。

如果被重写，它必须返回结果 pairlist（然后可能传入链中的下一个 Pairlist Handler）。

验证是可选的，父类提供了 `verify_blacklist(pairlist)` 和 `_whitelist_for_active_markets(pairlist)` 来执行默认过滤。如果您将结果限制为一定数量的交易对，请使用这些方法——这样最终结果就不会比预期的短。

在 `VolumePairList` 中，它实现了不同的排序方法，并进行早期验证，因此只返回预期数量的交易对。

##### 示例

``` python
    def filter_pairlist(self, pairlist: list[str], tickers: dict) -> List[str]:
        # Generate dynamic whitelist
        pairs = self._calculate_pairlist(pairlist, tickers)
        return pairs
```

### Protections

请先阅读 [Protection 文档](plugins.md#protections) 以了解保护机制。
本指南面向希望开发新保护机制的开发者。

任何保护机制都不应直接使用 datetime，而应使用提供的 `date_now` 变量进行日期计算。这保留了对保护机制进行回测的能力。

!!! Tip "编写新的 Protection"
    最好复制一个现有的 Protection 作为良好的示例。

#### 实现新的 Protection

所有 Protection 实现必须以 `IProtection` 作为父类。
因此，它们必须实现以下方法：

* `short_desc()`
* `global_stop()`
* `stop_per_pair()`

`global_stop()` 和 `stop_per_pair()` 必须返回一个 ProtectionReturn 对象，该对象包含：

* lock pair - 布尔值
* lock until - datetime - 交易对应被锁定到什么时候（将向上取整到下一根新 K 线）
* reason - 字符串，用于日志记录和数据库存储
* lock_side - long、short 或 '*'

`until` 部分应使用提供的 `calculate_lock_end()` 方法来计算。

所有 Protection 都应使用 `"stop_duration"` / `"stop_duration_candles"` 来定义交易对（或所有交易对）应被锁定多长时间。
其内容通过 `self._stop_duration` 提供给每个 Protection。

如果您的保护机制需要回溯期，请使用 `"lookback_period"` / `"lockback_period_candles"` 以保持所有保护机制的一致性。

#### 全局停止 vs. 局部停止

Protection 可以有两种不同的方式在有限时间内停止交易：

* 按交易对（局部）
* 对所有交易对（全局）

##### Protections - 按交易对

实现按交易对方式的 Protection 必须设置 `has_local_stop=True`。
当交易关闭（出场订单完成）时，将调用 `stop_per_pair()` 方法。

##### Protections - 全局保护

这些 Protection 应该跨所有交易对进行评估，因此也会锁定所有交易对的交易（称为全局 PairLock）。
全局保护必须设置 `has_global_stop=True` 才能被评估为全局停止。
当交易关闭（出场订单完成）时，将调用 `global_stop()` 方法。

##### Protections - 计算锁定结束时间

Protection 应该根据它考虑的最后一笔交易来计算锁定结束时间。
这避免了在回溯期长于实际锁定期时重新锁定的情况。

`IProtection` 父类在 `calculate_lock_end()` 中提供了一个辅助方法。

---

## 实现新的交易所（进行中）

!!! Note
    本节是进行中的工作，不是关于如何使用 Freqtrade 测试新交易所的完整指南。

!!! Note
    在运行以下任何测试之前，请确保使用最新版本的 CCXT。
    您可以在激活虚拟环境后运行 `pip install -U ccxt` 来获取最新版本的 ccxt。
    原生 docker 不支持这些测试，但可用的 dev-container 将支持所有必需的操作和可能需要的更改。

CCXT 支持的大多数交易所应该可以直接使用。

如果您需要实现特定的交易所类，这些文件位于 `freqtrade/exchange` 源代码文件夹中。您还需要在 `freqtrade/exchange/__init__.py` 中添加导入，以使加载逻辑知道新的交易所。
我们建议查看现有的交易所实现，以了解可能需要做什么。

!!! Warning
    实现和测试交易所可能需要大量的试错，请牢记这一点。
    您还应该有一定的开发经验，因为这不是一个初学者任务。

要快速测试交易所的公共端点，请在 `tests/exchange_online/conftest.py` 中为您的交易所添加配置，并使用 `pytest --longrun tests/exchange_online/test_ccxt_compat.py` 运行这些测试。
成功完成这些测试是一个很好的基准点（实际上是一个必要条件），但这并不能保证交易所功能的正确性，因为这只测试了公共端点，而没有测试私有端点（如生成订单等）。

还请尝试使用 `freqtrade download-data` 下载较长时间范围（多个月）的数据，并验证数据下载是否正确（无缺失、实际下载了指定的时间范围）。

这些是将交易所列为"支持"或"社区测试"（在主页上列出）的前提条件。
以下是"额外"要求，它们会使交易所更完善（功能完整）——但对于这两个类别来说并非绝对必要。

额外测试/需要完成的步骤：

* 验证 `fetch_ohlcv()` 提供的数据——并最终调整该交易所的 `ohlcv_candle_limit`
* 检查 L2 订单簿限制范围（API 文档）——并根据需要进行设置
* 检查余额是否正确显示 (*)
* 创建市价单 (*)
* 创建限价单 (*)
* 取消订单 (*)
* 完成交易（入场 + 出场）(*)
  * 比较交易所和机器人之间的结果计算
  * 确保费用正确应用（将数据库与交易所进行对比检查）

(*) 需要交易所的 API 密钥和余额。

### 交易所止损单

检查新交易所是否通过其 API 支持交易所止损单。

由于 CCXT 尚未为交易所止损单提供统一接口，我们需要自己实现交易所特定的参数。最好参考 `binance.py` 作为示例实现。您需要查阅交易所 API 的文档，了解如何具体实现。[CCXT Issues](https://github.com/ccxt/ccxt/issues) 也可能提供很大帮助，因为其他人可能已经为他们的项目实现了类似的功能。

### 不完整的 K 线

在获取 K 线（OHLCV）数据时，我们可能会得到不完整的 K 线（取决于交易所）。
为了说明这一点，我们将使用日线 K 线（`"1d"`）来保持简单。
我们通过 API (`ct.fetch_ohlcv()`) 查询时间框架并查看最后一条记录的日期。如果此记录发生变化或显示的是"不完整" K 线的日期，那么我们应该丢弃它，因为不完整的 K 线是有问题的——指标假设只接收完整的 K 线，不完整的 K 线会产生大量虚假的买入信号。因此，默认情况下，我们删除最后一根 K 线，假设它是不完整的。

要检查新交易所的行为，您可以使用以下代码片段：

``` python
import ccxt
from datetime import datetime, timezone
from freqtrade.data.converter import ohlcv_to_dataframe
ct = ccxt.binance()  # Use the exchange you're testing
timeframe = "1d"
pair = "BTC/USDT"  # Make sure to use a pair that exists on that exchange!
raw = ct.fetch_ohlcv(pair, timeframe=timeframe)

# convert to dataframe
df1 = ohlcv_to_dataframe(raw, timeframe, pair=pair, drop_incomplete=False)

print(df1.tail(1))
print(datetime.now(timezone.utc))
```

``` output
                         date      open      high       low     close  volume
499 2019-06-08 00:00:00+00:00  0.000007  0.000007  0.000007  0.000007   26264344.0
2019-06-09 12:30:27.873327
```

输出将显示来自交易所的最后一条记录以及当前 UTC 日期。
如果日期显示的是同一天，则可以认为最后一根 K 线是不完整的，应该被丢弃（保持交易所类中 `"ohlcv_partial_candle"` 设置不变 / 为 True）。否则，将 `"ohlcv_partial_candle"` 设置为 `False` 以不丢弃 K 线（如上例所示）。
另一种方法是连续多次运行此命令，观察成交量是否在变化（而日期保持不变）。

### 更新 Binance 缓存的杠杆层级

更新杠杆层级应定期进行——需要一个已启用合约交易的认证账户。

``` python
import ccxt
import json
from pathlib import Path

exchange = ccxt.binance({
    'apiKey': '<apikey>',
    'secret': '<secret>',
    'options': {'defaultType': 'swap'}
    })
_ = exchange.load_markets()

lev_tiers = exchange.fetch_leverage_tiers()

# Assumes this is running in the root of the repository.
file = Path('freqtrade/exchange/binance_leverage_tiers.json')
json.dump(dict(sorted(lev_tiers.items())), file.open('w'), indent=2)

```

此文件应该贡献到上游，以便其他人也能从中受益。

## 更新示例 Notebook

为了保持 Jupyter Notebook 与文档的一致性，在更新示例 Notebook 后应运行以下命令：

``` bash
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --inplace freqtrade/templates/strategy_analysis_example.ipynb
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --to markdown freqtrade/templates/strategy_analysis_example.ipynb --stdout > docs/strategy_analysis_example.md
```

## 回测文档结果

要生成回测输出，请使用以下命令：

``` bash
# Assume a dedicated user directory for this output
freqtrade create-userdir --userdir user_data_bttest/
# set can_short = True
sed -i "s/can_short: bool = False/can_short: bool = True/" user_data_bttest/strategies/sample_strategy.py

freqtrade download-data --timerange 20250625-20250801 --config tests/testdata/config.tests.usdt.json --userdir user_data_bttest/ -t 5m

freqtrade backtesting --config tests/testdata/config.tests.usdt.json -s SampleStrategy --userdir user_data_bttest/ --cache none --timerange 20250701-20250801
```

## 持续集成

本节记录了 CI 流水线的一些决策。

* CI 在所有操作系统变体上运行：Linux (ubuntu)、macOS 和 Windows。
* Docker 镜像为 `stable` 和 `develop` 分支构建，构建为多架构镜像，通过同一标签支持多个平台。
* 包含绘图依赖的 Docker 镜像也以 `stable_plot` 和 `develop_plot` 的形式提供。
* Docker 镜像包含一个文件 `/freqtrade/freqtrade_commit`，其中包含该镜像所基于的提交信息。
* 完整的 Docker 镜像重建通过计划任务每周运行一次。
* 部署在 ubuntu 上运行。
* 所有测试必须通过，PR 才能合并到 `stable` 或 `develop`。

## 创建发布版本

文档的这一部分面向维护者，展示如何创建发布版本。

### 创建发布分支

!!! Note
    确保 `stable` 分支是最新的！

首先，选择一个大约一周前的提交（以避免将最新的更改包含在发布版本中）。

``` bash
# create new branch
git checkout -b new_release <commitid>
```

确定在此提交和当前状态之间是否有关键的错误修复，并在需要时 cherry-pick 这些修复。

* 将发布分支（stable）合并到此分支。
* 编辑 `freqtrade/__init__.py` 并添加与当前日期匹配的版本号（例如，2025 年 7 月为 `2025.7`）。如果当月需要进行第二次发布，次要版本可以是 `2025.7.1`。版本号必须遵循 PEP0440 允许的版本格式，以避免推送到 pypi 时失败。
* 提交此部分。
* 将该分支推送到远程并创建一个针对 **stable 分支**的 PR。
* 将 develop 版本更新为下一个版本，遵循 `2025.8-dev` 的模式。

### 从 git 提交生成更新日志

``` bash
# Needs to be done before merging / pulling that branch.
git log --oneline --no-decorate --no-merges stable..new_release
```

为了保持发布日志简短，最好将完整的 git 更新日志包装在一个可折叠的详情部分中。

```markdown
<details>
<summary>Expand full changelog</summary>

... Full git changelog

</details>
```

### FreqUI 发布

如果 FreqUI 有较大更新，请确保在合并发布分支之前创建一个发布版本。
确保发布版本的 FreqUI CI 在合并发布版本之前已完成并通过。

### 创建 GitHub 发布版本 / 标签

一旦针对 stable 的 PR 被合并（最好在合并后立即进行）：

* 使用 Github UI 中的 "Draft a new release" 按钮（releases 子部分）。
* 使用指定的版本号作为标签。
* 使用 "stable" 作为参考（此步骤在上述 PR 合并之后）。
* 使用上述更新日志作为发布说明（以代码块形式）。
* 使用以下代码片段作为新发布模板

??? Tip "发布模板"
    ````
    --8<-- "includes/release_template.md"
    ````

## 发布

### pypi

!!! Warning "手动发布"
    此过程已作为 Github Actions 的一部分自动化。
    通常不需要手动推送到 pypi。

??? example "手动发布"
    要手动创建 pypi 发布，请运行以下命令：

    额外要求：`wheel`、`twine`（用于上传）、具有适当权限的 pypi 账户。

    ``` bash
    pip install -U build
    python -m build --sdist --wheel

    # For pypi test (to check if some change to the installation did work)
    twine upload --repository-url https://test.pypi.org/legacy/ dist/*

    # For production:
    twine upload dist/*
    ```

    请不要将非发布版本推送到生产/正式的 pypi 实例。
