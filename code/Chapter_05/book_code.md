# 第 5 章 统计和概率中级 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 股票价格分布

### 代码块 001

```r
# 提取价格并计算相关统计量值
prices <- SPY$SPY.Adjusted
mean_prices <- round(mean(prices), 2)
sd_prices <- round(sd(prices), 2)

# 绘制包含图例的直方图
hist(prices, breaks = 100, prob=T, cex.main = 0.9)
abline(v = mean_prices, lwd = 2)
legend("topright", cex = 0.8, border = NULL, bty = "n",
  paste("mean=", mean_prices, "; sd=", sd_prices))
```

### 代码块 002

```r
plot_4_ranges <- function(data, start_date, end_date, title) {

  # 设定绘图窗口为两行两列
  par(mfrow = c(2, 2))
  for(i in 1:4) {
    # 创建一个合适时间范围的字符串
    range <- paste(start_date[i], "::", end_date[i], sep = "")

    # 创建价格向量以及需要的统计量
    time_series <- data[range]

    mean_data <- round(mean(time_series, na.rm = TRUE), 3)
    sd_data <- round(sd(time_series, na.rm = TRUE), 3)

    # 绘制包含图例的直方图
    hist_title <- paste(title, range)
    hist(time_series, breaks = 100, prob=TRUE,
     xlab = "", main = hist_title, cex.main = 0.8)
    legend("topright", cex = 0.7, bty = 'n',
     paste("mean=", mean_data, "; sd=", sd_data))
  }

  # 重置绘图窗口  
  par(mfrow = c(1, 1))
}
```

### 代码块 003

```r
# 定义好我们感兴趣的时间起点和结点
begin_dates <- c("2007-01-01", "2008-06-06",
  "2009-10-10", "2011-03-03")
end_dates <- c("2008-06-05", "2009-09-09",
  "2010-12-30", "2013-01-06")

# 绘图
plot_4_ranges(prices, begin_dates,
  end_dates, "SPY prices for:")
```

## 平稳性

### 代码块 004

```r
# 计算对数收益率
returns <- diff(log(prices))

# 使用跟之前相同的函数绘制收益率而不是价格
plot_4_ranges(returns, begin_dates, end_dates, "SPY log prices for:")
```

## 用urca判断平稳

### 代码块 005

```r
# 获取SPY数据并且确认其为非平稳
require(quantmod)
getSymbols("SPY")
spy <- SPY$SPY.Adjusted

# 使用默认设置
require(urca)
test <- ur.kpss(as.numeric(spy))

# 输出结果是一个S4对象
class(test)
## [1] "ur.kpss"
## attr(,"package")
## [1] "urca"

# 提取检验统计量值test@teststat
test@teststat
## [1] 11.63543
# 看一下临界值
test@cval
##                              10pct  5pct    2.5pct 1pct
## critical values  0.347  0.463  0.574  0.739
```

### 代码块 006

```r
spy_returns <- diff(log(spy))

# 检验收益率的平稳性
test_returns <- ur.kpss(as.numeric(spy_returns))
test_returns@teststat
## [1] 0.336143
test_returns@cval
```

### 代码块 007

```r
test_post_2013 <- ur.kpss(as.numeric(spy_returns['2013::']))
test_post_2013@teststat
## [1] 0.06936672
```

## 正态性假设

### 代码块 008

```r
# 绘制直方图和密度曲线
mu <- mean(returns, na.rm = TRUE)
sigma <- sd(returns, na.rm = TRUE)
x <- seq(-5 * sigma, 5 * sigma, length = nrow(returns))

hist(returns, breaks = 100,
  main = "Histogram of returns for SPY",
  cex.main = 0.8, prob=TRUE)
lines(x, dnorm(x, mu, sigma), col = "red", lwd = 2)
```

### 代码块 009

```r
# 设定绘图窗口为一行两列
par(mfrow = c(1, 2))

# SPY数据
qqnorm(as.numeric(returns),
  main = "SPY empirical returns qqplot()",
  cex.main = 0.8)
qqline(as.numeric(returns),  lwd = 2)
grid()

# 正态分布随机数据
normal_data <- rnorm(nrow(returns), mean = mu, sd = sigma)

qqnorm(normal_data, main = "Normal returns", cex.main = 0.8)
qqline(normal_data, lwd = 2)
grid()
```

### 代码块 010

```r
answer <- shapiro.test(as.numeric(returns))

answer[[2]]
## [1] 5.118396e-34
```

### 代码块 011

```r
set.seed(129)
normal_numbers <- rnorm(5000, 0, 1)
ans <- shapiro.test(normal_numbers)

ans[[2]]
## [1] 0.9963835
```

### 代码块 012

```r
# 人为破坏一个数据点
normal_numbers[50] <- 1000
ans <- shapiro.test(normal_numbers)

ans[[2]]
## [1] 1.775666e-95
```

## 相关性

### 代码块 013

```r
sv <- as.xts(returns_matrix[, c(1, 6)])

head(sv)
##               SPY.Close    VXX.Close
## 2009-02-02 -0.003022794 -0.003160468
## 2009-02-03  0.013949240 -0.047941603
## 2009-02-04 -0.004908132  0.003716543
## 2009-02-05  0.014770965 -0.006134680
```

### 代码块 014

```text
cor(sv)
##            SPY.Close  VXX.Close
## SPY.Close  1.0000000 -0.4603908
## VXX.Close -0.4603908  1.0000000
```

## 过滤数据

### 代码块 015

```r
# 找出异常值
outliers <- which(sv[, 2] > 1.0)

# 如果异常值存在则删除之
if(length(outliers) > 0) {
  sv <- sv[-outliers, ]
}

cor(sv)
##            SPY.Close  VXX.Close
## SPY.Close  1.0000000 -0.8066466
## VXX.Close -0.8066466  1.0000000
```

## R 公式

### 代码块 016

```r
# 创建一个公式
my_formula <- as.formula("y ~ x")

# 输出是什么？ 
my_formula
## y ~ x

# my_formula的类是什么？ 
class(my_formula)
## [1] "formula"
```

### 代码块 017

```r
# 创建一个线性回归的对象
reg <- lm(VXX.Close ~ SPY.Close, data = sv)

# 下面是输出结果
summary(reg)

## Call:
## lm(formula = VXX.Close ~ SPY.Close, data = sv)

## Residuals:
##    Min        1Q    Median        3Q       Max
## -0.085607 -0.012830 -0.000865  0.012188  0.116349

## Coefficients:
##            Estimate Std. Error t value Pr(>|t|)
## (Intercept) -0.0024365  0.0006641  -3.669 0.000254 ***
## SPY.Close   -2.5848492  0.0552193 -46.811  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1

## Residual standard error: 0.02287 on 1187 degrees of freedom
## Multiple R-squared:  0.6486,Adjusted R-squared:  0.6483
## F-statistic:  2191 on 1 and 1187 DF,  p-value: < 2.2e-16
```

### 代码块 018

```r
b <- reg$coefficients[1]
a <- reg$coefficients[2]
```

### 代码块 019

```r
par(mfrow = c(2, 2))
plot(reg$residuals,
     main = "Residuals through time",
     xlab = "Days", ylab = "Residuals")
hist(reg$residuals, breaks = 100,
     main = "Distribution of residuals",
     xlab = "Residuals")
qqnorm(reg$residuals)
qqline(reg$residuals)
acf(reg$residuals, main = "Autocorrelation")
```

### 代码块 020

```r
vxx_lag_1 <- lag(VXX$VXX.Close, k = 1)

head(vxx_lag_1)
##               VXX.Close
## 2009-01-30    NA
## 2009-02-02    104.58
## 2009-02-03    104.25
## 2009-02-04    99.37
## 2009-02-05    99.74
## 2009-02-06    99.13

head(VXX$VXX.Close)
##               VXX.Close
## 2009-01-30    104.58
## 2009-02-02    104.25
## 2009-02-03    99.37
## 2009-02-04    99.74
## 2009-02-05    99.13
## 2009-02-06    97.70
```

### 代码块 021

```r
# 将收益率序列和收益率的滞后序列合并
sv <- merge(sv, lag(sv))

# SPY时候序列和VXX序列的散点图
plot(as.numeric(sv[, 3]), as.numeric(sv[, 2]),
  main = "Scatter plot SPY lagged vs. VXX.",
  xlab = "SPY lagged",
  ylab = "VXX",
  cex.main = 0.8,
  cex.axis = 0.8,
  cex.lab = 0.8)
grid()
```

### 代码块 022

```r
reg2 <- lm(VXX.Close ~ SPY.Close.1, data = sv)

summary(reg2)
## Coefficients:
##           Estimate Std. Error t value Pr(>|t|)
## (Intercept) -0.004140   0.001121  -3.694 0.000231 ***
## SPY.Close.1  0.104119   0.093154   1.118 0.263918

## Residual standard error: 0.03857 on 1186 degrees of freedom
## (1 observation deleted due to missingness)
## Multiple R-squared:  0.001052,Adjusted R-squared:  0.00021
## F-statistic: 1.249 on 1 and 1186 DF,  p-value: 0.2639
```

### 代码块 023

```text
ccf(as.numeric(sv[, 1]), as.numeric(sv[, 2]),
    main = "Cross correlation between SPY and VXX",
    ylab = "Cross correlation", xlab = "Lag", cex.main = 0.8,
    cex.lab = 0.8, cex.axis = 0.8)
```

## 线性回归中的线性

### 代码块 024

```r
x <- seq(1:100)
y <- x ^ 2

# 生成散点图
plot(x, y)

# 拟合回归
reg_parabola <- lm(y ~ x)

# 在散点图上添加最优拟合直线
abline(reg_parabola, lwd = 2)

# 检视回归结果
summary(reg_parabola)
```

### 代码块 025

```r
plot(x, sqrt(y))
reg_transformed <- lm(sqrt(y) ~ x)
abline(reg_transformed)

summary(reg_transformed)
```

## 波动性

### 代码块 026

```r
# 生成1000个正态分布的IID随机数
z <- rnorm(1000, 0, 1)

# 收益率和收益率平方序列的自相关图
par(mfrow = c(2, 1))
acf(z, main = "returns", cex.main = 0.8,
    cex.lab = 0.8, cex.axis = 0.8)

grid()
acf(z ^ 2, main = "returns squared",
    cex.lab = 0.8, cex.axis = 0.8)
grid()
```

### 代码块 027

```r
par(mfrow = c(1, 1))
acf(sv[, 1] ^ 2, main = "Actual returns squared",
    cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
grid()
```

### 代码块 028

```r
par(mfrow = c(1, 2))
acf(sv[, 1]^3)
acf(abs(sv[, 1]))
```
