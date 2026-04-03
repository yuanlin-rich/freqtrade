# cryptocom.py

## 概述

Crypto.com 交易所子类实现。适配内容极简，仅配置了 K 线数量限制。

## 架构图

```mermaid
classDiagram
    class Exchange {
        <<基类>>
    }

    class Cryptocom {
        -FtHas _ft_has
    }

    Exchange <|-- Cryptocom
```

## 核心类/函数

### Cryptocom 类

#### _ft_has 配置
- `ohlcv_candle_limit: 300` -- K 线数量限制 300

无方法重写。

## 依赖关系

### 内部依赖
- `freqtrade.exchange.Exchange` -- 基类
- `freqtrade.exchange.exchange_types.FtHas` -- 特性配置类型

### 被依赖
- `freqtrade.exchange.__init__` -- 导出 Cryptocom 类
