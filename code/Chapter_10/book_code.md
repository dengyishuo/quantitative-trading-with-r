# 第 10 章 最优化 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 以抛物线为引子

### 代码块 001

```r
# 创建该函数
f <- function(x) {
 return((1 + x) ^ 2)
}

# 创建该函数的一阶导数函数
fp <- function(x) {
 return(2 * (1 + x))
}

# 画出函数图及其一阶导数函数图
x <- seq(-5, 5, 0.1)
plot(x, f(x), type = 'l', lwd = 2,
  main = "f(x) and f'(x)",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()
lines(x, fp(x), lty = 3, lwd = 2)
abline(h = 0)
abline(v = 0)
```

## 牛顿法

### 代码块 002

```r
f <- function(x) {
    return(x ^ 2 - 4 * x + 1)
}
```

### 代码块 003

```r
uniroot(f, c(-8, -1))
## $root
## [1] -4.236068

## $f.root
## [1] -2.568755e-07

## $iter
## [1] 8

## $estim.prec
## [1] 6.103516e-05

uniroot(f, c(-1, 2))
## $root
## [1] 0.236044

## $f.root
## [1] -0.0001070205

## $iter
## [1] 6

## $estim.prec
## [1] 6.103516e-05
```

### 代码块 004

```r
# 基于一阶逼近的牛顿法
newton <- function(f, tol = 1E-12, x0 = 1, N = 20) {
  # N = 总迭代次数
  # x0 = 初始猜测值
  # tol = abs(xn+1 - xn)
  # f = 待求根的函数表达式

  h <- 0.001
  i <- 1; x1 <- x0
  p <- numeric(N)
  while (i <= N) {
    df_dx <- (f(x0 + h) - f(x0)) / h
    x1 <- (x0 - (f(x0) / df_dx))
    p[i] <- x1
    i <- i + 1
    if (abs(x1 - x0) < tol) {
      break
    }
    x0 <- x1
  }
  return(p[1:(i-1)])
}
```

### 代码块 005

```text
newton(f, x0 = -10)
## [1] -6.312270 -4.735693 -4.281609 -4.236513
## [5] -4.236068 -4.236068 -4.236068 -4.236068

newton(f, x0 = 10)
## [1] 4.2085746 1.5071738 0.4665599 0.2468819
## [5] 0.2360964 0.2360680 0.2360680 0.2360680
```

### 代码块 006

```text
options(digits = 14)
newton(f, x0 = 10)
## [1] 4.20857464272283 1.50717378796250 0.46655991958819
## [5] 0.24688186711921 0.23609640037269 0.23606798403444
## [8] 0.23606797750125 0.23606797749979 0.23606797749979
```

### 代码块 007

```text
newton(f, x0 = 0.25)
## [1] 0.23611419684515 0.23606798830978 0.23606797750221
## [4] 0.23606797749979 0.23606797749979
```

### 代码块 008

```r
# 创建一个函数表达式
e <- expression(sin(x))

# 计算导数函数
D(e, "x")
## cos(x)
```

### 代码块 009

```r
f_expr <- expression(x ^ 2 + 4 * x - 1)
```

### 代码块 010

```text
eval(f_expr, list(x = 2))
## [1] 11
```

### 代码块 011

```r
newton_alternate <- function(f, tol = 1E-12, x0 = 1, N = 20) {
  # N = 总迭代次数
  # x0 = 初始猜测值
  # tol = abs(xn+1 - xn)
  # f = 待求根的函数表达式

  # 计算导数函数的符号形式
  df_dx = D(f, "x")

  i <- 1; x1 <- x0
  p <- numeric(N)
  while (i <= N) {
    x1 <- (x0 - eval(f, list(x = x0)) /
      eval(df_dx, list(x = x0)))
    p[i] <- x1
    i <- i + 1
    if (abs(x1 - x0) < tol) {
      break
    }
    x0 <- x1
  }
  return(p[1:(i-1)])
}

newton_alternate(f_expr, x0 = 10)
 ## [1] 4.20833333333333 1.50685123042506 0.46631585084907
 ## [4] 0.24681560399775 0.23609368309733 0.23606797764754
 ## [7] 0.23606797749979 0.23606797749979

newton_alternate(f_expr, x0 = -10)
## [1] -6.3125000000000 -4.7359601449275 -4.2817360731259
## [4] -4.2365249924418 -4.2360680241934 -4.2360679774998
## [7] -4.2360679774998
```

## 暴力求解法

### 代码块 012

```r
# 生成一系列关于x的随机点
set.seed(123)
x <- rnorm(100, 0, 1)

# 令y为关于x的一个函数
y <- 3.2 + 2.9 * x + rnorm(100, 0, 0.1)

plot(x, y)
```

### 代码块 013

```r
objective_function <- function(y, x, a, b) {
  value <- sum((y - (a * x + b)) ^ 2)
  return(value)
}
```

### 代码块 014

```r
# 创建一系列的a与b值，并用循环遍历每一个值
a <- seq(-10, 10, 0.25)
b <- seq(-10, 10, 0.25)

output <- list()
z <- 1
for(i in 1:length(a)) {
  for(j in 1:length(b)) {
    output[[z]] <- c(objective_function(y, x, a[i], b[j]),
      a[i], b[j])
    z <- z + 1
  }
}

# 用输出的列表对象生成一个矩阵并从中找到最小值
mat <- do.call(rbind, output)
colnames(mat) <- c("obj", "a", "b")

smallest <- which(mat[, "obj"] == min(mat[, "obj"]))

mat[smallest, ]
##   obj      a      b
## 2.16076 3.00000 3.25000
```

### 代码块 015

```text
a = seq(-5, 5, 0.1)
b = seq(-5, 5, 0.1)
```

### 代码块 016

```text
##   obj       a      b
## 0.9077592 2.9000 3.2000
```

## R中的最优化程序

### 代码块 017

```r
args(optim)
function (par, fn, gr = NULL, ...,
  method = c("Nelder-Mead",
  "BFGS", "CG", "L-BFGS-B", "SANN", "Brent"),
  lower = -Inf,
  upper = Inf,
  control = list(),
  hessian = FALSE)
```

## 一个关于曲线拟合的练习

### 代码块 018

```r
# 创建一个虚拟的收益值向量
rates = c(0.025, 0.03, 0.034, 0.039, 0.04,
  0.045, 0.05, 0.06, 0.07, 0.071,
  0.07, 0.069, 0.07, 0.071, 0.072,
  0.074, 0.076, 0.082, 0.088, 0.09)
maturities = 1:20

plot(maturities, rates, xlab = "years",
  main = "Yields",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()
```

### 代码块 019

```r
poly_5 <- function(x, p) {
  f <- p[1] + p[2] * x + p[3] * x^2 +
    p[4] * x^3 + p[5] * x^4 + p[6] * x^5
  return(f)
}

obj_5 <- function(x, y, p) {
  error <- (y - poly_5(x, p)) ^ 2
  return(sum(error))
}

# 拟合参数。假设初始值都为0
out_5 = optim(obj_5, par = c(0, 0, 0, 0, 0, 0),
  x = maturities, y = rates)

out_5
## $par
## [1]  2.4301235956099e-02  1.3138951147963e-03
## [3]  5.5229326602931e-04  7.5685740385076e-07
## [5] -4.2119475163787e-06  1.5330958809806e-07

## $value
## [1] 0.00017311660458207

## $counts
## function gradient
##    501       NA

## $convergence
## [1] 1

## $message
## NULL
```

### 代码块 020

```text
lines(poly_5(maturities, out_5$par), lwd = 1.5, lty = 2)
```

### 代码块 021

```r
poly_7 <- function(x, p) {
  f <- p[1] + p[2] * x + p[3] * x^2 +
    p[4] * x^3 + p[5] * x^4 +
    p[6] * x^5 + p[6] * x^6 +
    p[7] * x^7
  return(f)
}

obj_7 <- function(x, y, p) {
  error <- (y - poly_7(x, p)) ^ 2
  return(sum(error))
}

# 拟合参数。假设初始值都为0
out_7 <- optim(obj_7, par = c(0, 0, 0, 0, 0, 0, 0, 0),
  x = maturities, y = rates)

lines(poly_7(maturities, out_7$par), lwd = 1.5, lty = 3)
```

### 代码块 022

```r
# 设定两个用于拟合的多项式函数
poly_5 <- function(x, a) {
  f <- a[1] + a[2] * x + a[3] * x ^ 2 +
    a[4] * x ^ 3 + a[5] * x ^ 4 +
    a[6] * x ^ 5
return(f)
}

poly_3 <- function(x, offset, intercept, b) {
  f <- intercept + b[1] * (x - offset) +
    b[2] * (x - offset) ^ 2 +
    b[3] * (x - offset) ^ 3
  return(f)
}
```

### 代码块 023

```r
obj_3_5 <- function(x, y, offset, p) {

  # 所有的初始值都设定为无穷大
  fit <- rep(Inf, length(x))
  ind_5 <- x <= offset
  ind_3 <- x > offset

  fit[ind_5] <- poly_5(x[ind_5], p[1:6])
  fit[ind_3] <- poly_3(x[ind_3], offset,
    poly_5(offset, p[1:6]), p[7:9])

  error <- (y - fit) ^ 2
  return(sum(error))
}

# 拟合参数。假设所有初始值为0.
offset <- 9
out_3_5 <- optim(obj_3_5, par = rep(0, 9),
  x = maturities, y = rates, offset = offset)

plot(maturities, rates, xlab = "years",
  main = "Yields",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()
lines(poly_5(maturities[maturities <= offset],
  out_3_5$par[1:6]), lwd = 2)
lines(c(rep(NA, offset),
  poly_3(maturities[maturities > offset], offset,
  poly_5(offset, out_3_5$par[1:6]),
  out_3_5$par[7:9])), lwd = 2)
abline(v = offset)
```

### 代码块 024

```r
# 对数据用loess进行拟合
obj <- loess(rates ~ maturities, span = 0.5)

# 画出数据与拟合曲线图
plot(maturities, rates, main = "Rates", cex.main = 0.8)
lines(predict(obj), lty = 2)
```

## 投资组合最优化

### 代码块 025

```r
install.packages("DEoptim")

require(DEoptim)

# 回撤函数
compute_drawdown <- function(x, returns_default = TRUE,
  geometric = TRUE) {
  # x = 损益（PnL)或者收益率向量
  # 如果 returns_default = FALSE, 那么geometric
  # 参数会被忽略，使用PnL作为x输入
  # 输出 = 最大回撤
  if(returns_default) {
    # 计算累积收益
    if(geometric) {
      cumulative_return <- cumprod(1 + x)
    } else {
      cumulative_return <- 1 + cumsum(x)
    }
    max_cumulative_return <- cummax(c(1, cumulative_return))[-1]
    drawdown <- -(cumulative_return / max_cumulative_return - 1)
  } else {
    # 使用PnL向量
    cumulative_pnl <- c(0, cumsum(x))
    drawdown <- cummax(cumulative_pnl) - cumulative_pnl
    drawdown <- drawdown[-1]
  }
  # 输出PnL或者收益率的最大回撤向量
  return(drawdown)
}

obj_max_drawdown <- function(w, r_matrix, small_weight) {
  # w 是每个股票的权重系数
  # r_matrix 是所有股票的收益率矩阵

  # 投资组合收益率
  portfolio_return <- r_matrix %*% w

  # 最大回撤
  drawdown_penalty <- max(compute_drawdown(portfolio_return))

  # 生成权重和的惩罚成分
  weight_penalty <- 100 * (1 - sum(w)) ^ 2

  # 生成负权重值的惩罚成分
  negative_penalty <- -sum(w[w < 0])

  # 生成小权重值的惩罚成分
  small_weight_penalty <- 100 * sum(w[w < small_weight])

  # 待最小化的目标函数
  obj <- drawdown_penalty + weight_penalty +
    negative_penalty + small_weight_penalty
  return(obj)
}

# 对多个股票计算收益率矩阵
symbol_names <- c("AXP", "BA", "CAT", "CVX",
  "DD", "DIS", "GE", "HD", "IBM",
  "INTC", "KO", "MMM", "MRK",
  "PG", "T", "UTX", "VZ")

# 将这些价格序列载入内存
price_matrix <- NULL
for(name in symbol_names) {
  # Extract the adjusted close price vector
  price_matrix <- cbind(price_matrix, get(name)[, 6])
}

colnames(price_matrix) <- symbol_names

# 计算收益率
returns_matrix <- apply(price_matrix, 2, function(x) diff(log(x)))

# 设定一个小权重阈值，低于该阈值的权重都强制降为0%。
small_weight_value <- 0.02

# 设定权重的上下限
lower <- rep(0, ncol(returns_matrix))
upper <- rep(1, ncol(returns_matrix))

optim_result <- DEoptim(obj_max_drawdown, lower, upper,
  control = list(NP = 400, itermax = 300, F = 0.25, CR = 0.75),
  returns_matrix, small_weight_value)
```

### 代码块 026

```text
...

Iteration: 299 bestvalit: 0.515606 bestmemit:
    0.020494 0.021594 0.020232
    0.044563 0.020729 0.172967
    0.021248 0.105254 0.084343
    0.032526 0.067988 0.034999
    0.020617 0.196651 0.042869
    0.033833 0.056965  

Iteration: 300 bestvalit: 0.515606 bestmemit:
    0.020494 0.021594 0.020232
    0.044563 0.020729 0.172967
    0.021248 0.105254 0.084343
    0.032526 0.067988 0.034999
    0.020617 0.196651 0.042869
    0.033833 0.056965
```

### 代码块 027

```r
weights <- optim_result$optim$bestmem
```

### 代码块 028

```text
sum(weights)
0.9978
```

### 代码块 029

```r
weights <- weights / sum(weights)
```

### 代码块 030

```r
# 等权重投资组合
equal_weights <- rep(1 / 17, 17)
equal_portfolio <- returns_matrix %*% equal_weights
equal_portfolio_cumprod <- cumprod(1 + equal_portfolio)

# 最优最大回撤投资组合
optimized_portfolio <- returns_matrix %*% weights
drawdown_portfolio_cumprod <- cumprod(1 + optimized_portfolio)

main_title <- "Equal vs. Optimized Weights"
plot(drawdown_portfolio_cumprod, type = 'l', xaxt = 'n',
  main = main_title, xlab = "", ylab = "cumprod(1 + r)")
lines(equal_portfolio_cumprod, lty = 3)
grid(col = 'black')

# 设定x轴的标签
label_location <- seq(1, length(drawdown_portfolio_cumprod),
  by = 90)
labels <- rownames(returns_matrix)[label_location]
axis(side = 1, at = label_location, labels = labels,
  las = 2, cex.axis= 0.8)
```

### 代码块 031

```text
# 等权重组合
max(compute_drawdown(equal_portfolio))
## [1] 0.597

# 最小化最大回撤的最优化组合
max(compute_drawdown(optimized_portfolio))
## [1] 0.515
```
