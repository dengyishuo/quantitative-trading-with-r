# R 语言与量化金融数据包

本目录集中存放原项目实际附带的书中数据，并按章组织。

## 已收录数据

- `Chapter_02/prices.csv`：第 2 章价格数据。
- `Chapter_03/aapl.csv`：第 3 章 AAPL 股价数据。
- `Chapter_03/sample_json_file.json`：第 3 章 JSON 读取示例。
- `Chapter_03/strategy.xlsx`：第 3 章 Excel 策略表示例。

## 未随原项目提供的数据

- 第 8 章使用 `SPY_2013_04_15_X_Q.asc` 和 `SPY_2013_04_15_X_T.asc`。
- 第 9 章使用 `SPY_20130415_T` 和 `SPY_20130410_QT` 目录中的期权数据。

这些高频和期权数据来自 Tick Data，原项目没有附带文件。对应章节目录中已放置说明文件。

## 路径约定

从 `整合书稿/code` 作为 R 工作目录时，整理后的 `code.R` 使用 `../data/Chapter_XX/` 访问本目录。

`manifest.csv` 记录每个已收录数据文件的原路径、大小和 SHA-256 校验值。
