# __init__.py

## 概述

`freqtrade/configuration/__init__.py` 是 configuration 子包的入口文件，负责将子模块中的核心类和函数导出到包级别，方便外部通过 `from freqtrade.configuration import XXX` 的方式直接引用。

## 导出内容

| 导出名称 | 来源模块 | 说明 |
|---------|---------|------|
| `remove_exchange_credentials` | `config_secrets` | 移除交易所敏感凭证 |
| `sanitize_config` | `config_secrets` | 脱敏配置信息 |
| `setup_utils_configuration` | `config_setup` | 工具子命令的配置初始化 |
| `validate_config_consistency` | `config_validation` | 验证配置一致性 |
| `Configuration` | `configuration` | 核心配置类 |
| `running_in_docker` | `detect_environment` | 检测是否在 Docker 中运行 |
| `TimeRange` | `timerange` | 时间范围解析类 |

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.configuration.config_secrets` — 提供 `remove_exchange_credentials`, `sanitize_config`
- `freqtrade.configuration.config_setup` — 提供 `setup_utils_configuration`
- `freqtrade.configuration.config_validation` — 提供 `validate_config_consistency`
- `freqtrade.configuration.configuration` — 提供 `Configuration`
- `freqtrade.configuration.detect_environment` — 提供 `running_in_docker`
- `freqtrade.configuration.timerange` — 提供 `TimeRange`

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- 项目中大量模块通过 `from freqtrade.configuration import ...` 引用本包，包括：
  - `freqtrade.commands.*` — 各种 CLI 命令
  - `freqtrade.rpc.*` — RPC 服务器相关模块
  - `freqtrade.optimize.*` — 回测和优化模块
  - `freqtrade.data.*` — 数据处理模块
  - `freqtrade.freqtradebot` — 主交易机器人
  - `freqtrade.worker` — 工作进程
