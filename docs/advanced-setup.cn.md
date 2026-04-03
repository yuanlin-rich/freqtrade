# 高级安装后任务

本页面说明了一些可以在机器人安装后执行的高级任务和配置选项，这些内容在某些环境中可能很有用。

如果你不了解这里提到的内容，那你可能不需要它。

## 运行多个 Freqtrade 实例

本节将向你展示如何在同一台机器上同时运行多个机器人。

### 需要考虑的事项

* 使用不同的数据库文件。
* 使用不同的 Telegram 机器人（需要多个不同的配置文件；仅在启用 Telegram 时适用）。
* 使用不同的端口（仅在启用 Freqtrade REST API 网络服务器时适用）。

### 不同的数据库文件

为了跟踪你的交易、利润等信息，freqtrade 使用 SQLite 数据库来存储各类信息，例如你过去执行的交易和当前持有的仓位。这使你能够跟踪利润，但更重要的是，如果机器人进程重启或意外终止，可以跟踪正在进行的活动。

默认情况下，Freqtrade 会为模拟运行和实盘机器人使用单独的数据库文件（假设配置中和命令行参数中都没有指定 database-url）。
对于实盘交易模式，默认数据库为 `tradesv3.sqlite`，模拟运行则为 `tradesv3.dryrun.sqlite`。

用于指定这些文件路径的 trade 命令可选参数是 `--db-url`，它需要一个有效的 SQLAlchemy URL。
因此，当你在模拟运行模式下仅使用配置和策略参数启动机器人时，以下两个命令的效果是相同的。

``` bash
freqtrade trade -c MyConfig.json -s MyStrategy
# is equivalent to
freqtrade trade -c MyConfig.json -s MyStrategy --db-url sqlite:///tradesv3.dryrun.sqlite
```

这意味着如果你在两个不同的终端中运行 trade 命令，例如在一个实例中测试 USDT 交易对的策略，在另一个实例中测试 BTC 交易对的策略，你需要使用不同的数据库。

如果你指定的数据库 URL 对应的数据库不存在，freqtrade 会以你指定的名称创建一个。因此，要用 BTC 和 USDT 作为计价货币测试你的自定义策略，你可以使用以下命令（在两个单独的终端中）：

``` bash
# Terminal 1:
freqtrade trade -c MyConfigBTC.json -s MyCustomStrategy --db-url sqlite:///user_data/tradesBTC.dryrun.sqlite
# Terminal 2:
freqtrade trade -c MyConfigUSDT.json -s MyCustomStrategy --db-url sqlite:///user_data/tradesUSDT.dryrun.sqlite
```

反之，如果你希望在生产模式中做同样的事情，你也需要创建至少一个新的数据库（除了默认的之外），并指定"实盘"数据库的路径，例如：

``` bash
# Terminal 1:
freqtrade trade -c MyConfigBTC.json -s MyCustomStrategy --db-url sqlite:///user_data/tradesBTC.live.sqlite
# Terminal 2:
freqtrade trade -c MyConfigUSDT.json -s MyCustomStrategy --db-url sqlite:///user_data/tradesUSDT.live.sqlite
```

有关 sqlite 数据库使用的更多信息，例如手动输入或删除交易，请参阅 [SQL 速查表](sql_cheatsheet.md)。

### 使用 Docker 运行多个实例

要使用 Docker 运行多个 freqtrade 实例，你需要编辑 docker-compose.yml 文件，并将所有你想要的实例添加为单独的服务。请记住，你可以将配置拆分为多个文件，因此最好考虑使它们模块化，这样如果你需要编辑所有机器人的共同配置，可以在单个配置文件中完成。
``` yml
---
version: '3'
services:
  freqtrade1:
    image: freqtradeorg/freqtrade:stable
    # image: freqtradeorg/freqtrade:develop
    # Use plotting image
    # image: freqtradeorg/freqtrade:develop_plot
    # Build step - only needed when additional dependencies are needed
    # build:
    #   context: .
    #   dockerfile: "./docker/Dockerfile.custom"
    restart: always
    container_name: freqtrade1
    volumes:
      - "./user_data:/freqtrade/user_data"
    # Expose api on port 8080 (localhost only)
    # Please read the https://www.freqtrade.io/en/stable/rest-api/ documentation
    # before enabling this.
     ports:
     - "127.0.0.1:8080:8080"
    # Default command used when running `docker compose up`
    command: >
      trade
      --logfile /freqtrade/user_data/logs/freqtrade1.log
      --db-url sqlite:////freqtrade/user_data/tradesv3_freqtrade1.sqlite
      --config /freqtrade/user_data/config.json
      --config /freqtrade/user_data/config.freqtrade1.json
      --strategy SampleStrategy

  freqtrade2:
    image: freqtradeorg/freqtrade:stable
    # image: freqtradeorg/freqtrade:develop
    # Use plotting image
    # image: freqtradeorg/freqtrade:develop_plot
    # Build step - only needed when additional dependencies are needed
    # build:
    #   context: .
    #   dockerfile: "./docker/Dockerfile.custom"
    restart: always
    container_name: freqtrade2
    volumes:
      - "./user_data:/freqtrade/user_data"
    # Expose api on port 8080 (localhost only)
    # Please read the https://www.freqtrade.io/en/stable/rest-api/ documentation
    # before enabling this.
    ports:
      - "127.0.0.1:8081:8080"
    # Default command used when running `docker compose up`
    command: >
      trade
      --logfile /freqtrade/user_data/logs/freqtrade2.log
      --db-url sqlite:////freqtrade/user_data/tradesv3_freqtrade2.sqlite
      --config /freqtrade/user_data/config.json
      --config /freqtrade/user_data/config.freqtrade2.json
      --strategy SampleStrategy

```

你可以使用任何你想要的命名约定，freqtrade1 和 2 是任意的。注意，如上所述，你需要为每个实例使用不同的数据库文件、端口映射和 Telegram 配置。

## 使用不同的数据库系统

Freqtrade 使用 SQLAlchemy，它支持多种不同的数据库系统。因此，应该可以支持多种数据库系统。
Freqtrade 不依赖或安装任何额外的数据库驱动程序。请参阅 [SQLAlchemy 文档](https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls) 了解各数据库系统的安装说明。

以下系统已经过测试，确认可以与 freqtrade 配合使用：

* sqlite（默认）
* PostgreSQL
* MariaDB

!!! Warning
    使用以下任何数据库系统，即表示你了解如何管理此类系统。freqtrade 团队不会为以下数据库系统的设置或维护（或备份）提供任何支持。

### PostgreSQL

安装：
`pip install "psycopg[binary]"`

用法：
`... --db-url postgresql+psycopg://<username>:<password>@localhost:5432/<database>`

Freqtrade 将在启动时自动创建必要的表。

如果你运行不同的 Freqtrade 实例，你必须为每个实例设置一个数据库，或者为你的连接使用不同的用户/模式。

### MariaDB / MySQL

Freqtrade 通过使用 SQLAlchemy 支持 MariaDB，SQLAlchemy 支持多种不同的数据库系统。

安装：
`pip install pymysql`

用法：
`... --db-url mysql+pymysql://<username>:<password>@localhost:3306/<database>`



## 将机器人配置为 systemd 服务运行

将 `freqtrade.service` 文件复制到你的 systemd 用户目录（通常是 `~/.config/systemd/user`），并更新 `WorkingDirectory` 和 `ExecStart` 以匹配你的设置。

!!! Note
    某些系统（如 Raspbian）不会从用户目录加载服务单元文件。在这种情况下，请将 `freqtrade.service` 复制到 `/etc/systemd/user/`（需要超级用户权限）。

之后你可以使用以下命令启动守护进程：

```bash
systemctl --user start freqtrade
```

要使其持久化（在用户注销后仍然运行），你需要为你的 freqtrade 用户启用 `linger`。

```bash
sudo loginctl enable-linger "$USER"
```

如果你将机器人作为服务运行，可以使用 systemd 服务管理器作为软件看门狗来监控 freqtrade 机器人的状态，并在故障时重启它。如果在配置中将 `internals.sd_notify` 参数设置为 true，或者使用了 `--sd-notify` 命令行选项，机器人将使用 sd_notify（systemd 通知）协议向 systemd 发送保活 ping 消息，并在状态变化时告诉 systemd 其当前状态（Running、Paused 或 Stopped）。

`freqtrade.service.watchdog` 文件包含了一个使用 systemd 作为看门狗的服务单元配置文件示例。

!!! Note
    如果机器人运行在 Docker 容器中，机器人和 systemd 服务管理器之间的 sd_notify 通信将不起作用。

## 高级日志

Freqtrade 使用 Python 提供的默认日志模块。
Python 在这方面允许进行广泛的[日志配置](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig) —— 远超此处所能涵盖的内容。

如果你的 freqtrade 配置中没有提供 `log_config`，则默认设置彩色终端输出的日志格式。
使用 `--logfile logfile.log` 将启用 RotatingFileHandler。

如果你对日志格式或 RotatingFileHandler 提供的默认设置不满意，你可以通过在 freqtrade 配置文件中添加 `log_config` 配置来自定义日志。

默认配置大致如下所示，文件处理程序已提供但未启用，因为 `filename` 被注释掉了。
取消注释该行并提供有效的路径/文件名即可启用。

``` json hl_lines="5-7 13-16 27"
{
  "log_config": {
      "version": 1,
      "formatters": {
          "basic": {
              "format": "%(message)s"
          },
          "standard": {
              "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
          }
      },
      "handlers": {
          "console": {
              "class": "freqtrade.loggers.ft_rich_handler.FtRichHandler",
              "formatter": "basic"
          },
          "file": {
              "class": "logging.handlers.RotatingFileHandler",
              "formatter": "standard",
              // "filename": "someRandomLogFile.log",
              "maxBytes": 10485760,
              "backupCount": 10
          }
      },
      "root": {
          "handlers": [
              "console",
              // "file"
          ],
          "level": "INFO",
      }
  }
}
```

!!! Note "高亮行"
    上面代码块中的高亮行定义了 Rich 处理程序，它们是一组相关联的配置。
    格式化器 "standard" 和 "file" 属于 FileHandler。

每个处理程序必须使用已定义的格式化器之一（按名称引用），其类必须可用，且必须是有效的日志类。
要实际使用一个处理程序，它必须在 "root" 段内的 "handlers" 部分中。
如果省略此部分，freqtrade 将不会提供任何输出（在未配置的处理程序中）。

!!! Tip "显式日志配置"
    我们建议将日志配置从主 freqtrade 配置文件中提取出来，并通过[多配置文件](configuration.md#multiple-configuration-files)功能提供给你的机器人。这将避免不必要的代码重复。

---

在许多 Linux 系统上，可以将机器人配置为将日志消息发送到 `syslog` 或 `journald` 系统服务。在 Windows 上也可以将日志发送到远程 `syslog` 服务器。可以使用 `--logfile` 命令行选项的特殊值来实现这一点。

### 记录日志到 syslog

要将 Freqtrade 日志消息发送到本地或远程 `syslog` 服务，请使用 `"log_config"` 设置选项来配置日志。

``` json
{
  // ...
  "log_config": {
    "version": 1,
    "formatters": {
      "syslog_fmt": {
        "format": "%(name)s - %(levelname)s - %(message)s"
      }
    },
    "handlers": {
      // Other handlers?
      "syslog": {
         "class": "logging.handlers.SysLogHandler",
          "formatter": "syslog_fmt",
          // Use one of the other options above as address instead?
          "address": "/dev/log"
      }
    },
    "root": {
      "handlers": [
        // other handlers
        "syslog",

      ]
    }

  }
}
```

可能需要配置[额外的日志处理程序](#高级日志)，例如同时在控制台中输出日志。

#### Syslog 使用方法

日志消息以 `user` 设施发送到 `syslog`。因此你可以使用以下命令查看它们：

* `tail -f /var/log/user`，或者
* 安装一个综合的图形查看器（例如 Ubuntu 的 'Log File Viewer'）。

在许多系统上，`syslog`（`rsyslog`）从 `journald` 获取数据（反之亦然），因此 syslog 或 journald 都可以使用，消息可以通过 `journalctl` 和 syslog 查看工具查看。你可以以任何适合你的方式组合使用。

对于 `rsyslog`，可以将机器人的消息重定向到单独的专用日志文件。为此，请添加

```
if $programname startswith "freqtrade" then -/var/log/freqtrade.log
```

到某个 rsyslog 配置文件中，例如 `/etc/rsyslog.d/50-default.conf` 的末尾。

对于 `syslog`（`rsyslog`），可以开启消息去重模式。这将减少重复消息的数量。例如，当机器人没有其他活动时，多条心跳消息将被合并为单条消息。为此，在 `/etc/rsyslog.conf` 中设置：

```
# Filter duplicated messages
$RepeatedMsgReduction on
```

#### Syslog 地址

syslog 地址可以是 Unix 域套接字（套接字文件名）或 UDP 套接字规范（由 IP 地址和 UDP 端口组成，以 `:` 字符分隔）。

因此，以下是可能地址的示例：

* `"address": "/dev/log"` -- 使用 `/dev/log` 套接字记录到 syslog（rsyslog），适用于大多数系统。
* `"address": "/var/run/syslog"` -- 使用 `/var/run/syslog` 套接字记录到 syslog（rsyslog）。在 MacOS 上使用此选项。
* `"address": "localhost:514"` -- 使用 UDP 套接字记录到本地 syslog（如果它监听 514 端口）。
* `"address": "<ip>:514"` -- 记录到指定 IP 地址和 514 端口的远程 syslog。这可在 Windows 上用于远程记录到外部 syslog 服务器。

??? Info "已弃用 - 通过命令行配置 syslog"
    `--logfile syslog:<syslog_address>` -- 使用 `<syslog_address>` 作为 syslog 地址将日志消息发送到 `syslog` 服务。

    syslog 地址可以是 Unix 域套接字（套接字文件名）或 UDP 套接字规范（由 IP 地址和 UDP 端口组成，以 `:` 字符分隔）。

    因此，以下是可能用法的示例：

    * `--logfile syslog:/dev/log` -- 使用 `/dev/log` 套接字记录到 syslog（rsyslog），适用于大多数系统。
    * `--logfile syslog` -- 同上，`/dev/log` 的快捷方式。
    * `--logfile syslog:/var/run/syslog` -- 使用 `/var/run/syslog` 套接字记录到 syslog（rsyslog）。在 MacOS 上使用此选项。
    * `--logfile syslog:localhost:514` -- 使用 UDP 套接字记录到本地 syslog（如果它监听 514 端口）。
    * `--logfile syslog:<ip>:514` -- 记录到指定 IP 地址和 514 端口的远程 syslog。这可在 Windows 上用于远程记录到外部 syslog 服务器。

### 记录日志到 journald

此功能需要安装 `cysystemd` Python 包作为依赖（`pip install cysystemd`），该包在 Windows 上不可用。因此，整个 journald 日志功能在 Windows 上运行的机器人中不可用。

要将 Freqtrade 日志消息发送到 `journald` 系统服务，请在配置中添加以下配置片段。

``` json
{
  // ...
  "log_config": {
    "version": 1,
    "formatters": {
      "journald_fmt": {
        "format": "%(name)s - %(levelname)s - %(message)s"
      }
    },
    "handlers": {
      // Other handlers?
      "journald": {
         "class": "cysystemd.journal.JournaldLogHandler",
          "formatter": "journald_fmt",
      }
    },
    "root": {
      "handlers": [
        // ..
        "journald",

      ]
    }

  }
}
```

可能需要配置[额外的日志处理程序](#高级日志)，例如同时在控制台中输出日志。

日志消息以 `user` 设施发送到 `journald`。因此你可以使用以下命令查看它们：

* `journalctl -f` -- 显示发送到 `journald` 的 Freqtrade 日志消息以及 `journald` 获取的其他日志消息。
* `journalctl -f -u freqtrade.service` -- 当机器人作为 `systemd` 服务运行时可以使用此命令。

`journalctl` 工具有许多其他选项可以过滤消息，请查阅该工具的手册页。

在许多系统上，`syslog`（`rsyslog`）从 `journald` 获取数据（反之亦然），因此 `--logfile syslog` 或 `--logfile journald` 都可以使用，消息可以通过 `journalctl` 和 syslog 查看工具查看。你可以以任何适合你的方式组合使用。

??? Info "已弃用 - 通过命令行配置 journald"
    要将 Freqtrade 日志消息发送到 `journald` 系统服务，请使用 `--logfile` 命令行选项，格式如下：

    `--logfile journald` -- 将日志消息发送到 `journald`。

### JSON 格式的日志

你也可以将默认输出流配置为使用 JSON 格式。
"fmt_dict" 属性定义了 JSON 输出的键 —— 以及 [Python 日志 LogRecord 属性](https://docs.python.org/3/library/logging.html#logrecord-attributes)。

以下配置将默认输出更改为 JSON。同样的格式化器也可以与 `RotatingFileHandler` 结合使用。
我们建议保留一种人类可读的格式。

``` json
{
  // ...
  "log_config": {
    "version": 1,
    "formatters": {
       "json": {
          "()": "freqtrade.loggers.json_formatter.JsonFormatter",
          "fmt_dict": {
              "timestamp": "asctime",
              "level": "levelname",
              "logger": "name",
              "message": "message"
          }
      }
    },
    "handlers": {
      // Other handlers?
      "jsonStream": {
          "class": "logging.StreamHandler",
          "formatter": "json"
      }
    },
    "root": {
      "handlers": [
        // ..
        "jsonStream",

      ]
    }

  }
}
```
