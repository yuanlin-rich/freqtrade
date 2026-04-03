# REST API

## FreqUI

FreqUI 现在有自己专门的[文档章节](freq-ui.md) - 请参阅该章节了解有关 FreqUI 的所有信息。

## 配置

通过在配置中添加 api_server 部分并将 `api_server.enabled` 设置为 `true` 来启用 REST API。

示例配置：

``` json
    "api_server": {
        "enabled": true,
        "listen_ip_address": "127.0.0.1",
        "listen_port": 8080,
        "verbosity": "error",
        "enable_openapi": false,
        "jwt_secret_key": "somethingRandomSomethingRandom123",
        "CORS_origins": [],
        "username": "Freqtrader",
        "password": "SuperSecret1!",
        "ws_token": "sercet_Ws_t0ken"
    },
```

!!! Danger "安全警告"
    默认情况下，配置仅监听 localhost（因此无法从其他系统访问）。我们强烈建议不要将此 API 暴露到互联网，并选择一个强大、唯一的密码，因为其他人可能会控制你的机器人。

??? Note "在远程服务器上访问 API/UI"
    如果你在 VPS 上运行，应该考虑使用 SSH 隧道或设置 VPN（OpenVPN、WireGuard）来连接你的机器人。
    这将确保 FreqUI 不会直接暴露在互联网上，出于安全原因不建议这样做（FreqUI 不支持开箱即用的 HTTPS）。
    这些工具的设置不属于本教程的范围，但互联网上可以找到许多好的教程。

然后你可以通过在浏览器中访问 `http://127.0.0.1:8080/api/v1/ping` 来检查 API 是否正常运行。
应该返回以下响应：

``` output
{"status":"pong"}
```

所有其他端点返回敏感信息，需要身份验证，因此无法通过网络浏览器访问。

### 安全性

要生成安全密码，最好使用密码管理器，或使用以下代码。

``` python
import secrets
secrets.token_hex()
```

!!! Hint "JWT 令牌"
    使用同样的方法来生成 JWT 密钥（`jwt_secret_key`）。

!!! Danger "密码选择"
    请确保选择一个非常强大、唯一的密码来保护你的机器人免受未授权访问。
    同时将 `jwt_secret_key` 更改为随机值（无需记住，但它将用于加密你的会话，所以最好是唯一的！）。此值也应为 32 个字符或更长以确保安全。

### Docker 配置

如果你使用 Docker 运行机器人，需要让机器人监听传入连接。安全性由 Docker 处理。

``` json
    "api_server": {
        "enabled": true,
        "listen_ip_address": "0.0.0.0",
        "listen_port": 8080,
        "username": "Freqtrader",
        "password": "SuperSecret1!",
        //...
    },
```

确保 docker-compose 文件中有以下两行：

```yml
    ports:
      - "127.0.0.1:8080:8080"
```

!!! Danger "安全警告"
    在 Docker 端口映射中使用 `"8080:8080"`（或 `"0.0.0.0:8080:8080"`），API 将对所有连接到服务器正确端口的人可用，因此其他人可能能够控制你的机器人。
    如果你在安全环境（如家庭网络）中运行机器人，这**可能**是安全的，但不建议将 API 暴露到互联网。

## Rest API

### 使用 API

我们建议使用受支持的 `freqtrade-client` 包（也可作为 `scripts/rest_client.py` 使用）来使用 API。

此命令可以独立于任何运行中的 freqtrade 机器人通过 `pip install freqtrade-client` 安装。

此模块设计为轻量级，仅依赖 `requests` 和 `python-rapidjson` 模块，跳过了 freqtrade 通常需要的所有重量级依赖。

``` bash
freqtrade-client <command> [optional parameters]
```

默认情况下，脚本假设使用 `127.0.0.1`（localhost）和端口 `8080`，但你可以指定配置文件来覆盖此行为。

#### 最简客户端配置

``` json
{
    "api_server": {
        "enabled": true,
        "listen_ip_address": "0.0.0.0",
        "listen_port": 8080,
        "username": "Freqtrader",
        "password": "SuperSecret1!",
        //...
    }
}
```

``` bash
freqtrade-client --config rest_config.json <command> [optional parameters]
```

带有多个参数的命令可能需要关键字参数（为了清晰） - 可以按以下方式提供：

``` bash
freqtrade-client --config rest_config.json forceenter BTC/USDT long enter_tag=GutFeeling
```

此方法适用于所有参数 - 查看 "show" 命令以获取可用参数列表。

??? Note "编程使用"
    `freqtrade-client` 包（可独立于 freqtrade 安装）可以在你自己的脚本中使用，与 freqtrade API 交互。
    为此，请使用以下方式：

    ``` python
    from freqtrade_client import FtRestClient


    client = FtRestClient(server_url, username, password)

    # Get the status of the bot
    ping = client.ping()
    print(ping)

    # Add pairs to blacklist
    client.blacklist("BTC/USDT", "ETH/USDT")
    # Add pairs to blacklist by supplying a list
    client.blacklist(*listPairs)
    # ...
    ```

    有关可用命令的完整列表，请参阅下面的列表。

#### Freqtrade 客户端 - 可用命令

可以使用 `help` 命令从 REST 客户端脚本中列出可用命令。

``` bash
freqtrade-client help
```

--8<-- "commands/freqtrade-client.md"


### 可用端点

如果你希望通过其他途径手动调用 REST API，例如直接通过 `curl`，下表显示了相关的 URL 端点和参数。
下表中的所有端点都需要以 API 的基础 URL 为前缀，例如 `http://127.0.0.1:8080/api/v1/` - 因此命令变为 `http://127.0.0.1:8080/api/v1/<command>`。

|  端点 | 方法 | 描述 / 参数 |
|-----------|--------|--------------------------|
| `/ping` | GET | 简单的 API 就绪状态测试命令 - 无需身份验证。
| `/start` | POST | 启动交易器。
| `/pause` | POST | 暂停交易器。根据规则优雅地处理未平仓交易。不进入新仓位。
| `/stop` | POST | 停止交易器。
| `/stopbuy` | POST | 阻止交易器开启新交易。根据规则优雅地关闭未平仓交易。
| `/reload_config` | POST | 重新加载配置文件。
| `/trades` | GET | 列出最近的交易。每次调用限制 500 笔交易。
| `/trade/<tradeid>` | GET | 获取特定交易。<br/>*参数：*<br/>- `tradeid` (`int`)
| `/trades/<tradeid>` | DELETE | 从数据库中删除交易。尝试关闭未完成的订单。需要在交易所手动处理此交易。<br/>*参数：*<br/>- `tradeid` (`int`)
| `/trades/<tradeid>/open-order` | DELETE | 取消此交易的未完成订单。<br/>*参数：*<br/>- `tradeid` (`int`)
| `/trades/<tradeid>/reload` | POST | 从交易所重新加载交易。仅在实盘模式下有效，可以帮助恢复在交易所上手动卖出的交易。<br/>*参数：*<br/>- `tradeid` (`int`)
| `/show_config` | GET | 显示当前配置中与操作相关的部分设置。
| `/logs` | GET | 显示最近的日志消息。
| `/status` | GET | 列出所有未平仓交易。
| `/count` | GET | 显示已使用和可用的交易数量。
| `/entries` | GET | 显示给定交易对（或所有交易对，如果未指定交易对）的每个入场标签的利润统计。交易对为可选。<br/>*参数：*<br/>- `pair` (`str`)
| `/exits` | GET | 显示给定交易对（或所有交易对，如果未指定交易对）的每个退出原因的利润统计。交易对为可选。<br/>*参数：*<br/>- `pair` (`str`)
| `/mix_tags` | GET | 显示给定交易对（或所有交易对，如果未指定交易对）的每个入场标签+退出原因组合的利润统计。交易对为可选。<br/>*参数：*<br/>- `pair` (`str`)
| `/locks` | GET | 显示当前锁定的交易对。
| `/locks` | POST | 锁定一个交易对直到 "until"。（Until 将向上取整到最近的时间框架）。Side 为可选，值为 `long` 或 `short`（默认为 `long`）。Reason 为可选。<br/>*参数：*<br/>- `<pair>` (`str`)<br/>- `<until>` (`datetime`)<br/>- `[side]` (`str`)<br/>- `[reason]` (`str`)
| `/locks/<lockid>` | DELETE | 按 ID 删除（禁用）锁定。<br/>*参数：*<br/>- `lockid` (`int`)
| `/profit` | GET | 显示已关闭交易的利润/亏损摘要以及一些性能统计数据。
| `/forceexit` | POST | 立即退出给定交易（忽略 `minimum_roi`），使用给定的订单类型（"market" 或 "limit"，如果未指定则使用配置设置），以及选定的数量（如果未指定则全部卖出）。如果提供 `all` 作为 `tradeid`，则所有当前未平仓交易将被强制退出。<br/>*参数：*<br/>- `<tradeid>` (`int` 或 `str`)<br/>- `<ordertype>` (`str`)<br/>- `[amount]` (`float`)
| `/forceenter` | POST | 立即进入给定交易对。Side 为可选，值为 `long` 或 `short`（默认为 `long`）。价格、质押数量、入场标签和杠杆为可选。订单类型为可选，值为 `market` 或 `long`（默认使用配置中设置的值）。（`force_entry_enable` 必须设置为 True）<br/>*参数：*<br/>- `<pair>` (`str`)<br/>- `<side>` (`str`)<br/>- `[price]` (`float`)<br/>- `[ordertype]` (`str`)<br/>- `[stakeamount]` (`float`)<br/>- `[entry_tag]` (`str`)<br/>- `[leverage]` (`float`)
| `/performance` | GET | 显示每个已完成交易按交易对分组的表现。
| `/balance` | GET | 按货币显示账户余额。
| `/daily` | GET | 显示过去 n 天的每日利润或亏损（n 默认为 7）。<br/>*参数：*<br/>- `timescale` (`int`)
| `/weekly` | GET | 显示过去 n 天的每周利润或亏损（n 默认为 4）。<br/>*参数：*<br/>- `timescale` (`int`)
| `/monthly` | GET | 显示过去 n 天的每月利润或亏损（n 默认为 3）。<br/>*参数：*<br/>- `timescale` (`int`)
| `/stats` | GET | 显示利润/亏损原因摘要以及平均持仓时间。
| `/whitelist` | GET | 显示当前白名单。
| `/blacklist` | GET | 显示当前黑名单。
| `/blacklist` | POST | 将指定交易对添加到黑名单。<br/>*参数：*<br/>- `blacklist` (`str`)
| `/blacklist` | DELETE | 从黑名单中删除指定的交易对列表。<br/>*参数：*<br/>- `[pair,pair]` (`list[str]`)
| `/pair_candles` | GET | 在机器人运行时返回交易对/时间框架组合的 dataframe。**Alpha**
| `/pair_candles` | POST | 在机器人运行时返回交易对/时间框架组合的 dataframe，通过提供的列列表进行过滤。**Alpha**<br/>*参数：*<br/>- `<column_list>` (`list[str]`)
| `/pair_history` | GET | 返回给定时间范围的分析 dataframe，由给定策略分析。**Alpha**
| `/pair_history` | POST | 返回给定时间范围的分析 dataframe，由给定策略分析，通过提供的列列表进行过滤。**Alpha**<br/>*参数：*<br/>- `<column_list>` (`list[str]`)
| `/plot_config` | GET | 从策略获取绘图配置（如果未配置则为空）。**Alpha**
| `/strategies` | GET | 列出策略目录中的策略。**Alpha**
| `/strategy/<strategy>` | GET | 按策略类名获取特定策略内容。**Alpha**<br/>*参数：*<br/>- `<strategy>` (`str`)
| `/available_pairs` | GET | 列出可用的回测数据。**Alpha**
| `/version` | GET | 显示版本。
| `/sysinfo` | GET | 显示系统负载信息。
| `/health` | GET | 显示机器人健康状态（最后一次机器人循环）。

!!! Warning "Alpha 状态"
    上面标记为 *Alpha 状态* 的端点可能随时更改，恕不另行通知。

### 消息 WebSocket

API 服务器包含一个 WebSocket 端点，用于订阅来自 freqtrade 机器人的 RPC 消息。
这可用于消费来自机器人的实时数据，例如入场/退出成交消息、白名单变更、交易对的已填充指标等。

这也用于在 Freqtrade 中设置[生产者/消费者模式](producer-consumer.md)。

假设你的 REST API 设置为 `127.0.0.1`，端口为 `8080`，端点可在 `http://localhost:8080/api/v1/message/ws` 访问。

要访问 WebSocket 端点，需要在端点 URL 中将 `ws_token` 作为查询参数传递。

要生成安全的 `ws_token`，你可以运行以下代码：

``` python
>>> import secrets
>>> secrets.token_urlsafe(25)
'hZ-y58LXyX_HZ8O1cJzVyN6ePWrLpNQv4Q'
```

然后你需要在 `api_server` 配置中的 `ws_token` 下添加该令牌。如下所示：

``` json
"api_server": {
    "enabled": true,
    "listen_ip_address": "127.0.0.1",
    "listen_port": 8080,
    "verbosity": "error",
    "enable_openapi": false,
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": [],
    "username": "Freqtrader",
    "password": "SuperSecret1!",
    "ws_token": "hZ-y58LXyX_HZ8O1cJzVyN6ePWrLpNQv4Q" // <-----
},
```

现在你可以连接到端点 `http://localhost:8080/api/v1/message/ws?token=hZ-y58LXyX_HZ8O1cJzVyN6ePWrLpNQv4Q`。

!!! Danger "重复使用示例令牌"
    请不要使用上面的示例令牌。为确保安全，请生成一个全新的令牌。

#### 使用 WebSocket

连接到 WebSocket 后，机器人将向所有订阅者广播 RPC 消息。要订阅消息列表，你必须通过 WebSocket 发送如下 JSON 请求。`data` 键必须是消息类型字符串的列表。

``` json
{
  "type": "subscribe",
  "data": ["whitelist", "analyzed_df"] // A list of string message types
}
```

有关消息类型的列表，请参阅 `freqtrade/enums/rpcmessagetype.py` 中的 RPCMessageType 枚举。

现在只要机器人中发送了这些类型的 RPC 消息，只要连接处于活动状态，你就会通过 WebSocket 收到它们。它们通常采用与请求相同的格式：

``` json
{
  "type": "analyzed_df",
  "data": {
      "key": ["NEO/BTC", "5m", "spot"],
      "df": {}, // The dataframe
      "la": "2022-09-08 22:14:41.457786+00:00"
  }
}
```

#### 反向代理设置

使用 [Nginx](https://nginx.org/en/docs/) 时，需要以下配置才能使 WebSocket 工作（注意此配置不完整，缺少一些信息，不能直接使用）：

请确保将 `<freqtrade_listen_ip>`（以及后续的端口）替换为与你的配置/设置匹配的 IP 和端口。

```
http {
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    #...

    server {
        #...

        location / {
            proxy_http_version 1.1;
            proxy_pass http://<freqtrade_listen_ip>:8080;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_set_header Host $host;
        }
    }
}
```

要正确（安全地）配置你的反向代理，请查阅其关于代理 WebSocket 的文档。

- **Traefik**：Traefik 开箱即用地支持 WebSocket，请参阅[文档](https://doc.traefik.io/traefik/)
- **Caddy**：Caddy v2 开箱即用地支持 WebSocket，请参阅[文档](https://caddyserver.com/docs/v2-upgrade#proxy)

!!! Tip "SSL 证书"
    你可以使用 certbot 等工具设置 SSL 证书，通过使用上述任何反向代理来通过加密连接访问你的机器人 UI。
    虽然这将保护传输中的数据，但我们不建议在你的私有网络（VPN、SSH 隧道）之外运行 freqtrade API。

### OpenAPI 接口

要启用内置的 OpenAPI 接口（Swagger UI），请在 api_server 配置中指定 `"enable_openapi": true`。
这将在 `/docs` 端点启用 Swagger UI。默认情况下，它运行在 <http://localhost:8080/docs> - 但具体取决于你的设置。

### 使用 JWT 令牌的高级 API 用法

!!! Note
    以下操作应在应用程序（通过 API 获取信息的 Freqtrade REST API 客户端）中完成，不适用于日常使用。

Freqtrade 的 REST API 也提供 JWT（JSON Web Tokens）。
你可以使用以下命令登录，然后使用返回的 access_token。

``` bash
> curl -X POST --user Freqtrader http://localhost:8080/api/v1/token/login
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1ODkxMTk2ODEsIm5iZiI6MTU4OTExOTY4MSwianRpIjoiMmEwYmY0NWUtMjhmOS00YTUzLTlmNzItMmM5ZWVlYThkNzc2IiwiZXhwIjoxNTg5MTIwNTgxLCJpZGVudGl0eSI6eyJ1IjoiRnJlcXRyYWRlciJ9LCJmcmVzaCI6ZmFsc2UsInR5cGUiOiJhY2Nlc3MifQ.qt6MAXYIa-l556OM7arBvYJ0SDI9J8bIk3_glDujF5g","refresh_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1ODkxMTk2ODEsIm5iZiI6MTU4OTExOTY4MSwianRpIjoiZWQ1ZWI3YjAtYjMwMy00YzAyLTg2N2MtNWViMjIxNWQ2YTMxIiwiZXhwIjoxNTkxNzExNjgxLCJpZGVudGl0eSI6eyJ1IjoiRnJlcXRyYWRlciJ9LCJ0eXBlIjoicmVmcmVzaCJ9.d1AT_jYICyTAjD0fiQAr52rkRqtxCjUGEMwlNuuzgNQ"}

> access_token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1ODkxMTk2ODEsIm5iZiI6MTU4OTExOTY4MSwianRpIjoiMmEwYmY0NWUtMjhmOS00YTUzLTlmNzItMmM5ZWVlYThkNzc2IiwiZXhwIjoxNTg5MTIwNTgxLCJpZGVudGl0eSI6eyJ1IjoiRnJlcXRyYWRlciJ9LCJmcmVzaCI6ZmFsc2UsInR5cGUiOiJhY2Nlc3MifQ.qt6MAXYIa-l556OM7arBvYJ0SDI9J8bIk3_glDujF5g"
# Use access_token for authentication
> curl -X GET --header "Authorization: Bearer ${access_token}" http://localhost:8080/api/v1/count

```

由于 access token 有较短的超时时间（15 分钟）- 应定期使用 `token/refresh` 请求来获取新的 access token：

``` bash
> curl -X POST --header "Authorization: Bearer ${refresh_token}"http://localhost:8080/api/v1/token/refresh
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1ODkxMTk5NzQsIm5iZiI6MTU4OTExOTk3NCwianRpIjoiMDBjNTlhMWUtMjBmYS00ZTk0LTliZjAtNWQwNTg2MTdiZDIyIiwiZXhwIjoxNTg5MTIwODc0LCJpZGVudGl0eSI6eyJ1IjoiRnJlcXRyYWRlciJ9LCJmcmVzaCI6ZmFsc2UsInR5cGUiOiJhY2Nlc3MifQ.1seHlII3WprjjclY6DpRhen0rqdF4j6jbvxIhUFaSbs"}
```

--8<-- "includes/cors.md"
