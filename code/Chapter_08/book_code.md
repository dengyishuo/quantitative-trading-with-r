# 第 8 章 高频数据分析 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 高频报价数据

### 代码块 001

```r
spy_file <- "path/Stocks/QUOTES/SPY_2013_04_15_X_Q.asc"
spy_quotes <- read.csv(file = syp_file,header = FALSE,stringsAsFactors = FALSE)
```

### 代码块 002

```text
head(spy_quotes, 3)
##           V1               V2  V3      V4   V5 V6 V7
## 1 04/15/2013 04:00:00.065  T 156.60   0.00  1  0  R
## 2 04/15/2013 04:00:00.626  P   0.00   0.00  0  0  R
## 3 04/15/2013 04:00:00.633  P 158.25 158.90  1 47  R

## V8 V9 V10 V11 V12 V13 V14 V15 V16 V17 V18 V19 V20 V21
## NA   1   T   T   1   2   A   C    NA   NA   NA   NA
## NA   2   P   P   0   2   A   C    NA   NA   NA   NA
## NA   3   P   P   1   2   A   C    NA   NA   NA   NA
```

### 代码块 003

```r
names(spy_quotes) <- c("date", "time", "exchange", "bid_price",
  "ask_price", "bid_size", "ask_size", "quote_condition",
  "mode", "market_maker_id", "sequence_number", "bid_exchange",
  "ask_exchange",  "national_bbo_indicator",
  "nasdaq_bbo_indicator", "quote_cancel_correction",
  "quote_source", "short_sale_restriction_indicator",
  "limit_up_down_bbo_indicator_cqs",
  "limit_up_down_bbo_indicator_utp",
  "finra_adf_mpid_indicator")
```

### 代码块 004

```r
spy_quotes_arca <- spy_quotes[spy_quotes$exchange %in% c("P"),
  c("date", "time", "bid_price",
  "ask_price", "bid_size", "ask_size")]
```

### 代码块 005

```r
require(xts)

# 设定全局变量参数以达到毫秒级精度
options(digits.secs = 3)

time_index <- as.POSIXct(paste(spy_quotes_arca$date,
  spy_quotes_arca$time), format = "%m/%d/%Y %H:%M:%OS")
spy <- xts(spy_quotes_arca[, -c(1, 2)], time_index)
rm(time_index)
```

### 代码块 006

```r
plot(spy$bid_price, type = 'l',
  main = "SPY bid price",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 007

```r
spy_filtered <- spy[spy$bid_price > 0, ]
```

### 代码块 008

```r
rows_removed <- nrow(spy) - nrow(spy_filtered)

rows_removed
## [1] 2
```

### 代码块 009

```r
plot(spy_filtered$bid_price, type = 'l',
  main = "SPY filtered bid price",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 010

```text
summary(as.numeric(spy_filtered$ask_price))
  ## Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  ## 154.3   156.1   156.7   156.8   157.7   158.9
```

### 代码块 011

```text
summary(as.numeric(spy_filtered$bid_size))
## Min. 1st Qu.  Median  Mean 3rd Qu.  Max.
## 1.00 22.00 52.00 94.93 100.00 1565.00

summary(as.numeric(spy_filtered$ask_size))
## Min. 1st Qu. Median Mean 3rd Qu. Max.
## 1.0  24.0  59.0  110.8 118.0 1412.0
```

### 代码块 012

```r
# 生成一个表示当买入价不低于卖出价的布尔值向量
crossed_prices <- spy_filtered$bid_price >=
  spy_filtered$ask_price

any(crossed_prices)
## [1] FALSE
```

## 内部报价更新时间

### 代码块 013

```r
# 提取时间索引
quote_times <- index(spy_filtered)

# 计算连续时间差值
time_differences <- as.numeric(diff(quote_times))

summary(time_differences)
## Min.   1st Qu. Median   Mean   3rd Qu.    Max.
## 0.0000 0.0000  0.0010  0.0645  0.0100  1010.0000
```

### 代码块 014

```r
# 定位可能异常的时间点
long_times <- which(time_differences > 1000)
long_times
## [1] 884312

# 展示异常时间点附近的数据
spy_abnormal <- spy_filtered[(long_times - 2):(long_times + 2), ]
##                            bid_price ask_price bid_size ask_size
## 2013-04-15 16:13:00.987    155.00    155.02      289      270
## 2013-04-15 16:13:01.256    155.00    155.02      290      270
## 2013-04-15 16:13:01.282    155.00    155.02      295      270
## 2013-04-15 16:29:50.869    154.76    154.79        3        3
## 2013-04-15 16:29:50.887    154.76    154.77        3       10
```

## 识别流动性机制

### 代码块 015

```r
# 计算买卖差价
bid_ask_spread <- spy_filtered$ask_price -
  spy_filtered$bid_price

# 过滤掉异常值
outliers <- which(bid_ask_spread > 0.25)
if(length(outliers) > 0) {
  bid_ask_spread <- bid_ask_spread[-outliers, ]
}

# 绘制差价图
plot(bid_ask_spread, type = "l",
  main = "Bid ask spread",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 016

```r
# 将SPY数据划分为三个时段
early_morning <- "2013-04-15 04:00:00::2013-04-15 08:29:00"
regular_trading <- "2013-04-15 08:30:00::2013-04-15 16:15:00"
after_hours <- "2013-04-15 16:15:01::2013-04-15 20:00:00"

# 为每一个时段绘制一个买卖差价直方图
par(mfrow = c(3, 1))

# 清晨时段
data <- bid_ask_spread[early_morning]
hist(data, main = early_morning, breaks = 1000,
  xlim = c(0, 0.1))
abline(v = mean(data), lwd = 2, lty = 2)

# 下午时段
data <- bid_ask_spread[regular_trading]
hist(data, main = regular_trading, breaks = 1000,
  xlim = c(0, 0.1))
abline(v = mean(data), lwd = 2, lty = 2)

# 夜间时段
data <- bid_ask_spread[after_hours]
hist(data, main = after_hours, breaks = 1000,
  xlim = c(0, 0.1))
abline(v = mean(data), lwd = 2, lty = 2)
```

### 代码块 017

```r
spy_day <- spy_filtered[regular_trading]
```

## 市场微观价格

### 代码块 018

```r
spy_micro_price <- (spy_day$bid_price * spy_day$ask_size +
  spy_day$ask_price * spy_day$bid_size) /
  (spy_day$bid_size + spy_day$ask_size)
```

### 代码块 019

```r
par(mfrow = c(1, 1))
range <- 10000:10100
title <- "Micro-price between bid-ask prices"

plot(spy_day$ask_price[range],
  ylim = c(min(spy_day$bid_price[range]),
  max(spy_day$ask_price[range])),
  main = title,
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
lines(spy_day$bid_price[range])
lines(spy_micro_price[range], lty = 2)
```

## 分布和自相关性

### 代码块 020

```r
spy_returns <- diff(log(spy_micro_price))

par(mfrow = c(2, 1))
plot(spy_returns,
  main = "Time series plot of micro-price returns",
  cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
hist(spy_returns, breaks = 1000,
  main = "Micro-price distribution",
  cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
```

### 代码块 021

```r
par(mfrow = c(1, 1))
mean_spy <- mean(as.numeric(spy_returns), na.rm = TRUE)
sd_spy <- sd(as.numeric(spy_returns), na.rm = TRUE)

hist(spy_returns, breaks = 10000, prob = TRUE,
  xlim = c(-0.00003, 0.00003),
  main = "Micro-price distribution vs. Normal",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)

curve(dnorm(x, mean_spy, sd_spy), add = TRUE,
  yaxt = "n", lwd = 3, lty = 3)
```

### 代码块 022

```r
spy_acf <- acf(as.numeric(spy_returns),
  na.action = na.pass,
  main = "Autocorrelation",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 023

```r
# 为我们的逐笔交易数据获取SPY的交易价格
spy_trades_file <- "path/SPY_2013_04_15_X_T.asc"
spy_trades <- read.csv(file = spy_trades_file,
  header = FALSE, stringsAsFactors = FALSE)

names(spy_trades) <- c("date", "time", "price",
  "volume", "exchange", "sales_condition",
  "correction_indicator", "sequence_number",
  "trade_stop_indicator", "source_of_trade", "trf",
  "exclude_record_flag", "filtered_price")

# 只提取出ARCA交易部分数据
spy_trades_arca <- spy_trades[spy_trades$exchange %in% c("P"),
  c("date", "time", "price", "volume", "correction_indicator",
  "filtered_price")]

# 检查是否存在过滤价格
any(!is.na(spy_trades_arca$filtered_price))
## [1] FALSE

# 检查是否存在特别更正指示数据
unique(spy_trades_arca$correction_indicator)
## [1] 0

# 从数据框中丢弃最后两列数据
spy_trades_arca <- spy_trades_arca[, 1:4]

# 为下面的分析将数据转换为xts对象
time_index <- as.POSIXct(paste(spy_trades_arca$date,
  spy_trades_arca$time), format = "%m/%d/%Y %H:%M:%OS")

spy_t <- xts(spy_trades_arca[, -c(1, 2)], time_index)
rm(time_index)

# 前6个数据
head(spy_t)
##                          price volume
## 2013-04-15 04:00:00.697 158.25    100
## 2013-04-15 04:00:00.697 158.24    200
## 2013-04-15 04:00:00.697 158.15    150
## 2013-04-15 04:01:42.190 158.06    200
## 2013-04-15 04:07:16.545 157.94    100
## 2013-04-15 04:12:45.265 157.92  10000

# 取常规交易时间段的数据子集
regular_trading <- "2013-04-15 08:30:00::2013-04-15 16:15:00"
spy_t_day <- spy_t[regular_trading]

# 检视交易数据量
dim(spy_t_day)
## [1] 93197     2

# 检查已占用内存大小
object.size(spy_t_day)
## [1] 2239080 bytes
```

### 代码块 024

```r
# 计算收益率
spy_t_day_returns <- diff(log(spy_t_day$price))[-1]

# 绘制分布图和自相关函数图
par(mfrow = c(2, 1))
plot(spy_t_day_returns, main = "SPY returns on trades",
  cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
acf(as.numeric(spy_t_day_returns), main = "SPY trades acf",
  cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
```

### 代码块 025

```r
# 交易收益率的分布图
par(mfrow = c(1, 1))
hist(spy_t_day_returns, breaks = 1000, prob = TRUE,
  xlim = c(-0.0001, 0.0001),
  main = "Distribution of SPY trade returns",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)

curve(dnorm(x, mean(spy_t_day_returns),
  sd(spy_t_day_returns)),
  add = TRUE,
  yaxt = "n",
  lwd = 3,
  lty = 3)
```

### 代码块 026

```r
# 使用rle()函数找到价格序列
prices_rle <- rle(as.numeric(spy_t_day$price))

# 下面是我们想要保留的行索引
end_indexes <- cumsum(prices_rle$lengths)

# 下面是我们汇总交易量的时候想要的起始时间索引
start_indexes <- end_indexes - prices_rle$lengths + 1

# 为每一个价格创建一个总交易量的向量
volume_list <- list()
volume_vector <- as.numeric(spy_t_day$volume)
for (i in 1:length(end_indexes)) {
  volume_list[[i]] <- sum(volume_vector[start_indexes[i]:
    end_indexes[i]], na.rm = TRUE)
}

# 创建一个包含独一价格的简化数据集
spy_t_day_reduced <- spy_t_day[end_indexes, ]
spy_t_day_reduced$volume <- unlist(volume_list)
```

### 代码块 027

```text
head(spy_t_day_reduced, 10)
##                          price  volume
## 2013-04-15 08:30:01.964 158.17   510
## 2013-04-15 08:30:02.783 158.15   1000
## 2013-04-15 08:30:04.930 158.14   340
## 2013-04-15 08:30:11.964 158.12   100
## 2013-04-15 08:30:23.763 158.11   1100
## 2013-04-15 08:30:29.739 158.10   1720
## 2013-04-15 08:30:31.963 158.09   200
## 2013-04-15 08:30:45.164 158.08   4995
## 2013-04-15 08:30:46.888 158.07   100
## 2013-04-15 08:30:46.970 158.06   3330

head(spy_t_day, 10)
##                         price volume
## 2013-04-15 08:30:01.964 158.17   100
## 2013-04-15 08:30:01.964 158.17   410
## 2013-04-15 08:30:02.783 158.15   1000
## 2013-04-15 08:30:04.930 158.14   340
## 2013-04-15 08:30:11.964 158.12   100
## 2013-04-15 08:30:23.763 158.11   1000
## 2013-04-15 08:30:23.763 158.11   100
## 2013-04-15 08:30:28.153 158.10   400
## 2013-04-15 08:30:28.529 158.10   320
## 2013-04-15 08:30:29.739 158.10   180
```

### 代码块 028

```text
# 识别出每天中最多交易的价格
hist(as.numeric(spy_t_day_reduced$price), breaks = 200,
  main = "Histogram of traded prices for SPY on 2013-04-15",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 029

```text
acf(diff(log(as.numeric(spy_t_day_reduced$price))),
  main = "Autocorrelation of trades",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 030

```r
# 在滞后为1附近簇集的随机收益率
n <- rnorm(50, 0, .20)
n_lag1 <- c(NA, n[-length(n)])
plot(n_lag1, n)

# 在点之间生成箭头
s <- seq(length(n)-1)
arrows(n_lag1[s], n[s], n_lag1[s+1], n[s+1])
```

### 代码块 031

```r
# 簇集在滞后1处的SPY收益率
spy_t_returns <- diff(log(as.numeric(
  spy_t_day_reduced$price[100:150])))
spy_t_returns_lag1 <- c(NA, spy_t_returns[
  -length(spy_t_returns)])
plot(spy_t_returns_lag1, spy_t_returns)

s <- seq(length(spy_t_returns)-1)
arrows(spy_t_returns_lag1[s], spy_t_returns[s],
  spy_t_returns_lag1[s+1], spy_t_returns[s+1])
```

## highfrequency包

### 代码块 032

```r
library("devtools")
install_github("highfrequency", "jonathancornelissen")
```
