# 如何更新

要更新您的 freqtrade 安装，请根据您的安装方式使用以下方法之一。

!!! Note "跟踪变更"
    破坏性变更 / 行为变更将记录在每个版本附带发布的变更日志中。
    对于 develop 分支，请关注 PR 以避免被变更所影响。

## 为什么要更新？

保持机器人更新不仅可以确保您拥有最新的功能和改进，而且是保持机器人平稳运行的必要条件。
Freqtrade 高度依赖底层交易所 API，如果综合考虑所有交易所，这些 API 变更相当频繁。
为确保持续兼容性，请务必定期更新您的机器人。

## Docker

!!! Note "使用 `master` 镜像的旧版安装"
    我们正在将发布镜像从 master 切换到 stable——请调整您的 docker 文件，将 `freqtradeorg/freqtrade:master` 替换为 `freqtradeorg/freqtrade:stable`

``` bash
docker compose pull
docker compose up -d
```

## 通过安装脚本安装

``` bash
./setup.sh --update
```

!!! Note
    请确保在禁用虚拟环境的情况下运行此命令！

## 原生安装

请确保同时更新依赖项——否则可能会在您不知情的情况下出现问题。

``` bash
git pull
pip install -U -r requirements.txt
pip install -e .

# 确保 freqUI 为最新版本
freqtrade install-ui
```

## 更新问题

更新问题通常来自缺少依赖项（您没有按照上述说明操作）——或者来自安装失败的依赖项。
我们尽力确保重量级依赖项在主要平台上有预编译的 wheel 包可用，但有时这并不可能。

请参阅相应的安装部分（常见问题部分链接如下）。

[常见安装问题](installation.md#troubleshooting)
[常见安装问题 - Windows](installation.md#windows-installation-error)
