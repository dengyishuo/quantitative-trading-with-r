# 代码静态检查说明

本次整理保留了原项目代码内容，仅将第 2、3、8、9 章的数据路径统一调整为 `../data/Chapter_XX/`。使用 `Rscript` 的 `parse()` 对收录的 `.R` 文件做了不执行代码的语法扫描。

## 可完整解析的文件

- `Chapter_01/code.R`
- `Chapter_02/code.R`
- `Chapter_04/code.R`
- `Chapter_07/code.R`
- `Chapter_07/demoData.R`
- `Chapter_08/code.R`
- `Chapter_09/code.R`
- `Chapter_10/code.R`

## 需按小节使用的原文件

| 文件 | 首个静态解析中断位置 | 说明 |
|---|---:|---|
| `Chapter_03/code.R` | 第 62 行 | 文件内直接展示了 JSON 样例，该片段不是 R 语句。 |
| `Chapter_05/code.R` | 第 268 行 | 原文的 `plot()` 参数之间缺少逗号。 |
| `Chapter_06/code.R` | 第 421 行 | 原文留有单独的管道符号，属于排版残留。 |
| `Chapter_08/Pre.R` | 第 1 行 | 原文件使用旧编码，当前 UTF-8 环境无法直接解析。 |
| `Chapter_11/code.R` | 第 55 行 | 文件内直接展示 C++ 函数，该片段不是 R 语句。 |

上述情况来自原项目，不是数据路径调整过程引入的错误。各文件的源路径和整理后 SHA-256 校验值可在 `manifest.csv` 中查看。
