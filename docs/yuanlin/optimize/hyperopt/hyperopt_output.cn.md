# hyperopt_output.py

## 概述

Hyperopt 结果输出格式化模块。使用 Rich 库创建美观的终端表格，实时展示超参数优化过程中每个 epoch 的结果。支持流式输出模式，根据终端窗口大小自动调整显示的行数。

表格包含以下列：Best（最佳标记）、Epoch（轮次）、Trades（交易次数）、Win/Draw/Loss/Win%（胜负比）、Avg profit（平均利润）、Profit（总利润）、Avg duration（平均持仓时间）、Objective（目标函数值）、Max Drawdown（最大回撤）。

## 架构图

```mermaid
classDiagram
    class HyperoptOutput {
        -list _results
        -bool _streaming
        -Table table
        +__init__(streaming: bool)
        +__call__(*args, **kwds) Align
        +print(console, print_colorized)
        +add_data(config, results, total_epochs, highlight_best)
        -__init_table()
    }

    HyperoptOutput --> "rich.Table" : 使用
    HyperoptOutput --> "rich.Console" : 打印输出
    HyperoptOutput --> "rich.Text" : 带样式文本
    HyperoptOutput --> "rich.Align" : 居中对齐

    note for HyperoptOutput "作为 callable 传给进度条的 cust_callables\n每次调用返回居中对齐的表格"
```

## 核心类/函数

### HyperoptOutput

#### `__init__(self, streaming=False) -> None`
- **参数**: `streaming` — 是否启用流式模式（根据终端大小限制行数）
- **职责**: 初始化结果列表和表格

#### `__call__(self, *args, **kwds) -> Align`
- **返回**: 居中对齐的 Rich 表格
- **用途**: 作为 callable 被进度条系统调用，实现实时刷新显示

#### `__init_table(self) -> None`
- 初始化/重置 Rich `Table` 对象
- 设置 9 列标题：Best, Epoch, Trades, Win/Draw/Loss/Win%, Avg profit, Profit, Avg duration, Objective, Max Drawdown (Acct)

#### `add_data(self, config: Config, results: list, total_epochs: int, highlight_best: bool) -> None`
- **参数**:
  - `config` — 全局配置（用于获取 `stake_currency`）
  - `results` — 新增的结果列表
  - `total_epochs` — 总 epoch 数
  - `highlight_best` — 是否高亮最佳结果
- **职责**: 格式化并添加数据行到表格
- **流式模式逻辑**:
  - 获取终端尺寸（`get_terminal_size()`）
  - 如果终端宽度 < 148 列，显示行数减半
  - 每次调用时重新创建表格并填充最后 N 行数据
- **行样式**:
  - 最佳结果（`is_best=True` 且 `highlight_best`）：`bold gold1`
  - 初始随机点（`is_initial_point`）：`italic`
  - 利润为正：绿色；利润为负：红色
  - 目标值为 `MAX_LOSS`（100000）时显示 "N/A"

#### `print(self, console: Console | None = None, *, print_colorized=True)`
- **参数**: `console` — 可选的 Rich Console 实例；`print_colorized` — 是否彩色输出
- **职责**: 将表格打印到控制台
- 测试环境（`pytest` 在 `sys.modules` 中）时使用 200 列宽度

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.constants` — `Config` 类型
- `freqtrade.optimize.optimize_reports` — `generate_wins_draws_losses` 生成胜负统计字符串
- `freqtrade.util` — `fmt_coin` 货币格式化

### 外部依赖（第三方库）
- `rich` — `Console`, `Table`, `Text`, `Align` 终端美化输出库
- `os` — `get_terminal_size` 获取终端尺寸

### 被依赖（谁引用了本文件）
- `freqtrade.optimize.hyperopt.hyperopt` — `Hyperopt` 类创建 `HyperoptOutput` 实例用于实时展示结果
- `freqtrade.commands.hyperopt_commands` — 命令行工具使用 `HyperoptOutput` 显示历史结果
