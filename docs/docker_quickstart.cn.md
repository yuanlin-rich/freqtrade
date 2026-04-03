# 使用 Docker 运行 Freqtrade

本页面介绍如何使用 Docker 运行机器人。它并不是开箱即用的。你仍然需要阅读文档并了解如何正确配置它。

## 安装 Docker

首先下载并安装适用于你平台的 Docker / Docker Desktop：

* [Mac](https://docs.docker.com/docker-for-mac/install/)
* [Windows](https://docs.docker.com/docker-for-windows/install/)
* [Linux](https://docs.docker.com/install/)

!!! Info "Docker compose 安装"
    Freqtrade 文档假设使用 Docker Desktop（或 docker compose 插件）。
    虽然 docker-compose 独立安装仍然可以工作，但需要将所有 `docker compose` 命令从 `docker compose` 更改为 `docker-compose` 才能工作（例如 `docker compose up -d` 将变为 `docker-compose up -d`）。

??? Warning "Windows 上的 Docker"
    如果你刚在 Windows 系统上安装了 Docker，请确保重启系统，否则可能会遇到与 Docker 容器网络连接相关的无法解释的问题。

## 使用 Docker 运行 Freqtrade

Freqtrade 在 [Dockerhub](https://hub.docker.com/r/freqtradeorg/freqtrade/) 上提供官方 Docker 镜像，以及一个可直接使用的 [docker compose 文件](https://github.com/freqtrade/freqtrade/blob/stable/docker-compose.yml)。

!!! Note
    - 以下部分假设 `docker` 已安装并可供登录用户使用。
    - 以下所有命令使用相对目录，必须从包含 `docker-compose.yml` 文件的目录中执行。

### Docker 快速开始

创建一个新目录并将 [docker-compose 文件](https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml) 放入此目录。

``` bash
mkdir ft_userdata
cd ft_userdata/
# Download the docker-compose file from the repository
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml

# Pull the freqtrade image
docker compose pull

# Create user directory structure
docker compose run --rm freqtrade create-userdir --userdir user_data

# Create configuration - Requires answering interactive questions
docker compose run --rm freqtrade new-config --config user_data/config.json
```

上面的代码片段创建了一个名为 `ft_userdata` 的新目录，下载最新的 compose 文件并拉取 freqtrade 镜像。
代码片段中的最后 2 个步骤创建了包含 `user_data` 的目录，以及（交互式地）根据你的选择创建默认配置。

!!! Question "如何编辑机器人配置？"
    你可以随时编辑配置，使用上述配置时，配置文件位于 `user_data/config.json`（在 `ft_userdata` 目录内）。

    你也可以通过编辑 `docker-compose.yml` 文件的命令部分来更改策略和命令。

#### 添加自定义策略

1. 配置现在位于 `user_data/config.json`
2. 将自定义策略复制到 `user_data/strategies/` 目录
3. 将策略的类名添加到 `docker-compose.yml` 文件中

默认运行的是 `SampleStrategy`。

!!! Danger "`SampleStrategy` 只是一个演示！"
    `SampleStrategy` 仅供参考，为你的策略提供思路。
    在投入真实资金之前，请始终回测你的策略并使用模拟运行一段时间！
    你可以在 [策略文档](strategy-customization.md) 中找到更多关于策略开发的信息。

完成此操作后，你就可以在交易模式下启动机器人了（模拟运行或实盘交易，取决于你对上述相应问题的回答）。

``` bash
docker compose up -d
```

!!! Warning "默认配置"
    虽然生成的配置大部分是可用的，但在启动机器人之前，你仍然需要验证所有选项是否符合你的要求（如定价、交易对列表等）。

#### 访问 UI

如果你在 `new-config` 步骤中选择启用 FreqUI，你将在 `localhost:8080` 端口获得 freqUI。

你现在可以通过在浏览器中输入 localhost:8080 来访问 UI。

??? Note "在远程服务器上访问 UI"
    如果你在 VPS 上运行，应考虑使用 SSH 隧道或设置 VPN（OpenVPN、WireGuard）来连接你的机器人。
    这将确保 freqUI 不直接暴露在互联网上，出于安全原因不建议这样做（freqUI 不支持开箱即用的 https）。
    这些工具的设置不在本教程范围内，但互联网上有许多好的教程。
    请同时阅读 [使用 Docker 的 API 配置](rest-api.md#configuration-with-docker) 部分以了解更多关于此配置的信息。

#### 监控机器人

你可以使用 `docker compose ps` 检查正在运行的实例。
这应该将 `freqtrade` 服务列为 `running`。如果不是，最好检查日志（见下一点）。

#### Docker compose 日志

日志将写入：`user_data/logs/freqtrade.log`。
你也可以使用命令 `docker compose logs -f` 查看最新日志。

#### 数据库

数据库将位于：`user_data/tradesv3.sqlite`

#### 使用 Docker 更新 Freqtrade

使用 `docker` 时更新 Freqtrade 只需运行以下 2 个命令：

``` bash
# Download the latest image
docker compose pull
# Restart the image
docker compose up -d
```

这将首先拉取最新的镜像，然后使用刚拉取的版本重新启动容器。

!!! Warning "查看更新日志"
    你应该始终检查更新日志中的破坏性更改/需要手动干预的内容，并确保机器人在更新后正确启动。

### 编辑 docker-compose 文件

高级用户可以进一步编辑 docker-compose 文件以包含所有可能的选项或参数。

所有 freqtrade 参数都可以通过运行 `docker compose run --rm freqtrade <command> <optional arguments>` 获得。

!!! Warning "交易命令使用 `docker compose`"
    交易命令（`freqtrade trade <...>`）不应通过 `docker compose run` 运行——而应使用 `docker compose up -d`。
    这确保容器正确启动（包括端口转发），并确保容器在系统重启后会重新启动。
    如果你打算使用 freqUI，请同时确保相应地调整 [配置](rest-api.md#configuration-with-docker)，否则 UI 将不可用。

!!! Note "`docker compose run --rm`"
    包含 `--rm` 将在完成后删除容器，强烈建议在除交易模式（使用 `freqtrade trade` 命令运行）之外的所有模式中使用。

??? Note "不使用 docker compose 而使用 docker"
    "`docker compose run --rm`" 将需要提供一个 compose 文件。
    一些不需要认证的 freqtrade 命令（如 `list-pairs`）可以使用 "`docker run --rm`" 代替。
    例如 `docker run --rm freqtradeorg/freqtrade:stable list-pairs --exchange binance --quote BTC --print-json`。
    这对于在不影响正在运行的容器的情况下获取交易所信息以添加到 `config.json` 中很有用。

#### 示例：使用 Docker 下载数据

从 Binance 下载 ETH/BTC 交易对 5 天的 1 小时时间周期回测数据。数据将存储在主机的 `user_data/data/` 目录中。

``` bash
docker compose run --rm freqtrade download-data --pairs ETH/BTC --exchange binance --days 5 -t 1h
```

前往 [数据下载文档](data-download.md) 了解更多关于下载数据的详情。

#### 示例：使用 Docker 回测

在 Docker 容器中对 SampleStrategy 运行回测，使用指定时间范围的历史数据，5 分钟时间周期：

``` bash
docker compose run --rm freqtrade backtesting --config user_data/config.json --strategy SampleStrategy --timerange 20190801-20191001 -i 5m
```

前往 [回测文档](backtesting.md) 了解更多。

### 使用 Docker 添加额外依赖

如果你的策略需要默认镜像中未包含的依赖——则需要在你的主机上构建镜像。
为此，请创建一个 Dockerfile，其中包含额外依赖的安装步骤（查看 [docker/Dockerfile.custom](https://github.com/freqtrade/freqtrade/blob/develop/docker/Dockerfile.custom) 作为示例）。

然后你还需要修改 `docker-compose.yml` 文件，取消注释构建步骤，并重命名镜像以避免命名冲突。

``` yaml
    image: freqtrade_custom
    build:
      context: .
      dockerfile: "./Dockerfile.<yourextension>"
```

然后你可以运行 `docker compose build --pull` 来构建 Docker 镜像，并使用上述命令运行它。

### 使用 Docker 绘图

通过在 `docker-compose.yml` 文件中将镜像更改为 `*_plot`，可以使用 `freqtrade plot-profit` 和 `freqtrade plot-dataframe` 命令（[文档](plotting.md)）。
然后你可以如下使用这些命令：

``` bash
docker compose run --rm freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --timerange=20180801-20180805
```

输出将存储在 `user_data/plot` 目录中，可以使用任何现代浏览器打开。

### 使用 Docker compose 进行数据分析

Freqtrade 提供了一个 docker-compose 文件，用于启动 Jupyter Lab 服务器。
你可以使用以下命令运行此服务器：

``` bash
docker compose -f docker/docker-compose-jupyter.yml up
```

这将创建一个运行 Jupyter Lab 的 Docker 容器，可通过 `https://127.0.0.1:8888/lab` 访问。
请使用启动后控制台中打印的链接进行简化登录。

由于此镜像的一部分在你的机器上构建，建议不时重新构建镜像以保持 freqtrade（和依赖项）的最新状态。

``` bash
docker compose -f docker/docker-compose-jupyter.yml build --no-cache
```

## 故障排除

### Windows 上的 Docker

* 错误：`"Timestamp for this request is outside of the recvWindow."`
  市场 API 请求需要同步时钟，但 Docker 容器中的时间会随着时间推移逐渐向过去偏移。
  要临时修复此问题，你需要运行 `wsl --shutdown` 并重新启动 Docker（Windows 10 上会弹出提示要求你这样做）。
  永久解决方案是在 Linux 主机上托管 Docker 容器，或使用计划任务定期重启 WSL。

  ``` bash
  taskkill /IM "Docker Desktop.exe" /F
  wsl --shutdown
  start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"
  ```

* 无法连接到 API（Windows）
  如果你在 Windows 上且刚安装了 Docker（Desktop），请确保重启系统。Docker 在没有重启的情况下可能会出现网络连接问题。
  你显然还应确保你的 [设置](#accessing-the-ui) 相应配置。

!!! Warning
    由于以上原因，我们不建议在 Windows 上使用 Docker 进行生产部署，仅建议用于实验、数据下载和回测。
    最好使用 Linux VPS 来可靠地运行 freqtrade。
