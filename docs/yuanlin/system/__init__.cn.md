# system/__init__.py

## 概述
`freqtrade/system/__init__.py` 是 system 子包的入口文件，统一导出该包中所有系统配置和性能调优相关的函数。该模块汇集了 asyncio 事件循环配置、垃圾回收优化、多进程启动方式设置和版本信息打印等系统级功能。

## 导出列表
- `asyncio_setup` — 来自 `freqtrade.system.asyncio_config`，配置 asyncio 事件循环
- `gc_set_threshold` — 来自 `freqtrade.system.gc_setup`，优化 GC 阈值
- `set_mp_start_method` — 来自 `freqtrade.system.set_mp_start_method`，设置多进程启动方式
- `print_version_info` — 来自 `freqtrade.system.version_info`，打印版本信息

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.system.asyncio_config`
- `freqtrade.system.gc_setup`
- `freqtrade.system.set_mp_start_method`
- `freqtrade.system.version_info`

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- `freqtrade.main` — 导入所有四个系统配置函数
