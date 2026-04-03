``` output
usage: freqtrade trade [-h] [-v] [--no-color] [--logfile FILE] [-V] [-c PATH]
                       [-d PATH] [--userdir PATH] [-s NAME]
                       [--strategy-path PATH] [--recursive-strategy-search]
                       [--freqaimodel NAME] [--freqaimodel-path PATH]
                       [--db-url PATH] [--sd-notify] [--dry-run]
                       [--dry-run-wallet DRY_RUN_WALLET] [--fee FLOAT]

选项:
  -h, --help            显示帮助信息并退出
  --db-url PATH         覆盖交易数据库 URL，这在自定义部署中很有用
                        （默认值：实盘运行模式为 `sqlite:///tradesv3.sqlite`，
                        模拟运行为 `sqlite:///tradesv3.dryrun.sqlite`）。
  --sd-notify           通知 systemd 服务管理器。
  --dry-run             强制模拟运行交易（移除交易所密钥并模拟交易）。
  --dry-run-wallet, --starting-balance DRY_RUN_WALLET
                        起始余额，用于回测/超参数优化和模拟运行。
  --fee FLOAT           指定手续费比率。将应用两次（交易入场和退出时）。

通用参数:
  -v, --verbose         详细模式（-vv 显示更多，-vvv 获取所有消息）。
  --no-color            禁用超参数优化结果的着色。如果你将输出重定向到
                        文件，这可能很有用。
  --logfile, --log-file FILE
                        将日志记录到指定文件。特殊值包括：
                        'syslog'、'journald'。详情请参阅文档。
  -V, --version         显示程序版本号并退出
  -c, --config PATH     指定配置文件（默认值：
                        `userdir/config.json` 或 `config.json`，
                        以存在的为准）。可以使用多个 --config 选项。
                        可以设置为 `-` 从标准输入读取配置。
  -d, --datadir, --data-dir PATH
                        包含历史回测数据的交易所基础目录路径。
                        要查看合约数据，需额外使用 trading-mode。
  --userdir, --user-data-dir PATH
                        用户数据目录路径。

策略参数:
  -s, --strategy NAME   指定机器人将使用的策略类名。
  --strategy-path PATH  指定额外的策略查找路径。
  --recursive-strategy-search
                        在策略文件夹中递归搜索策略。
  --freqaimodel NAME    指定自定义 freqaimodels。
  --freqaimodel-path PATH
                        指定 freqaimodels 的额外查找路径。

```
