# backteststate.py

## 概述

`freqtrade/enums/backteststate.py` 定义了 `BacktestState` 枚举类，表示回测引擎在执行过程中的不同阶段。用于向 UI 或日志系统报告回测进度。

## 架构图

```mermaid
stateDiagram-v2
    [*] --> STARTUP
    STARTUP --> DATALOAD: 数据加载阶段
    DATALOAD --> ANALYZE: 分析阶段
    ANALYZE --> CONVERT: 转换阶段
    CONVERT --> BACKTEST: 回测执行阶段
    BACKTEST --> [*]
```

## 核心类/函数

### `class BacktestState(Enum)`

回测状态枚举，继承自 `Enum`。

| 枚举值 | 整数值 | 说明 |
|--------|-------|------|
| `STARTUP` | 1 | 启动阶段 |
| `DATALOAD` | 2 | 数据加载阶段 |
| `ANALYZE` | 3 | 数据分析阶段 |
| `CONVERT` | 4 | 数据转换阶段 |
| `BACKTEST` | 5 | 回测执行阶段 |

#### `__str__(self) -> str`

返回枚举名称的小写形式，如 `"startup"`、`"dataload"` 等。

## 依赖关系

### 内部依赖（本项目模块）
- 无

### 外部依赖（第三方库）
- `enum.Enum` — Python 标准库枚举基类

### 被依赖（谁引用了本文件）
- `freqtrade.enums.__init__` — 导出到包级别
- 通过包级别引用被 `freqtrade.optimize.backtesting` 和 `freqtrade.rpc` 等模块使用
