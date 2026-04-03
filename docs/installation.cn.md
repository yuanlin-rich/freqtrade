# 安装

本页面介绍如何准备运行机器人的环境。

freqtrade 文档描述了多种安装 freqtrade 的方式：

* [Docker 镜像](docker_quickstart.md)（单独页面）
* [脚本安装](#script-installation)
* [手动安装](#manual-installation)
* [使用 Conda 安装](#installation-with-conda)

请考虑使用预构建的 [docker 镜像](docker_quickstart.md) 来快速上手。

!!! Note "更新"
    保持 freqtrade 更新对于[确保与交易所 API 的持续兼容性](updating.md#why-update)非常重要。
    请参阅[更新指南](updating.md)了解如何更新安装的详细信息。

!!! Note "Windows 用户"
    我们**强烈**建议 Windows 用户使用 [Docker](docker_quickstart.md)，因为这将更加简单流畅（也更安全）。

    如果无法使用 Docker，请尝试使用 Windows Linux 子系统（WSL）——Ubuntu/Linux 的安装说明同样适用于 WSL。
    如果你确实想在 Windows 上原生安装 freqtrade，最好使用 [`./setup.ps1` 安装脚本](#use-setupps1-windows)。

    另外请确保使用 64 位版本的 Python，因为 32 位版本有严重的内存限制，可能会对回测/超参数优化的体验产生负面影响。

------

## 信息

安装和运行 Freqtrade 最简单的方式是克隆机器人的 Github 仓库，然后运行 `./setup.sh`（Windows 使用 `./setup.ps1`）脚本（如果你的平台支持的话）。

!!! Note "版本注意事项"
    克隆仓库时，默认工作分支名为 `develop`。该分支包含所有最新功能（得益于自动化测试，可以认为是相对稳定的）。
    `stable` 分支包含最新发布版本的代码（通常每月发布一次，基于约一周前的 `develop` 分支快照，以防止打包错误，因此可能更加稳定）。

!!! Note
    假设系统已安装 [uv](https://docs.astral.sh/uv/)，或者 Python3.11 及更高版本以及对应的 `pip`。如果未安装，安装脚本会发出警告并停止。还需要 `git` 来克隆 Freqtrade 仓库。
    此外，还必须安装 python 头文件（`python<yourversion>-dev` / `python<yourversion>-devel`）才能成功完成安装。

!!! Warning "系统时钟需要准确"
    运行机器人的系统时钟必须准确，需要频繁与 NTP 服务器同步，以避免与交易所通信时出现问题。

------

## 系统要求

这些要求同时适用于[脚本安装](#script-installation)和[手动安装](#manual-installation)。

!!! Note "ARM64 系统"
    如果你使用的是 ARM64 系统（如 MacOS M1 或 Oracle VM），请使用 [docker](docker_quickstart.md) 来运行 freqtrade。
    虽然通过一些手动操作可以进行原生安装，但目前不提供官方支持。

### 安装指南

* [Python >= 3.11](http://docs.python-guide.org/en/latest/starting/installation/)
* [pip](https://pip.pypa.io/en/stable/installing/)
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [virtualenv](https://virtualenv.pypa.io/en/stable/installation.html)（推荐）

### 安装代码

我们提供了 Ubuntu、MacOS 和 Windows 的安装说明。这些是指导性的，在其他发行版上的效果可能会有所不同。
下面先列出各操作系统的特定步骤，之后的通用部分适用于所有系统。

!!! Note
    假设系统已安装 Python3.11 或更高版本以及对应的 pip。

=== "Debian/Ubuntu"
    #### 安装必要的依赖

    ```bash
    # update repository
    sudo apt-get update

    # install packages
    sudo apt install -y python3-pip python3-venv python3-dev python3-pandas git curl
    ```

=== "MacOS"
    #### 安装必要的依赖

    如果尚未安装 [Homebrew](https://brew.sh/)，请先安装。

    ```bash
    # install packages
    brew install gettext libomp
    ```
    !!! Note
        `setup.sh` 脚本会自动为你安装这些依赖——前提是你的系统已安装 brew。

=== "RaspberryPi/Raspbian"
    以下假设使用最新的 [Raspbian Buster lite 镜像](https://www.raspberrypi.org/downloads/raspbian/)。
    该镜像预装了 python3.11，可以轻松启动和运行 freqtrade。

    使用 Raspberry Pi 3 和 Raspbian Buster lite 镜像测试通过，已应用所有更新。


    ```bash
    sudo apt-get install python3-venv libatlas-base-dev cmake curl libffi-dev
    # Use piwheels.org to speed up installation
    sudo echo "[global]\nextra-index-url=https://www.piwheels.org/simple" > tee /etc/pip.conf

    git clone https://github.com/freqtrade/freqtrade.git
    cd freqtrade

    bash setup.sh -i
    ```

    !!! Note "安装时间"
        根据你的网速和 Raspberry Pi 版本，安装可能需要数小时才能完成。
        因此，我们建议使用 Raspberry 的预构建 docker 镜像，请参阅 [Docker 快速入门文档](docker_quickstart.md)

    !!! Note
        以上操作不会安装 hyperopt 依赖。如需安装，请使用 `python3 -m pip install -e .[hyperopt]`。
        我们不建议在 Raspberry Pi 上运行 hyperopt，因为这是一个非常消耗资源的操作，应在高性能机器上执行。

------

## Freqtrade 仓库

Freqtrade 是一个开源的加密货币交易机器人，其代码托管在 `github.com` 上。

```bash
# Download `develop` branch of freqtrade repository
git clone https://github.com/freqtrade/freqtrade.git

# Enter downloaded directory
cd freqtrade

# your choice (1): novice user
git checkout stable

# your choice (2): advanced user
git checkout develop
```

(1) 此命令将克隆的仓库切换到 `stable` 分支。如果你希望留在 (2) `develop` 分支，则不需要执行此命令。

之后你可以随时使用 `git checkout stable`/`git checkout develop` 命令在分支之间切换。

??? Note "从 pypi 安装"
    安装 Freqtrade 的另一种方式是从 [pypi](https://pypi.org/project/freqtrade/) 安装。缺点是此方法需要事先正确安装 ta-lib，因此目前不是推荐的安装方式。

    ``` bash
    pip install freqtrade
    ```

------

## 脚本安装

安装 Freqtrade 的第一种方式是使用提供的 Linux/MacOS `./setup.sh` 脚本，该脚本会安装所有依赖并帮助你配置机器人。

确保你满足[系统要求](#requirements)并已下载 [Freqtrade 仓库](#freqtrade-repository)。

### 使用 /setup.sh -install（Linux/MacOS）

如果你使用的是 Debian、Ubuntu 或 MacOS，freqtrade 提供了安装脚本。

```bash
# --install, Install freqtrade from scratch
./setup.sh -i
```

#### /setup.sh 脚本的其他选项

你也可以使用 `./setup.sh` 更新、配置和重置机器人的代码库。

```bash
# --update, Command git pull to update.
./setup.sh -u
# --reset, Hard reset your develop/stable branch.
./setup.sh -r
```

```
** --install **

With this option, the script will install the bot and most dependencies:
You will need to have git and python3.11+ installed beforehand for this to work.

* Mandatory software as: `ta-lib`
* Setup your virtualenv under `.venv/`

This option is a combination of installation tasks and `--reset`

** --update **

This option will pull the last version of your current branch and update your virtualenv. Run the script with this option periodically to update your bot.

** --reset **

This option will hard reset your branch (only if you are on either `stable` or `develop`) and recreate your virtualenv.
```

#### 激活虚拟环境

每次打开新终端时，你必须运行 `source .venv/bin/activate` 来激活虚拟环境。

```bash
# activate virtual environment
source ./.venv/bin/activate
```

### 使用 ./setup.ps1（Windows）

该脚本会询问你几个问题，以确定应安装哪些部分。

```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass
cd freqtrade
. .\setup.ps1
```

#### 激活虚拟环境（Windows）

```powershell
# activate virtual environment
. .\.venv\Scripts\Activate.ps1
```

[现在你已准备就绪](#you-are-ready)，可以运行机器人了。

-----

## 手动安装

确保你满足[系统要求](#requirements)并已下载 [Freqtrade 仓库](#freqtrade-repository)。

### 设置 Python 虚拟环境（virtualenv）

你将在独立的`虚拟环境`中运行 freqtrade。

```bash
# create virtualenv in directory /freqtrade/.venv
python3 -m venv .venv

# run virtualenv
source .venv/bin/activate
```

### 安装 Python 依赖

```bash
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
# install freqtrade
python3 -m pip install -e .
```

[现在你已准备就绪](#you-are-ready)，可以运行机器人了。

### （可选）安装后任务

!!! Note
    如果你在服务器上运行机器人，应考虑使用 [Docker](docker_quickstart.md) 或终端复用器（如 `screen` 或 [`tmux`](https://en.wikipedia.org/wiki/Tmux)），以避免注销时机器人被停止。

在使用 `systemd` 软件套件的 Linux 系统上，作为可选的安装后任务，你可能希望将机器人设置为 `systemd service` 运行，或将其配置为将日志消息发送到 `syslog`/`rsyslog` 或 `journald` 守护进程。详情请参阅[高级日志记录](advanced-setup.md#advanced-logging)。

------

## 使用 Conda 安装

Freqtrade 也可以使用 Miniconda 或 Anaconda 安装。我们推荐使用 Miniconda，因为它的安装体积更小。Conda 将自动准备和管理 Freqtrade 程序所需的大量库依赖。

### 什么是 Conda？

Conda 是一个支持多种编程语言的包管理、依赖管理和环境管理工具：[conda 文档](https://docs.conda.io/projects/conda/en/latest/index.html)

### 使用 conda 安装

#### 安装 Conda

[在 Linux 上安装](https://conda.io/projects/conda/en/latest/user-guide/install/linux.html#install-linux-silent)

[在 Windows 上安装](https://conda.io/projects/conda/en/latest/user-guide/install/windows.html)

回答所有问题。安装完成后，必须关闭并重新打开终端。

#### 下载 Freqtrade

下载并安装 freqtrade。

```bash
# download freqtrade
git clone https://github.com/freqtrade/freqtrade.git

# enter downloaded directory 'freqtrade'
cd freqtrade
```

#### 安装 Freqtrade：Conda 环境

```bash
conda create --name freqtrade python=3.12
```

!!! Note "创建 Conda 环境"
    conda 命令 `create -n` 会自动安装所选库的所有嵌套依赖，安装命令的一般结构为：

    ```bash
    # choose your own packages
    conda env create -n [name of the environment] [python version] [packages]
    ```

#### 进入/退出 freqtrade 环境

要查看可用的环境，请输入：

```bash
conda env list
```

进入已安装的环境：

```bash
# enter conda environment
conda activate freqtrade

# exit conda environment - don't do it now
conda deactivate
```

使用 pip 安装最新的 python 依赖：

```bash
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -e .
```

[现在你已准备就绪](#you-are-ready)，可以运行机器人了。

### 常用快捷命令

```bash
# list installed conda environments
conda env list

# activate base environment
conda activate

# activate freqtrade environment
conda activate freqtrade

#deactivate any conda environments
conda deactivate
```

### 关于 Anaconda 的更多信息

!!! Info "大型软件包"
    可能会出现这样的情况：在创建时就填充所选包的新 Conda 环境，比在已有环境中安装大型库或应用程序花费更少的时间。

!!! Warning "在 conda 中使用 pip install"
    conda 的文档指出不应在 conda 中使用 pip，因为可能会出现内部问题。
    不过这种情况很少见。[Anaconda 博客文章](https://www.anaconda.com/blog/using-pip-in-a-conda-environment)

    尽管如此，这就是为什么推荐使用 `conda-forge` 频道的原因：

    * 提供更多的库（减少使用 `pip` 的需要）
    * `conda-forge` 与 `pip` 配合更好
    * 库的版本更新

祝交易愉快！

------

## 准备就绪

你已经走到了这一步，说明你已成功安装了 freqtrade。

### 初始化配置

```bash
# Step 1 - Initialize user folder
freqtrade create-userdir --userdir user_data

# Step 2 - Create a new configuration file
freqtrade new-config --config user_data/config.json
```

你已准备好运行了，请阅读[机器人配置](configuration.md)，记得从 `dry_run: True` 开始，并验证一切正常运行。

要了解如何设置配置，请参阅[机器人配置](configuration.md)文档页面。

### 启动机器人

```bash
freqtrade trade --config user_data/config.json --strategy SampleStrategy
```

!!! Warning
    你应该通读其余文档，对你将使用的策略进行回测，并在使用真实资金交易之前先使用模拟交易（dry-run）。

-----

## 故障排除

### 常见问题："command not found"

如果你使用了 (1)`脚本` 或 (2)`手动` 安装方式，你需要在虚拟环境中运行机器人。如果遇到以下错误，请确保虚拟环境已激活。

```bash
# if:
bash: freqtrade: command not found

# then activate your virtual environment
source ./.venv/bin/activate
```

### MacOS 安装错误

较新版本的 MacOS 可能会出现安装失败，错误信息类似 `error: command 'g++' failed with exit status 1`。

此错误需要显式安装 SDK Headers，在此版本的 MacOS 中默认未安装。
对于 MacOS 10.14，可以使用以下命令完成安装：

```bash
open /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg
```

如果此文件不存在，那么你可能使用的是不同版本的 MacOS，可能需要在网上查找具体的解决方案。

### Windows 安装错误

```bash
error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": http://landinghub.visualstudio.com/visual-cpp-build-tools
```

不幸的是，许多需要编译的包没有提供预构建的 wheel。因此必须安装 C/C++ 编译器，并使其可供你的 python 环境使用。

你可以从 [Visual Studio 网站](https://visualstudio.microsoft.com/visual-cpp-build-tools/)下载 Visual C++ 构建工具，并以默认配置安装"使用 C++ 的桌面开发"。不幸的是，这是一个较大的下载/依赖项，所以你可能需要先考虑 WSL2 或 [docker compose](docker_quickstart.md)。

![Windows 安装](assets/windows_install.png)
