# Webhook 使用

## 配置

通过在配置文件中添加 webhook 部分，并将 `webhook.enabled` 设置为 `true` 来启用 webhook。

示例配置（使用 IFTTT 测试通过）。

```json
  "webhook": {
        "enabled": true,
        "url": "https://maker.ifttt.com/trigger/<YOUREVENT>/with/key/<YOURKEY>/",
        "entry": {
            "value1": "Buying {pair}",
            "value2": "limit {limit:8f}",
            "value3": "{stake_amount:8f} {stake_currency}"
        },
        "entry_cancel": {
            "value1": "Cancelling Open Buy Order for {pair}",
            "value2": "limit {limit:8f}",
            "value3": "{stake_amount:8f} {stake_currency}"
        },
         "entry_fill": {
            "value1": "Buy Order for {pair} filled",
            "value2": "at {open_rate:8f}",
            "value3": ""
        },
        "exit": {
            "value1": "Exiting {pair}",
            "value2": "limit {limit:8f}",
            "value3": "profit: {profit_amount:8f} {stake_currency} ({profit_ratio})"
        },
        "exit_cancel": {
            "value1": "Cancelling Open Exit Order for {pair}",
            "value2": "limit {limit:8f}",
            "value3": "profit: {profit_amount:8f} {stake_currency} ({profit_ratio})"
        },
        "exit_fill": {
            "value1": "Exit Order for {pair} filled",
            "value2": "at {close_rate:8f}.",
            "value3": ""
        },
        "status": {
            "value1": "Status: {status}",
            "value2": "",
            "value3": ""
        }
    },
```

`webhook.url` 中的 URL 应指向你的 webhook 的正确 URL。如果你使用的是 [IFTTT](https://ifttt.com)（如上面的示例所示），请在 URL 中插入你的事件和密钥。

你可以将 POST 请求体格式设置为 Form-Encoded（默认）、JSON-Encoded 或原始数据。分别使用 `"format": "form"`、`"format": "json"` 或 `"format": "raw"`。Mattermost Cloud 集成的示例配置：

```json
  "webhook": {
        "enabled": true,
        "url": "https://<YOURSUBDOMAIN>.cloud.mattermost.com/hooks/<YOURHOOK>",
        "format": "json",
        "status": {
            "text": "Status: {status}"
        }
    },
```

结果将是一个 POST 请求，例如请求体为 `{"text":"Status: running"}`，请求头为 `Content-Type: application/json`，在 Mattermost 频道中显示 `Status: running` 消息。

使用 Form-Encoded 或 JSON-Encoded 配置时，你可以配置任意数量的负载值，键和值都将输出到 POST 请求中。然而，使用原始数据格式时，你只能配置一个值，且**必须**命名为 `"data"`。在这种情况下，data 键将不会输出到 POST 请求中，只有值会输出。例如：

```json
  "webhook": {
        "enabled": true,
        "url": "https://<YOURHOOKURL>",
        "format": "raw",
        "webhookstatus": {
            "data": "Status: {status}"
        }
    },
```

结果将是一个 POST 请求，例如请求体为 `Status: running`，请求头为 `Content-Type: text/plain`。

### 嵌套 Webhook 配置

某些 webhook 目标需要嵌套结构。
这可以通过将内容设置为字典或列表而非直接文本来实现。

这仅支持 JSON 格式。

```json
"webhook": {
    "enabled": true,
    "url": "https://<yourhookurl>",
    "format": "json",
    "status": {
        "msgtype": "text",
        "text": {
            "content": "Status update: {status}"
        }
    }
}
```

结果将是一个 POST 请求，例如请求体为 `{"msgtype":"text","text":{"content":"Status update: running"}}`，请求头为 `Content-Type: application/json`。

## 附加配置

`webhook.retries` 参数可以设置 webhook 请求在不成功时（即 HTTP 响应状态不是 200）应尝试的最大重试次数。默认设置为 `0`，即禁用。可以设置额外的 `webhook.retry_delay` 参数来指定重试之间的等待时间（以秒为单位）。默认设置为 `0.1`（即 100ms）。请注意，增加重试次数或重试延迟可能会在 webhook 存在连接问题时减慢交易速度。
你还可以指定 `webhook.timeout`——定义机器人在认为对方主机无响应之前等待多长时间（默认为 10 秒）。

重试的示例配置：

```json
  "webhook": {
        "enabled": true,
        "url": "https://<YOURHOOKURL>",
        "timeout": 10,
        "retries": 3,
        "retry_delay": 0.2,
        "status": {
            "status": "Status: {status}"
        }
    },
```

可以通过策略中的 `self.dp.send_msg()` 函数向 Webhook 端点发送自定义消息。要启用此功能，请将 `allow_custom_messages` 选项设置为 `true`：

```json
  "webhook": {
        "enabled": true,
        "url": "https://<YOURHOOKURL>",
        "allow_custom_messages": true,
        "strategy_msg": {
            "status": "StrategyMessage: {msg}"
        }
    },
```

可以为不同的事件配置不同的负载。不是所有字段都是必需的，但你应该至少配置其中一个字典，否则 webhook 将永远不会被调用。

## Webhook 消息类型

### 入场 / 入场成交

`webhook.entry` 和 `webhook.entry_fill` 中的字段在机器人下达做多/做空订单以增加仓位时或该订单成交时分别填充。参数使用 string.format 填充。
可用参数有：

* `trade_id`
* `exchange`
* `pair`
* `direction`
* `leverage`
* ~~`limit` # 已弃用 - 不应再使用。~~
* `open_rate`
* `amount`
* `open_date`
* `stake_amount`
* `stake_currency`
* `base_currency`
* `quote_currency`
* `fiat_currency`
* `order_type`
* `current_rate`
* `enter_tag`

### 入场取消

`webhook.entry_cancel` 中的字段在机器人取消做多/做空订单时填充。参数使用 string.format 填充。
可用参数有：

* `trade_id`
* `exchange`
* `pair`
* `direction`
* `leverage`
* `limit`
* `amount`
* `open_date`
* `stake_amount`
* `stake_currency`
* `base_currency`
* `quote_currency`
* `fiat_currency`
* `order_type`
* `current_rate`
* `enter_tag`

### 出场 / 出场成交

`webhook.exit` 和 `webhook.exit_fill` 中的字段在机器人下达出场订单或出场订单成交时分别填充。参数使用 string.format 填充。
可用参数有：

* `trade_id`
* `exchange`
* `pair`
* `direction`
* `leverage`
* `gain`
* `amount`
* `open_rate`
* `close_rate`
* `current_rate`
* `profit_amount`
* `profit_ratio`
* `stake_currency`
* `base_currency`
* `quote_currency`
* `fiat_currency`
* `enter_tag`
* `exit_reason`
* `order_type`
* `open_date`
* `close_date`
* `sub_trade`
* `is_final_exit`


### 出场取消

`webhook.exit_cancel` 中的字段在机器人取消出场订单时填充。参数使用 string.format 填充。
可用参数有：

* `trade_id`
* `exchange`
* `pair`
* `direction`
* `leverage`
* `gain`
* `order_rate`
* `amount`
* `open_rate`
* `current_rate`
* `profit_amount`
* `profit_ratio`
* `stake_currency`
* `base_currency`
* `quote_currency`
* `fiat_currency`
* `exit_reason`
* `order_type`
* `open_date`
* `close_date`

### 状态

`webhook.status` 中的字段用于常规状态消息（已启动/已停止/...）。参数使用 string.format 填充。

此处唯一可用的值是 `{status}`。

## Discord

Discord 提供了一种特殊形式的 webhook。
你可以按如下方式配置：

```json
"discord": {
    "enabled": true,
    "webhook_url": "https://discord.com/api/webhooks/<Your webhook URL ...>",
    "exit_fill": [
        {"Trade ID": "{trade_id}"},
        {"Exchange": "{exchange}"},
        {"Pair": "{pair}"},
        {"Direction": "{direction}"},
        {"Open rate": "{open_rate}"},
        {"Close rate": "{close_rate}"},
        {"Amount": "{amount}"},
        {"Open date": "{open_date:%Y-%m-%d %H:%M:%S}"},
        {"Close date": "{close_date:%Y-%m-%d %H:%M:%S}"},
        {"Profit": "{profit_amount} {stake_currency}"},
        {"Profitability": "{profit_ratio:.2%}"},
        {"Enter tag": "{enter_tag}"},
        {"Exit Reason": "{exit_reason}"},
        {"Strategy": "{strategy}"},
        {"Timeframe": "{timeframe}"},
    ],
    "entry_fill": [
        {"Trade ID": "{trade_id}"},
        {"Exchange": "{exchange}"},
        {"Pair": "{pair}"},
        {"Direction": "{direction}"},
        {"Open rate": "{open_rate}"},
        {"Amount": "{amount}"},
        {"Open date": "{open_date:%Y-%m-%d %H:%M:%S}"},
        {"Enter tag": "{enter_tag}"},
        {"Strategy": "{strategy} {timeframe}"},
    ]
}
```

以上是默认配置（`exit_fill` 和 `entry_fill` 是可选的，将默认使用上述配置）——显然可以进行修改。
要禁用两个默认值中的任何一个（`entry_fill` / `exit_fill`），你可以将它们分配一个空数组（`exit_fill: []`）。

可用字段对应于 webhook 的字段，并在相应的 webhook 部分中有文档说明。

默认情况下，通知将如下所示。

![discord-notification](assets/discord_notification.png)

可以通过 dataprovider.send_msg() 函数从策略向 Discord 端点发送自定义消息。要启用此功能，请将 `allow_custom_messages` 选项设置为 `true`：

```json
  "discord": {
        "enabled": true,
        "webhook_url": "https://discord.com/api/webhooks/<Your webhook URL ...>",
        "allow_custom_messages": true,
    },
```
