# SQL 帮助手册

本页面包含一些有关查询 sqlite 数据库的帮助信息。

!!! Tip "其他数据库系统"
    要使用 PostgreSQL 或 MariaDB 等其他数据库系统，您可以使用相同的查询语句，但需要使用相应数据库系统的客户端。[点击此处](advanced-setup.md#use-a-different-database-system)了解如何使用 freqtrade 设置不同的数据库系统。

!!! Warning
    如果您不熟悉 SQL，在数据库上运行查询时应非常小心。
    在运行任何查询之前，务必确保已备份数据库。

## 安装 sqlite3

Sqlite3 是一个基于终端的 sqlite 应用程序。
如果您更习惯使用可视化数据库编辑器，可以使用 SqliteBrowser。

### Ubuntu/Debian 安装

```bash
sudo apt-get install sqlite3
```

### 通过 docker 使用 sqlite3

freqtrade docker 镜像包含 sqlite3，因此您可以在不在主机系统上安装任何东西的情况下编辑数据库。

``` bash
docker compose exec freqtrade /bin/bash
sqlite3 <database-file>.sqlite
```

## 打开数据库

```bash
sqlite3
.open <filepath>
```

## 表结构

### 列出表

```bash
.tables
```

### 显示表结构

```bash
.schema <table_name>
```

### 获取表中的所有交易

```sql
SELECT * FROM trades;
```

## 破坏性查询

写入数据库的查询。
这些查询通常不应该是必要的，因为 freqtrade 会尝试自行处理所有数据库操作 - 或者通过 API 或 telegram 命令暴露它们。

!!! Warning
    请确保在运行以下任何查询之前已备份数据库。

!!! Danger
    您也**绝不**应该在机器人连接到数据库时运行任何写入查询（`update`、`insert`、`delete`）。
    这会导致数据损坏 - 很可能无法恢复。

### 修复在交易所手动卖出后仍然显示为未平仓的交易

!!! Warning
    在交易所手动卖出交易对不会被机器人检测到，它仍会尝试卖出。在可能的情况下，应使用 /forceexit <tradeid> 来完成相同的操作。
    强烈建议在进行任何手动更改之前备份数据库文件。

!!! Note
    在 /forceexit 之后不应需要此操作，因为强制退出的订单会在下一次迭代中被机器人自动关闭。

```sql
UPDATE trades
SET is_open=0,
  close_date=<close_date>,
  close_rate=<close_rate>,
  close_profit = close_rate / open_rate - 1,
  close_profit_abs = (amount * <close_rate> * (1 - fee_close) - (amount * (open_rate * (1 - fee_open)))),
  exit_reason=<exit_reason>
WHERE id=<trade_ID_to_update>;
```

#### 示例

```sql
UPDATE trades
SET is_open=0,
  close_date='2020-06-20 03:08:45.103418',
  close_rate=0.19638016,
  close_profit=0.0496,
  close_profit_abs = (amount * 0.19638016 * (1 - fee_close) - (amount * (open_rate * (1 - fee_open)))),
  exit_reason='force_exit'
WHERE id=31;
```

### 从数据库中删除交易

!!! Tip "使用 RPC 方法删除交易"
    考虑通过 telegram 或 rest API 使用 `/delete <tradeid>`。这是删除交易的推荐方式，因为它还会删除相应的订单和自定义数据，并且会在机器人中触发必要的事件以保持所有内容同步。

如果您仍然想直接从数据库中删除交易，可以使用以下查询。

!!! Danger
    某些系统（Ubuntu）在其 sqlite3 软件包中禁用了外键。使用 sqlite 时，请确保在上述查询之前运行 `PRAGMA foreign_keys = ON` 来启用外键。

```sql
DELETE FROM trades WHERE id = <tradeid>;
DELETE FROM orders WHERE ft_trade_id = <tradeid>;
DELETE FROM trade_custom_data WHERE ft_trade_id = <tradeid>;


DELETE FROM trades WHERE id = 31;
DELETE FROM orders WHERE ft_trade_id = 31;
DELETE FROM trade_custom_data WHERE ft_trade_id = 31;
```

!!! Warning
    这将从数据库中删除指定的交易。请确保您获取了正确的 id，并且**绝不**在没有 `where` 子句的情况下运行此查询。
