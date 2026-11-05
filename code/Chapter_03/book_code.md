# 第 3 章 与数据打交道 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 数据导入

### 代码块 001

```r
# 在windows电脑中
aapl <- read.table("clipboard")
# 在Mac/Linux中
aapl <- read.table(pipe("pbpaste"))
```

### 代码块 002

```text
head(aapl)
```

### 代码块 003

```text
class(aapl)
```

### 代码块 004

```r
aapl <- aapl[rev(rownames(aapl)), , drop = FALSE]
```

### 代码块 005

```r
prices <- aapl$V1
plot(prices, main = "AAPL plot", type = 'l')
```

### 代码块 006

```r
# 加载.csv 文件
aapl_2 <- read.csv(file = "Chapter_03/aapl.csv", header = TRUE, stringsAsFactors = FALSE)

# 倒转行元素
aapl_2 <- aapl_2[rev(rownames(aapl_2)), ]
```

### 代码块 007

```r
aapl_close <- aapl_2[, "Close"]
```

### 代码块 008

```text
summary(aapl_close)
```

## R包安装

### 代码块 009

```r
install.packages(pkgs, lib, repos = getOption("repos"),
contriburl = contrib.url(repos, type),
method, available = NULL, destdir = NULL,
dependencies = NA, type = getOption("pkgType"),
configure.args = getOption("configure.args"),
configure.vars = getOption("configure.vars"),
clean = FALSE, Ncpus = getOption("Ncpus", 1L),
verbose = getOption("verbose"),
libs_only = FALSE, INSTALL_opts, quiet = FALSE,
keep_outputs = FALSE, ...)
```

## 数据储藏和传输

### 代码块 010

```text
{
"CVX":
{
"Currency": "USD",
"Sector": "Basic Materials",
"Industry": "Major Integrated Oil & Gas"
},
"GOOG":
{
"Currency": "USD",
"Sector": "Technology",
"Industry": "Internet Information Providers"
}
}
```

### 代码块 011

```r
# 安装和加载所需包
install.packages("RJSONIO")
library(RJSONIO)
# 读取数据文件 
out <- fromJSON(content = "path/sample_json_file.json" )
# 检视结果对象的结构str(out)
## List of 2
## $ CVX : Named chr [1:3] "USD" "Basic Materials...
## ..- attr(*, "names")= chr [1:3] "Currency"...
## $ GOOG: Named chr [1:3] "USD" "Technology"...
## ..- attr(*, "names")= chr [1:3] "Currency"...
```

### 代码块 012

```text
write.csv(aapl_2, file = "path/aapl_2.csv")
```

### 代码块 013

```text
save(aapl_2, file = "path/aapl_2.rdata")
```

### 代码块 014

```r
aapl_old <- aapl_2
rm(aapl_2)
load(file = "path/aapl_2.rdata")
```

### 代码块 015

```text
identical(aapl_old, aapl_2)
## [1] TRUE
```

## 从表单提取数据

### 代码块 016

```r
library(XLConnect)

# 创建一个工作簿对象
book <- loadWorkbook("Chapter_03/strategy.xlsx")

# 将其转换成数据框格式
signals <- readWorksheet(book, sheet = "signals", header = TRUE)

signals
##     time signal1 signal2
## 1 08:30:00  0.43   -0.20
## 2 08:31:00  0.54    0.33
## 3 08:32:00  0.32   -0.21

strength <- readWorksheet(book, sheet = "strength", header = TRUE)

strength
## intensity score
## 1 2 7.5
## 2 3 8.4
## 3 6 5.4
```

### 代码块 017

```r
# 创建一个新的工作表
book <- loadWorkbook("demo_sheet.xlsx", create = TRUE)

# 创建一个名为stock1的工作表
createSheet(book, name = "stock1")

# 创建一个名为stock2的工作表C 
createSheet(book, name = "stock2")

# 将数据载入工作簿
df <- data.frame(a = c(1, 2, 3), b = c(4, 5, 6))
writeWorksheet (book, data=df, sheet="stock1", header = TRUE)

# 保存该工作簿
saveWorkbook(book, file = "Chapter_03/demo_sheet.xlsx")
```

## 数据库登录

### 代码块 018

```r
# 加载RODBC 包
require(RODBC)

# 创建一个MySQL连接
con <- odbcConnect("rfortraders")

# 选择数据库和数据表名称
database_name <- "OptionsData"
table_name <- "ATMVolatilities"
symbol <- "SPY"

sql_command <- paste0("SELECT Symbol, Date, Maturity,
  Delta, CallPut, ImpliedVolatility
  FROM ", database_name, ".", table_name,
  " WHERE Maturity = 91
  AND Symbol IN ('", symbol, "');")

iv <- sqlQuery(con, sql_command)

# 断开与数据库的连接
odbcClose(con)
```

### 代码块 019

```text
head(iv)
## Symbol Date Maturity Delta CallPut ImpliedVolatility
## SPY 6/9/2014       91    55       C          0.115925
## SPY 6/9/2014       91    60       C          0.119577
## SPY 6/9/2014       91    65       C          0.123468
## SPY 6/9/2014       91    70       C          0.127629
## SPY 6/9/2014       91    75       C          0.132094
## SPY 6/9/2014       91    80       C          0.136776
```

### 代码块 020

```r
# 载入必要的包 
require(RMySQL)

# 建立一个连接
con <- dbConnect(MySQL(), user="your_login",
password="your_password",
dbname="OptionsData",
host="location_of_database")

# 列出其中表和字段
dbListTables(con)

# 定义查询命令提取数据
sql_command <- paste0("SELECT Symbol, Date, Maturity,
Delta, CallPut, ImpliedVolatility FROM ",
database_name, ".", table_name,
" WHERE Maturity = 91
AND Symbol IN ('", symbol, "');")

result <- dbGetQuery(con, sql_command)

# 关闭连接
dbDisconnect(con)
```

### 代码块 021

```r
results <- dbSendQuery(con, sql_command)
partial_results <- fetch(results, n = 100)
```

## dplyr包

### 代码块 022

```r
# 获取CRAN版本
install.packages("dplyr")
require(dplyr)
# 或者先加载devtools包
install.packages("devtools")
require(devtools)
# 再获取 github版本
devtools::install_github("hadley/dplyr")
require(dplyr)
```

### 代码块 023

```r
# 加载dplyr相关的航线数据集hflights
library(hflights)
# 检查行数和列数
dim(hflights)
## [1] 227496 21
```

### 代码块 024

```r
library(dplyr)
library(knitr)
library(hflights)

hflights %>%
  head(1) %>%
  t() %>%
  kable()
```

### 代码块 025

```r
# 首先，将数据强制转换成data.table
flights_dt <- dtplyr::tbl_dt(hflights)
# 该对象的类型是什么？
class(flights_dt)
## [1] "tbl_dt" "tbl" "data.table" "data.frame"
```

### 代码块 026

```r
# 按照航空公司分组
carrier_group <- group_by(flights_dt, UniqueCarrier)

# 现在计算汇总统计量
summarise(carrier_group, avg_delay = mean(ArrDelay, na.rm = TRUE))
```

## 使用xts包

### 代码块 027

```r
# 加载xts包
library(xts)

# 加载xts自带的一个小数据集
# 我们也可以使用我们自己的.csv文件
data(sample_matrix)

# 检查数据
head(sample_matrix)
##            Open     High      Low    Close
## 2007-01-02 50.03978 50.11778 49.95041 50.11778
## 2007-01-03 50.23050 50.42188 50.23050 50.39767
## 2007-01-04 50.42096 50.42096 50.26414 50.33236
## 2007-01-05 50.37347 50.37347 50.22103 50.33459
## 2007-01-06 50.24433 50.24433 50.11121 50.18112
## 2007-01-07 50.13211 50.21561 49.99185 49.99185

# 该数据对象的类型是？ 
class(sample_matrix)
## [1] "matrix"

# 使用str() 命令得到关于该数据对象的更多细节
str(sample_matrix)
## num [1:180, 1:4] 50 50.2 50.4 50.4 50.2 ...
## - attr(*, "dimnames")=List of 2
## ..$ : chr [1:180] "2007-01-02" "2007-01-03"
## "2007-01-04" "2007-01-05" ...
## ..$ : chr [1:4] "Open" "High" "Low" "Close"
```

### 代码块 028

```r
xts_matrix <- as.xts(sample_matrix, descr ='my new xts object')
```

### 代码块 029

```text
str(xts_matrix)
## An 'xts' object on 2007-01-02/2007-06-30 containing: 
## Data: num [1:180, 1:4] 50 50.2 50.4 50.4 50.2 ...
## - attr(*, "dimnames")=List of 2
## ..$ : NULL
## ..$ : chr [1:4] "Open" "High" "Low" "Close"
## Indexed by objects of class: [POSIXct,POSIXt] TZ: 
## xts Attributes:
## List of 3
## $ tclass: chr [1:2] "POSIXct" "POSIXt"
## $tzone:chr""
## $ descr : chr "my new xts object"
```

### 代码块 030

```r
# 简单时间序列图 
plot(xts_matrix[,1], main = "Our first xts plot",
cex.main = 0.8)
# 或者我们尝试一些更花哨的图
plot(xts_matrix, main = "Candle plot on xts object",
cex.main = 0.8, type = "candles")
```

### 代码块 031

```r
plot(xts_matrix["2007-01-01::2007-02-12"],
  main = "An xts candle plot with subsetting",
  cex.main = 0.8, type = "candles")
```

### 代码块 032

```r
range <- "2007-03-15::2007-06-15"
plot(xts_matrix(range))
```

### 代码块 033

```r
start_date <- "2007-05-05"
end_date <- "2007-12-31"

plot(xts_matrix[paste(start_date, "::",
                      end_date, sep = "")])

# 默认使用空格连接符
paste("Hello", "World", "in R")
## [1] "Hello World in R"

paste("Hello", "Again", sep = "**")
## [1] "Hello**Again"
```

### 代码块 034

```r
paste(c(1,2,3,4,5), collapse = "oooo")
## [1] "1oooo2oooo3oooo4oooo5"
```

### 代码块 035

```r
# 编造一个包含10个股票以及以微秒为单位的时间索引信息的价格向量
price_vector <- c(101.02, 101.03, 101.03, 101.04, 101.05,
                  101.03, 101.02, 101.01, 101.00, 100.99)

dates <- c("03/12/2013 08:00:00.532123",
           "03/12/2013 08:00:01.982333",
           "03/12/2013 08:00:01.650321",
           "03/12/2013 08:00:02.402321",
           "03/12/2013 08:00:02.540432",
           "03/12/2013 08:00:03.004554",
           "03/12/2013 08:00:03.900213",
           "03/12/2013 08:00:04.050323",
           "03/12/2013 08:00:04.430345",
           "03/12/2013 08:00:05.700123")

# 允许R控制台显示到微秒级别options(digits.secs = 6)

# 创建经过正确格式化的时间索引
time_index <- strptime(dates, format = "%d/%m/%Y %H:%M:%OS")

# 将时间索引值传递给数据对象
xts_price_vector <- xts(price_vector, time_index)
```

### 代码块 036

```r
# 绘制编造股票价格数据图
plot(xts_price_vector, main = "Fictitious price series",
  cex.main = 0.8)
  
# 添加一个表示均值的横线
abline(h = mean(xts_price_vector), lwd = 2)

# 添加标识一个特定时间戳的竖线
my_time <- as.POSIXct("03/12/2013 08:00:03.004554",
  format = "%d/%m/%Y %H:%M:%OS")

abline(v = my_time, lwd = 2, lty = 2)
```

### 代码块 037

```r
es_price <- c(1700.00, 1700.25, 1700.50, 1700.00, 1700.75,
  1701.25, 1701.25, 1701.25, 1700.75, 1700.50)

es_time <- c("09/12/2013 08:00:00.532123",
  "09/12/2013 08:00:01.982333",
  "09/12/2013 08:00:05.650321",
  "09/12/2013 08:10:02.402321",
  "09/12/2013 08:12:02.540432",
  "09/12/2013 08:12:03.004554",
  "09/12/2013 08:14:03.900213",
  "09/12/2013 08:15:07.090323",
  "09/12/2013 08:16:04.430345",
  "09/12/2013 08:18:05.700123")

# 创建一个xts时间序列对象
xts_es <- xts(es_price, as.POSIXct(es_time,
  format = "%d/%m/%Y %H:%M:%OS"))

names(xts_es) <- c("price")
```

### 代码块 038

```r
time_diff <- difftime(index(xts_es)[2], index(xts_es)[1],
  units = "secs")

time_diff
## Time difference of 1.45021 secs
```

### 代码块 039

```r
diffs <- c()

for(i in 2:length(index(xts_es))) {
  diffs[i] <- difftime(index(xts_es)[i], index(xts_es)[i - 1],
    units = "secs")
}
```

### 代码块 040

```r
diffs <- index(xts_es)[-1] - index(xts_es)[-length(index(xts_es))]

diffs
## Time differences in secs
## [1] 1.4502099 3.6679881 596.7520001
## [4] 120.1381109 0.4641221 120.8956590
## [7] 63.1901100 57.3400221 121.2697780
## attr(,"tzone")

class(diffs)
## [1] "difftime"
```

### 代码块 041

```r
es_times <- index(xts_es)
diffs <- es_times[-1] - es_times[-length(es_times)]

diffs
## Time differences in secs
## [1] 1.4502099 3.6679881 596.7520001
## [4] 120.1381109 0.4641221 120.8956590
## [7] 63.1901100 57.3400221 121.2697780
## attr(,"tzone")
```

### 代码块 042

```r
par(mfrow = c(2, 1))
diffs <- as.numeric(diffs)
plot(diffs, main = "Time difference in seconds for ES trades",
     xlab = "", ylab = "Time differences",
     cex.lab = 0.8,
     cex.main = 0.8)
grid()

hist(diffs, main = "Time difference in seconds for ES trades",
     xlab = "Time difference (secs)", ylab = "Observations",
     breaks = 20,
     cex.lab = 0.8,
     cex.main = 0.8)
grid()
```

## 使用quantmod包

### 代码块 043

```r
# 本地安装之后加载quantmod包
library(quantmod)
AAPL <- getSymbols("AAPL", auto.assign=FALSE)
head(AAPL)
```

## quantmod包与图表

### 代码块 044

```text
# 在原图基础上添加一些技术指标
chartSeries(AAPL, subset='2010::2010-04',
theme = chartTheme('white'),
TA = "addVo(); addBBands()")
```

### 代码块 045

```text
reChart(subset='2009-01-01::2009-03-03')
```

### 代码块 046

```text
chartSeries(AAPL, subset='2011::2012', theme = chartTheme('white'), TA = "addBBands(); addDEMA()")
```

### 代码块 047

```text
addVo()
addDPO()
```

### 代码块 048

```r
# 创建不包含任何技术指标的基础图
chartSeries(AAPL, theme = chartTheme('white'), TA = NULL)

# 自定义函数
my_indicator <- function(x) {
  return(x + 90)
}

add_my_indicator <- newTA(FUN = my_indicator, preFUN=Cl,
  legend.name = "My Fancy Indicator", on = 1)
add_my_indicator()
```

## 用ggplot2包绘图

### 代码块 049

```r
# 创建一个包含价格和交易量的矩阵
df <- AAPL[, c("AAPL.Adjusted", "AAPL.Volume")]
names(df) <- c("price", "volume")

# 构造
df$return <- diff(log(df[, 1]))
df <- df[-1, ]
```

### 代码块 050

```r
df$cuts <- cut(abs(df$return),
  breaks = c(0, 0.02, 0.04, 0.25),
  include.lowest = TRUE)
  
# 构造另外一列用来保存均值
df$means <- NA
for(i in 1:3) {
  group <- which(df$cuts == i)
  if(length(group) > 0) {
    df$means[group] <- mean(df$volume[group])
  }
}  
```

### 代码块 051

```r
# 加载ggplot2包
library(ggplot2)
ggplot(df) +
geom_histogram(aes(x=volume)) +
facet_grid(cuts ~ .) +
geom_vline(aes(xintercept=means), linetype="dashed", size=1)
```
