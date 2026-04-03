# historic_precision.py

## 概述

`historic_precision.py` 提供了计算交易对历史 tick size（最小价格变动单位）随时间变化的功能。该模块分析 OHLCV K线数据中价格的有效小数位数，按月计算最大精度，并将其转换为对应的 tick size 值。这对于评估交易所价格精度变化、回测时确定合理的 tick size 设定非常有用。

## 架构图

```mermaid
flowchart LR
    A[OHLCV DataFrame] --> B[计算每个价格列的有效小数位数]
    B --> C[取 open/high/low/close 四列的最大值]
    C --> D[按月重采样取最大值]
    D --> E[转换为 tick size: 1/10^digits]
    E --> F[返回 Series]
```

## 核心类/函数

### get_tick_size_over_time(candles: DataFrame) -> Series
计算 K 线数据中价格精度随时间的变化。

**参数：**
- `candles: DataFrame` -- 包含 OHLCV 数据的 DataFrame，必须含有 `date`, `open`, `high`, `low`, `close` 列

**返回值：**
- `Series` -- 以月为周期的 tick size 序列，例如精度为 5 位小数时返回 0.00001

**处理逻辑：**
1. 对 `open`, `high`, `low`, `close` 四列，使用 `format_float_positional` 将浮点数转为精确字符串表示
2. 通过正则表达式 `r"\.(\d*[1-9])"` 提取小数部分的有效数字
3. 计算有效数字的长度（即有效小数位数）
4. 对四列取每行的最大值
5. 以 `date` 为索引，按月初（`"MS"`）重采样取最大值
6. 将位数转换为 tick size：`1 / 10^位数`

**应用场景：**
- 评估交易所是否在某个时间点改变了交易对的价格精度
- 为回测设置合适的价格精度参数

## 依赖关系

### 内部依赖（本项目模块）
无

### 外部依赖（第三方库）
- `numpy.format_float_positional` -- 浮点数精确格式化输出
- `pandas.DataFrame` -- 数据结构
- `pandas.Series` -- 返回类型

### 被依赖（谁引用了本文件）
- 通过 `freqtrade.data.btanalysis.__init__` 导出 `get_tick_size_over_time`
- `tests/data/test_historic_precision.py` -- 测试文件
