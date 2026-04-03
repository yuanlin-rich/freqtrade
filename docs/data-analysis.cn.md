# 使用 Jupyter 笔记本分析机器人数据

您可以使用 Jupyter 笔记本轻松分析回测和交易历史记录的结果。在使用 `freqtrade create-userdir --userdir user_data` 初始化用户目录后，示例笔记本位于 `user_data/notebooks/`。

## 使用 Docker 快速开始

Freqtrade 提供了一个 docker-compose 文件，用于启动 Jupyter Lab 服务器。
您可以使用以下命令运行此服务器：`docker compose -f docker/docker-compose-jupyter.yml up`

这将创建一个运行 Jupyter Lab 的 Docker 容器，可通过 `https://127.0.0.1:8888/lab` 访问。
请使用启动后控制台中打印的链接进行简化登录。

更多信息请访问 [使用 Docker 进行数据分析](docker_quickstart.md#data-analysis-using-docker-compose) 部分。

### 专业技巧

* 请参阅 [jupyter.org](https://jupyter.org/documentation) 了解使用说明。
* 不要忘记从您的 conda 或 venv 环境中启动 Jupyter 笔记本服务器，或者使用 [nb_conda_kernels](https://github.com/Anaconda-Platform/nb_conda_kernels)*
* 在使用前复制示例笔记本，以免您的更改在下次 freqtrade 更新时被覆盖。

### 在系统范围的 Jupyter 安装中使用虚拟环境

有时可能需要使用系统范围的 Jupyter 笔记本安装，并使用来自虚拟环境的 Jupyter 内核。
这可以避免在每个系统上多次安装完整的 Jupyter 套件，并提供一种在任务（freqtrade / 其他分析任务）之间轻松切换的方式。

要实现此功能，请先激活您的虚拟环境并运行以下命令：

``` bash
# Activate virtual environment
source .venv/bin/activate

pip install ipykernel
ipython kernel install --user --name=freqtrade
# Restart jupyter (lab / notebook)
# select kernel "freqtrade" in the notebook
```

!!! Note
    本节仅为完整性而提供，Freqtrade 团队不会为此设置的问题提供完整支持，并建议直接在虚拟环境中安装 Jupyter，因为这是启动和运行 Jupyter 笔记本最简单的方式。如需此设置的帮助，请参阅 [Project Jupyter](https://jupyter.org/) 的[文档](https://jupyter.org/documentation)或[帮助频道](https://jupyter.org/community)。

!!! Warning
    某些任务在笔记本中运行效果不佳。例如，任何使用异步执行的操作都会给 Jupyter 带来问题。此外，freqtrade 的主要入口点是 shell 命令行，因此在笔记本中使用纯 Python 会绕过为辅助函数提供所需对象和参数的参数。您可能需要手动设置这些值或创建预期的对象。

## 推荐的工作流程

| 任务 | 工具 |
  --- | ---
机器人操作 | CLI
重复性任务 | Shell 脚本
数据分析和可视化 | 笔记本

1. 使用 CLI 来

    * 下载历史数据
    * 运行回测
    * 使用实时数据运行
    * 导出结果

1. 将这些操作收集到 Shell 脚本中

    * 保存带有参数的复杂命令
    * 执行多步骤操作
    * 自动化测试策略和准备分析数据

1. 使用笔记本来

    * 可视化数据
    * 处理和绘图以生成洞察

## 示例实用代码片段

### 切换到根目录

Jupyter 笔记本从笔记本目录执行。以下代码片段搜索项目根目录，以便相对路径保持一致。

```python
import os
from pathlib import Path

# Change directory
# Modify this cell to insure that the output shows the correct path.
# Define all paths relative to the project root shown in the cell output
project_root = "somedir/freqtrade"
i=0
try:
    os.chdir(project_root)
    assert Path('LICENSE').is_file()
except:
    while i<4 and (not Path('LICENSE').is_file()):
        os.chdir(Path(Path.cwd(), '../'))
        i+=1
    project_root = Path.cwd()
print(Path.cwd())
```

### 加载多个配置文件

此选项可用于检查传入多个配置文件的结果。
这也会运行整个 Configuration 初始化过程，因此配置会被完全初始化，以便传递给其他方法。

``` python
import json
from freqtrade.configuration import Configuration

# Load config from multiple files
config = Configuration.from_files(["config1.json", "config2.json"])

# Show the config in memory
print(json.dumps(config['original_config'], indent=2))
```

对于交互式环境，请准备一个额外的配置文件指定 `user_data_dir`，并最后传入，这样您就不必在运行机器人时切换目录。
最好避免使用相对路径，因为它是从 Jupyter 笔记本的存储位置开始的，除非目录已被更改。

``` json
{
    "user_data_dir": "~/.freqtrade/"
}
```

### 更多数据分析文档

* [策略调试](strategy_analysis_example.md) - 也可作为 Jupyter 笔记本使用（`user_data/notebooks/strategy_analysis_example.ipynb`）
* [绘图](plotting.md)
* [标签分析](advanced-backtesting.md)

如果您想分享关于如何最好地分析数据的想法，请随时提交 issue 或 Pull Request 来增强此文档。
