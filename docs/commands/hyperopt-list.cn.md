``` output
usage: freqtrade hyperopt-list [-h] [-v] [--no-color] [--logfile FILE] [-V]
                               [-c PATH] [-d PATH] [--userdir PATH] [--best]
                               [--profitable] [--min-trades INT]
                               [--max-trades INT] [--min-avg-time FLOAT]
                               [--max-avg-time FLOAT] [--min-avg-profit FLOAT]
                               [--max-avg-profit FLOAT]
                               [--min-total-profit FLOAT]
                               [--max-total-profit FLOAT]
                               [--min-objective FLOAT] [--max-objective FLOAT]
                               [--print-json] [--no-details]
                               [--hyperopt-filename FILENAME]
                               [--export-csv FILE]

选项:
  -h, --help            显示帮助信息并退出
  --best                仅选择最佳轮次。
  --profitable          仅选择盈利的轮次。
  --min-trades INT      选择交易数量超过 INT 的轮次。
  --max-trades INT      选择交易数量少于 INT 的轮次。
  --min-avg-time FLOAT  选择平均时间高于该值的轮次。
  --max-avg-time FLOAT  选择平均时间低于该值的轮次。
  --min-avg-profit FLOAT
                        选择平均利润高于该值的轮次。
  --max-avg-profit FLOAT
                        选择平均利润低于该值的轮次。
  --min-total-profit FLOAT
                        选择总利润高于该值的轮次。
  --max-total-profit FLOAT
                        选择总利润低于该值的轮次。
  --min-objective FLOAT
                        选择目标值高于该值的轮次。
  --max-objective FLOAT
                        选择目标值低于该值的轮次。
  --print-json          以 JSON 格式打印输出。
  --no-details          不打印最佳轮次详情。
  --hyperopt-filename FILENAME
                        超参数优化结果文件名。示例：`--hyperopt-
                        filename=hyperopt_results_2020-09-27_16-20-48.pickle`
  --export-csv FILE     导出为 CSV 文件。这将禁用表格打印。
                        示例：--export-csv hyperopt.csv

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
