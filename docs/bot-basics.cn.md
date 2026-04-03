# Freqtrade 基础知识

本页面为您介绍 Freqtrade 的基本概念及其工作原理。

## Freqtrade 术语

* **Strategy（策略）**: 您的交易策略，告诉机器人该做什么。
* **Trade（交易）**: 已开仓的持仓。
* **Open Order（挂单）**: 当前已提交到交易所但尚未成交的订单。
* **Pair（交易对）**: 可交易的交易对，通常格式为 Base/Quote（例如现货为 `XRP/USDT`，合约为 `XRP/USDT:USDT`）。
* **Timeframe（时间周期）**: 使用的 K 线周期（例如 `"5m"`、`"1h"` 等）。
* **Indicators（指标）**: 技术指标（SMA、EMA、RSI 等）。
* **Limit order（限价单）**: 以指定限价或更优价格成交的限价订单。
* **Market order（市价单）**: 保证成交，可能会根据订单大小影响价格。
* **Current Profit（当前利润）**: 该笔交易当前的待实现（浮动）利润。主要在机器人和 UI 中使用。
* **Realized Profit（已实现利润）**: 已经实现的利润。仅在结合[部分平仓](strategy-callbacks.md#adjust-trade-position)时才有意义——其中也解释了相关的计算逻辑。
* **Total Profit（总利润）**: 已实现利润和未实现利润的合计。相对数值（%）是基于该笔交易的总投入计算的。

## 手续费处理

Freqtrade 的所有利润计算都包含手续费。在回测 / 超参数优化 / 模拟运行模式下，使用交易所的默认手续费（交易所的最低等级费率）。在实盘运行中，使用交易所实际收取的手续费（包括 BNB 返佣等）。

## 交易对命名

Freqtrade 遵循 [ccxt 命名规范](https://docs.ccxt.com/#/README?id=consistency-of-base-and-quote-currencies)来命名货币。
在错误的市场中使用错误的命名规范通常会导致机器人无法识别交易对，通常会出现"this pair is not available"之类的错误。

### 现货交易对命名

现货交易对的命名格式为 `base/quote`（例如 `ETH/USDT`）。

### 合约交易对命名

合约交易对的命名格式为 `base/quote:settle`（例如 `ETH/USDT:USDT`）。

## 机器人执行逻辑

以模拟运行或实盘模式启动 freqtrade（使用 `freqtrade trade`）将启动机器人并开始机器人的迭代循环。
同时也会运行 `bot_start()` 回调函数。

默认情况下，机器人循环每隔几秒运行一次（`internals.process_throttle_secs`），执行以下操作：

* 从持久化存储中获取未平仓交易。
* 计算当前可交易的交易对列表。
* 下载交易对列表（包括所有[信息交易对](strategy-customization.md#get-data-for-non-tradeable-pairs)）的 OHLCV 数据。
  此步骤每根 K 线只执行一次，以避免不必要的网络流量。
* 调用 `bot_loop_start()` 策略回调函数。
* 对每个交易对进行策略分析。
  * 调用 `populate_indicators()`
  * 调用 `populate_entry_trend()`
  * 调用 `populate_exit_trend()`
* 从交易所更新未平仓交易的订单状态。
  * 对已成交的订单调用 `order_filled()` 策略回调函数。
  * 检查挂单是否超时。
    * 对未成交的入场订单调用 `check_entry_timeout()` 策略回调函数。
    * 对未成交的出场订单调用 `check_exit_timeout()` 策略回调函数。
    * 对挂单调用 `adjust_order_price()` 策略回调函数。
      * 对未成交的入场订单调用 `adjust_entry_price()` 策略回调函数。*仅在未实现 `adjust_order_price()` 时调用*
      * 对未成交的出场订单调用 `adjust_exit_price()` 策略回调函数。*仅在未实现 `adjust_order_price()` 时调用*
* 验证现有持仓并在必要时下出场订单。
  * 考虑止损、ROI 和出场信号、`custom_exit()` 和 `custom_stoploss()`。
  * 根据 `exit_pricing` 配置设置或使用 `custom_exit_price()` 回调函数确定出场价格。
  * 在下出场订单之前，会调用 `confirm_trade_exit()` 策略回调函数。
* 如果启用了仓位调整，检查未平仓交易的仓位调整，调用 `adjust_trade_position()` 并在需要时下额外的订单。
* 检查交易槽位是否仍然可用（是否已达到 `max_open_trades` 限制）。
* 验证入场信号，尝试开新仓。
  * 根据 `entry_pricing` 配置设置或使用 `custom_entry_price()` 回调函数确定入场价格。
  * 在保证金和合约模式下，会调用 `leverage()` 策略回调函数来确定所需的杠杆倍数。
  * 通过调用 `custom_stake_amount()` 回调函数确定下单金额。
  * 在下入场订单之前，会调用 `confirm_trade_entry()` 策略回调函数。

此循环将不断重复，直到机器人停止。

## 回测 / 超参数优化执行逻辑

[回测](backtesting.md)或[超参数优化](hyperopt.md)仅执行上述逻辑的一部分，因为大多数交易操作都是完全模拟的。

* 为配置的交易对列表加载历史数据。
* 调用一次 `bot_start()`。
* 计算指标（对每个交易对调用一次 `populate_indicators()`）。
* 计算入场/出场信号（对每个交易对调用一次 `populate_entry_trend()` 和 `populate_exit_trend()`）。
* 逐根 K 线循环模拟入场和出场点。
  * 调用 `bot_loop_start()` 策略回调函数。
  * 通过 `unfilledtimeout` 配置或通过 `check_entry_timeout()` / `check_exit_timeout()` 策略回调函数检查订单超时。
  * 对挂单调用 `adjust_order_price()` 策略回调函数。
    * 对未成交的入场订单调用 `adjust_entry_price()` 策略回调函数。*仅在未实现 `adjust_order_price()` 时调用！*
    * 对未成交的出场订单调用 `adjust_exit_price()` 策略回调函数。*仅在未实现 `adjust_order_price()` 时调用！*
  * 检查交易入场信号（`enter_long` / `enter_short` 列）。
  * 确认交易入场/出场（如果在策略中实现了 `confirm_trade_entry()` 和 `confirm_trade_exit()` 则调用之）。
  * 调用 `custom_entry_price()`（如果在策略中实现了）以确定入场价格（价格会被调整到开盘 K 线范围内）。
  * 在保证金和合约模式下，调用 `leverage()` 策略回调函数以确定所需的杠杆倍数。
  * 通过调用 `custom_stake_amount()` 回调函数确定下单金额。
  * 如果启用了仓位调整，检查未平仓交易的仓位调整，调用 `adjust_trade_position()` 以确定是否需要额外的订单。
  * 对已成交的入场订单调用 `order_filled()` 策略回调函数。
  * 调用 `custom_stoploss()` 和 `custom_exit()` 以查找自定义出场点。
  * 对于基于出场信号、自定义出场和部分出场的退出：调用 `custom_exit_price()` 以确定出场价格（价格会被调整到收盘 K 线范围内）。
  * 对已成交的出场订单调用 `order_filled()` 策略回调函数。
* 生成回测报告输出

!!! Note
    回测和超参数优化都在计算中包含了交易所默认手续费。可以通过指定 `--fee` 参数向回测/超参数优化传入自定义手续费。

!!! Warning "回调函数调用频率"
    回测对每个回调函数最多每根 K 线调用一次（`--timeframe-detail` 会修改此行为，变为每根详细 K 线调用一次）。
    大多数回调函数在实盘中每次迭代调用一次（通常大约每 ~5 秒）——这可能导致回测结果不匹配。
