``` output
可用命令：

available_pairs
	根据时间周期/权益货币选择返回可用的交易对（回测数据）

:param timeframe: 仅具有此时间周期可用的交易对。
:param stake_currency: 仅包含此权益货币的交易对。

balance
	获取账户余额。

blacklist
	显示当前黑名单。

:param add: 要添加的币种列表（例如："BNB/BTC"）

cancel_open_order
	取消交易的未结订单。

:param trade_id: 取消此交易的未结订单。

count
	返回未结交易的数量。

daily
	返回每天的利润和交易数量。

delete_lock
	从数据库中删除（禁用）锁定。

:param lock_id: 要删除的锁定的 ID

delete_trade
	从数据库中删除交易。
尝试关闭未结订单。需要在交易所手动处理此资产。

:param trade_id: 从数据库中删除具有此 ID 的交易。

entries
	返回包含所有交易的字典列表，基于买入标签性能
可以是所有交易对的平均值或提供的特定交易对

exits
	返回包含所有交易的字典列表，基于退出原因性能
可以是所有交易对的平均值或提供的特定交易对

forcebuy
	买入资产。

:param pair: 要买入的交易对（ETH/BTC）
:param price: 可选 - 买入价格

forceenter
	强制进入交易

:param pair: 要买入的交易对（ETH/BTC）
:param side: 'long' 或 'short'
:param price: 可选 - 买入价格
:param order_type: 可选关键字参数 - 'limit' 或 'market'
:param stake_amount: 可选关键字参数 - 权益金额（作为浮点数）
:param leverage: 可选关键字参数 - 杠杆（作为浮点数）
:param enter_tag: 可选关键字参数 - 进入标签（作为字符串，默认值：'force_enter'）

forceexit
	强制退出交易。

:param tradeid: 交易的 ID（可以通过 status 命令获取）
:param ordertype: 要使用的订单类型（必须是 market 或 limit）
:param amount: 要卖出的数量。如果未给出，则全部卖出

health
	提供运行中机器人的快速健康检查。

list_custom_data
	列出运行中机器人的特定交易的自定义数据。

:param trade_id: 交易的 ID
:param key: str，可选 - 自定义数据的键

list_open_trades_custom_data
	列出运行中机器人的未结交易自定义数据。

:param key: str，可选 - 自定义数据的键
:param limit: 交易限制
:param offset: 分页的交易偏移量

lock_add
	锁定交易对

:param pair: 要锁定的交易对
:param until: 锁定到此日期（格式 "2024-03-30 16:00:00Z"）
:param side: 要锁定的方向（long、short、*）
:param reason: 锁定原因

locks
	返回当前锁定

logs
	显示最新日志。

:param limit: 将日志消息限制为最后 <limit> 条日志。无限制以获取整个日志。

mix_tags
	返回包含所有交易的字典列表，基于 entry_tag + exit_reason 性能
可以是所有交易对的平均值或提供的特定交易对

monthly
	返回每月的利润和交易数量。

pair_candles
	返回 <pair><timeframe> 的实时数据帧。

:param pair: 要获取数据的交易对
:param timeframe: 仅具有此时间周期可用的交易对。
:param limit: 将结果限制为最后 n 根蜡烛图。
:param columns: 要返回的数据帧列列表。空列表将返回 OHLCV。

pair_history
	返回历史的、已分析的数据帧

:param pair: 要获取数据的交易对
:param timeframe: 仅具有此时间周期可用的交易对。
:param strategy: 要分析和获取值的策略
:param freqaimodel: 用于分析的 FreqAI 模型
:param timerange: 要获取数据的时间范围（与 --timerange 端点相同的格式）

pairlists_available
	列出可用的交易对列表提供程序

performance
	返回不同币种的性能。

ping
	简单 ping

plot_config
	如果策略定义了绘图配置，则返回绘图配置。

profit
	返回利润摘要。

reload_config
	重新加载配置。

show_config
	返回配置的一部分，与交易操作相关。

start
	如果机器人处于停止状态，则启动机器人。

stats
	返回统计报告（持续时间、卖出原因）。

status
	获取未结交易的状态。

stop
	停止机器人。使用 `start` 重新启动。

stopbuy
	停止买入（但优雅地处理卖出）。使用 `reload_config` 重置。

strategies
	列出可用策略

strategy
	获取策略详细信息

:param strategy: 策略类名称

sysinfo
	提供系统信息（CPU、RAM 使用情况）

trade
	返回特定交易

:param trade_id: 指定要获取的交易。

trades
	返回交易历史，按 id 排序（如果 order_by_id=False，则按最新时间戳排序）

:param limit: 将交易限制为最后 X 笔交易。最多 500 笔交易。
:param offset: 按此交易数量偏移。
:param order_by_id: 按 id 排序交易（默认值：True）。如果为 False，则按最新时间戳排序。

version
	返回机器人的版本。

weekly
	返回每周的利润和交易数量。

whitelist
	显示当前白名单。


```
