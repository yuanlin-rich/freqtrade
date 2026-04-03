``` output
usage: freqtrade hyperopt-show [-h] [-v] [--no-color] [--logfile FILE] [-V]
                               [-c PATH] [-d PATH] [--userdir PATH] [--best]
                               [--profitable] [-n INT] [--print-json]
                               [--hyperopt-filename FILENAME] [--no-header]
                               [--disable-param-export]
                               [--breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]]

选项:
  -h, --help            显示帮助信息并退出
  --best                仅选择最佳轮次。
  --profitable          仅选择盈利的轮次。
  -n, --index INT       指定要打印详情的轮次索引。
  --print-json          以 JSON 格式打印输出。
  --hyperopt-filename FILENAME
                        超参数优化结果文件名。示例：`--hyperopt-
                        filename=hyperopt_results_2020-09-27_16-20-48.pickle`
  --no-header           不打印轮次详情标题。
  --disable-param-export
                        禁用自动超参数优化参数导出。
  --breakdown {day,week,month,year,weekday} [{day,week,month,year,weekday} ...]
                        按 [天、周、月、年、星期几] 显示回测细分。

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

```
