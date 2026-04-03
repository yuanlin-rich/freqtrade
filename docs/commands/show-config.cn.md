``` output
usage: freqtrade show-config [-h] [--userdir PATH] [-c PATH]
                             [--show-sensitive]

options:
  -h, --help            显示此帮助信息并退出
  --userdir, --user-data-dir PATH
                        用户数据目录的路径。
  -c, --config PATH     指定配置文件（默认值：
                        `userdir/config.json` 或 `config.json`，以存在的为准）。
                        可以使用多个 --config 选项。可以设置为 `-` 以从 stdin 读取配置。
  --show-sensitive      在输出中显示敏感信息。

```
