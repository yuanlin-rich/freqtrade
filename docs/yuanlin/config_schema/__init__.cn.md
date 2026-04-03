# config_schema/__init__.py

## 概述
`freqtrade/config_schema/__init__.py` 是 config_schema 子包的入口文件，导出 `CONF_SCHEMA` 配置验证 schema。该包负责定义 freqtrade 配置文件的完整 JSON Schema，用于配置文件的格式校验。

## 导出列表
- `CONF_SCHEMA` — 来自 `freqtrade.config_schema.config_schema`，完整的配置 JSON Schema 字典

## 依赖关系

### 内部依赖（本项目模块）
- `freqtrade.config_schema.config_schema` — CONF_SCHEMA 的实际定义

### 外部依赖（第三方库）
- 无

### 被依赖（谁引用了本文件）
- `freqtrade.configuration.config_validation` — 使用 CONF_SCHEMA 验证用户配置
- `build_helpers/extract_config_json_schema.py` — 提取 schema 用于文档生成
