# 第 6 章 价格差 贝塔系数和风险 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 定义股票价格差

### 代码块 001

```r
pepsi <- getSymbols('PEP', from = '2013-01-01',
  to = '2014-01-01', adjust = T, auto.assign = FALSE)

coke <- getSymbols('COKE', from = '2013-01-01',
  to = '2014-01-01', adjust = T, auto.assign = FALSE)

Sys.setenv(TZ = "UTC")

prices <- cbind(pepsi[, 6], coke[, 6])
price_changes <- apply(prices, 2, diff)
plot(price_changes[, 1], price_changes[, 2],
  xlab = "Coke price changes",
  ylab = "Pepsi price changes",
  main = "Pepsi vs. Coke",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()

ans <- lm(price_changes[, 1] ~ price_changes[, 2])
beta <- ans$coefficients[2]
```

### 代码块 002

```r
ans2 <- lm(price_changes[, 2] ~ price_changes[, 1])
beta2 <- ans2$coefficients[2]

beta
## [1] 0.2614627

beta2
## [1] 0.2539855
```

## 普通最小二乘法和总体最小二乘法

### 代码块 003

```r
# 获取数据
SPY <- getSymbols('SPY', from = '2011-01-01',
  to = '2012-12-31', adjust = T, auto.assign = FALSE)

AAPL <- getSymbols('AAPL', from = '2011-01-01',
  to = '2012-12-31', adjust = T, auto.assign = FALSE)

# 计算价格的差
x <- diff(as.numeric(SPY[, 4]))
y <- diff(as.numeric(AAPL[, 4]))

plot(x, y, main = "Scatter plot of returns. SPY vs. AAPL",
  cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
abline(lm(y ~ x))
abline(lm(x ~ y), lty = 2)
grid()

# 总体最小二乘回归
r <- prcomp( ~ x + y )
slope <- r$rotation[2, 1]  /  r$rotation[1, 1]
intercept <- r$center[2] - slope * r$center[1]

# 在图中显示第一主成分
abline(a = intercept, b = slope, lty = 3)
```

## 构建价格差

### 代码块 004

```r
# 计算价格差的函数
calculate_spread <- function(x, y, beta) {
  return(y - beta * x)
}

# 给定起始和结束日期，计算beta与平均值的函数
calculate_beta_and_level <- function(x, y,
  start_date, end_date) {
  require(xts)

  time_range <- paste(start_date, "::",
    end_date, sep = "")
  x <- x[time_range]
  y <- y[time_range]

  dx <- diff(x[time_range])
  dy <- diff(y[time_range])
  r <- prcomp( ~ dx + dy)

  beta <- r$rotation[2, 1] / r$rotation[1, 1]
  spread <- calculate_spread(x, y, beta)
  names(spread) <- "spread"
  level <- mean(spread, na.rm = TRUE)

  outL <- list()
  outL$spread <- spread
  outL$beta <- beta
  outL$level <- level

  return(outL)
}

# 计算买卖信号与上下阈值的函数
calculate_buy_sell_signals <- function(spread, beta,
  level, lower_threshold, upper_threshold) {

  buy_signals <- ifelse(spread <= level -
   lower_threshold, 1, 0)
  sell_signals <- ifelse(spread >= level +
    upper_threshold, 1, 0)

  # 将这些向量捆绑为一个矩阵
  output <- cbind(spread, buy_signals,
    sell_signals)
  colnames(output) <- c("spread", "buy_signals",
    "sell_signals")

  return(output)
}
```

### 代码块 005

```r
# 实现
# 选择一个样本内数据区间
start_date <- "2009-01-01"
end_date <- "2011-12-31"
x <- SPY[, 6]
y <- AAPL[, 6]

results <- calculate_beta_and_level(x, y,
  start_date, end_date)
results$beta
## [1] 4.923278

results$level
## [1] -239.0602

plot(results$spread, ylab = "Spread Value",
  main = "AAPL - beta * SPY",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```

### 代码块 006

```r
# 样本外起始与结束日期
start_date_out_sample <- "2012-01-01"
end_date_out_sample <- "2012-10-22"
range <- paste(start_date_out_sample, "::",
  end_date_out_sample, sep = "")

# 样本外分析
spread_out_of_sample <- calculate_spread(x[range],
  y[range], results$beta)

plot(spread_out_of_sample, main = "AAPL - beta * SPY",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
abline(h = results$level, lwd = 2)
```

## 生成信号和验证

### 代码块 007

```r
# 交易日滚动窗口
window_length <- 10

# 时间区间
start_date <- "2011-01-01"
end_date <- "2011-12-31"
range <- paste(start_date, "::",
  end_date, sep = "")

# 我们的股票配对
x <- SPY[range, 6]
y <- AAPL[range, 6]

dF <- cbind(x, y)
names(dF) <- c("x", "y")

# 用于计算beta值的函数
run_regression <- function(dF) {
  return(coef(lm(y ~ x - 1, data = as.data.frame(dF))))
}

rolling_beta <- function(z, width) {
  rollapply(z, width = width, FUN = run_regression,
  by.column = FALSE, align = "right")
}

betas <- rolling_beta(diff(dF), 10)

data <- merge(betas, dF)
data$spread <- data$y - lag(betas, 1) * data$x
```

### 代码块 008

```r
returns <- diff(dF) / dF
return_beta <- rolling_beta(returns, 10)
data$spreadR <- diff(data$y) / data$y -
  return_beta * diff(data$x) / data$x
```

### 代码块 009

```text
tail(data)
##            betas    x      y      spread     spreadR
## 2011-12-22 2.770586 119.60 383.07 138.70795 -0.002322110
## 2011-12-23 3.094533 120.67 387.66  53.33343  0.003311904
## 2011-12-27 3.450416 120.76 390.74  17.04417  0.007083611
## 2011-12-28 3.364819 119.18 387.00 -24.22055  0.004194527
## 2011-12-29 3.004804 120.41 389.38 -15.77781 -0.003361064
```

### 代码块 010

```r
threshold <- sd(data$spread, na.rm = TRUE)

threshold
## [1] 143.7734
```

### 代码块 011

```r
plot(data$spread, main = "AAPL vs. SPY In-Sample",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
abline(h = threshold, lty = 2)
abline(h = -threshold, lty = 2)
```

### 代码块 012

```r
# 创建样本外价格差
# 日滚动窗口保持不变
window_length <- 10

# 时间区间
start_date <- "2012-01-01"
end_date <- "2013-12-31"
range <- paste(start_date, "::",
  end_date, sep = "")

# 我们的股票配对
x <- SPY[range, 6]
y <- AAPL[range, 6]

# 将其捆绑为一个矩阵
dF <- cbind(x, y)
names(dF) <- c("x", "y")

# 计算样本外滚动beta值
beta_out_of_sample <- rolling_beta(diff(dF), 10)

# 买卖阈值
data_out <- merge(beta_out_of_sample, dF)
data_out$spread <- data_out$y -
  lag(beta_out_of_sample, 1) * data_out$x

# 画出样本内价格差的分布区间
plot(data_out$spread, main = "AAPL vs. SPY out of sample",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
abline(h = threshold, lwd = 2)
abline(h = -threshold, lwd = 2)
```

### 代码块 013

```r
# 生成买卖信号
buys <- ifelse(data_out$spread > threshold, 1, 0)
sells <- ifelse(data_out$spread < -threshold, -1, 0)
data_out$signal <- buys + sells
```

### 代码块 014

```r
plot(data_out$spread, main = "AAPL vs. SPY out of sample",
 cex.main = 0.8,
 cex.lab = 0.8,
 cex.axis = 0.8)
abline(h = threshold, lty = 2)
abline(h = -threshold, lty = 2)

point_type <- rep(NA, nrow(data_out))
buy_index <- which(data_out$signal == 1)
sell_index <- which(data_out$signal == -1)

point_type[buy_index] <- 21
point_type[sell_index] <- 24
points(data_out$spread, pch = point_type)
```

### 代码块 015

```r
num_of_buy_signals <- sum(buys, na.rm = TRUE)
num_of_sell_signals <- sum(abs(sells), na.rm = TRUE)

num_of_buy_signals
## [1] 303
num_of_sell_signals
## [1] 189
```

## 价格差交易

### 代码块 016

```text
##     beta_out_of_sample     x      y    spread signal
## 2011-01-13         NA 128.37 345.68        NA     NA
## 2011-01-14  1.7511157 129.30 348.48        NA     NA
## 2011-01-18  1.1630714 129.52 340.65 113.84550      1
## 2011-01-19  1.2803161 128.25 338.84 189.67609      1
## 2011-01-20  1.2286891 128.08 332.68 168.69711      1
## 2011-01-21  0.8045108 128.37 326.72 168.99319      1
## 2011-01-24  2.4936855 129.10 337.45 233.58766      1
## 2011-01-25  2.7762163 129.17 341.40  19.29065      0
## 2011-01-26  3.0802946 129.67 343.85 -16.14196      0
```

### 代码块 017

```r
prev_x_qty <- 0
position <- 0
trade_size <- 100
signal <- as.numeric(data_out$signal)
signal[is.na(signal)] <- 0
beta <- as.numeric(data_out$beta_out_of_sample)

qty_x <- rep(0, length(signal))
qty_y <- rep(0, length(signal))

for(i in 1:length(signal)) {
  if(signal[i] == 1 && position == 0) {
    # buy the spread买入价格差
    prev_x_qty <- round(beta[i] * trade_size)
    qty_x[i] <- -prev_x_qty
    qty_y[i] <- trade_size
    position <- 1
  }

  if(signal[i] == -1 && position == 0) {
    # 初始时卖出价格差
    prev_x_qty <- round(beta[i] * trade_size)
    qty_x[i] <- prev_x_qty
    qty_y[i] <- -trade_size
    position <- -1
  }

  if(signal[i] == 1 && position == -1) {
  #看空价差并买入
  qty_x[i] <- -(round(beta[i] * trade_size) +
    prev_x_qty)
  prev_x_qty <- round(beta[i] * trade_size)
  qty_y[i] <- 2 * trade_size
  position <- 1
  }

  if(signal[i] == -1 && position == 1) {
    #看多价差并卖出
    qty_x[i] <- round(beta[i] * trade_size) + prev_x_qty
    prev_x_qty <- round(beta[i] * trade_size)
    qty_y[i] <- -2 * trade_size
    position <- -1
  }
}
```

### 代码块 018

```r
qty_x[length(qty_x)] <- -sum(qty_x)
qty_y[length(qty_y)] <- -sum(qty_y)
```

### 代码块 019

```r
data_out$qty_x <- qty_x
data_out$qty_y <- qty_y

data_out[1:3, ]
##  beta_out_of_sample        x      y  spread signal qty_x qty_y
## 2012-01-17  2.1511279 123.48 408.20     NA     NA    0     0
## 2012-01-18  2.5890817 124.85 412.44  143.87168  1 -259   100
## 2012-01-19  2.0711505 125.51 411.13   86.17435  0    0     0

tail(data_out, 3)
##   beta_out_of_sample       x      y  spread signal qty_x qty_y
## 2012-12-27  6.5051194 138.15 499.45 -404.90307 -1    0     0
## 2012-12-28  5.6770827 136.66 494.14 -394.84962 -1    0     0
## 2012-12-31  6.3934172 138.98 516.04 -272.96095 -1 -668   100
```

### 代码块 020

```r
# 计算资金曲线的函数
compute_equity_curve <- function(qty, price) {

  cash_buy <- ifelse(sign(qty) == 1,
    qty * price, 0)
  cash_sell <- ifelse(sign(qty) == -1,
    -qty * price, 0)
  position <- cumsum(qty)
  cumulative_buy <- cumsum(cash_buy)
  cumulative_sell <- cumsum(cash_sell)

  equity <- cumulative_sell - cumulative_buy +
    position * price
  return(equity)
}

# 将资金曲线列并入data_out表
data_out$equity_curve_x <- compute_equity_curve(
  data_out$qty_x, data_out$x)
data_out$equity_curve_y <- compute_equity_curve(
  data_out$qty_y, data_out$y)

plot(data_out$equity_curve_x +
  data_out$equity_curve_y, type = 'l',
  main = "AAPL / SPY spread", ylab = "P&L",
  cex.main = 0.8,
  cex.axis = 0.8,
  cex.lab = 0.8)
```

### 代码块 021

```r
plot(data_out$equity_curve_x +
  data_out$equity_curve_y, type = 'l',
  main = "AAPL / SPY spread", ylab = "P&L",
  cex.main = 0.8,
  cex.axis = 0.8,
  cex.lab = 0.8)
```

## 资金曲线的进一步探究

### 代码块 022

```r
# 计算夏普比率
sharpe_ratio <- function(x, rf) {
  sharpe <- (mean(x, na.rm = TRUE) - rf) /
    sd(x, na.rm = TRUE)
  return(sharpe)
}

# 计算最大损失信息
drawdown <- function(x) {
  cummax(x) - x
}
```

### 代码块 023

```r
par(mfrow = c(2, 1))
equity_curve <- data_out$equity_curve_x + data_out$equity_curve_y

plot(equity_curve, main = "Equity Curve",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)

plot(drawdown(equity_curve), main = "Drawdown of equity curve",
 cex.main = 0.8,
 cex.lab = 0.8,
 cex.axis = 0.8)
```

### 代码块 024

```r
equity <- as.numeric(equity_curve[, 1])
equity_curve_returns <- diff(equity) / equity[-length(equity)]

# 去除无限大和空值
invalid_values <- is.infinite(equity_curve_returns)
  | is.nan(equity_curve_returns)

sharpe_ratio(equity_curve_returns[!invalid_values], 0.03)
## [1] 0.0658528
```

### 代码块 025

```r
omega_ratio <- function(r, T) {
  omega <- mean(pmax(r - T, 0)) / mean(pmax(T - r, 0))
  return(omega)
}
```

## 策略属性

### 代码块 026

```r
# 找出交易发生的时刻
trade_dates <- data_out$qty_x[data_out$qty_x != 0]

# trade_dates是一个xts对象，其索引包含必要的时间信息
duration <- as.numeric(diff(index(trade_dates)))

# 汇总统计量
summary(duration)
## Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
## 1.00   13.50   21.00   31.84   44.00  128.00

# 交易久期的直方图
hist(duration, breaks = 20,
  main = "Histogram of trade durations",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
```
