# R 语言与量化金融代码包

本目录将原项目源码、辅助文件和中文完整书稿中的代码块按 11 章集中整理。书中数据已单独放在同级 `data` 目录。

## 目录说明

- `Chapter_01` 至 `Chapter_11`：各章代码和示例数据。
- `code.R`：原书官方仓库中的该章代码集。部分章节混有 JSON、C++ 或控制台排版，应按小节阅读和运行。
- `book_code.md`：从中文完整书稿逐块提取的代码，保留原文顺序、输出和混合语言示例。
- `packages.txt`：从本章原仓库源文件检出的 R 扩展包名称。
- `manifest.csv`：文件来源、大小和 SHA-256 校验值。

## 使用建议

1. 将 R 工作目录设为本 `code` 目录，再打开 `Chapter_XX/code.R` 按小节运行；数据路径统一指向 `../data/Chapter_XX/`。
2. 依据各章 `packages.txt` 安装缺失的 R 包。
3. 第 8 章和第 9 章的部分示例依赖原项目未附带的商业日内数据，需自行配置文件路径。
4. `book_code.md` 中含有演示输出和非 R 语言片段，不建议整份直接执行。
5. `VALIDATION.md` 记录了原仓库 R 文件的静态语法检查结果。

## 整理结果

- 中文书稿代码块：309 个。
- 检出的 R 扩展包：18 个。
- 不纳入 `.Rhistory`、`.Rapp.history`、`.RData` 和章节草稿 Word 文件，避免将个人会话状态或非代码内容混入代码包。

## 全部扩展包

`DEoptim`, `devtools`, `dplyr`, `ggplot2`, `hflights`, `microbenchmark`, `PerformanceAnalytics`, `quantmod`, `quantstrat`, `Rcpp`, `RJSONIO`, `RMySQL`, `RODBC`, `RQuantLib`, `testthat`, `urca`, `XLConnect`, `xts`
