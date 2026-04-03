# __init__.py

## 概述

`freqtrade/data/history/datahandlers/__init__.py` 是数据处理器子包的初始化文件，仅从 `idatahandler` 模块导出两个关键接口。

## 导出内容

- `IDataHandler` -- 数据处理器抽象基类
- `get_datahandler` -- 数据处理器实例的工厂函数

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.data.history.datahandlers.idatahandler` -- 抽象接口和工厂函数

### 被依赖（谁引用了本文件）
- `freqtrade.data.history.__init__` -- history 包导出 `get_datahandler`
- `freqtrade.data.history.history_utils` -- 导入 IDataHandler 和 get_datahandler
