# 第 11 章 速度 测试与报告 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 运行时执行的改进

### 代码块 001

```r
sum_with_loop_in_r <- function(max_value) {
  sum <- 0
  for(i in 1:max_value) {
    sum <- sum + i
  }
  return(sum)
}
```

### 代码块 002

```r
sum_with_vectorization_in_r = function(max_value) {
  numbers <- as.double(1:max_value)
  return(sum(numbers))
}
```

## R代码基准测试

### 代码块 003

```r
library(microbenchmark)
microbenchmark(loop = sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5))

## Unit: microseconds
## expr        min       lq        median
## loop        57615.323 59424.740 60992.7720
## vectorized  260.602   273.673   286.5495

## uq        max       neval
## 89608.441 96694.469 100
## 294.236   414.349   100
```

### 代码块 004

```r
compiled_sum_with_loop_in_r <- cmpfun(sum_with_loop_in_r)
  microbenchmark(loop = sum_with_loop_in_r(1e5),
  compiled = compiled_sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5))

## Unit: microseconds
## expr        min       lq         median
## loop        56746.652 58343.8945 60602.445
## compiled    4688.146  4758.6770  4892.246
## vectorized  249.457   273.8635   284.050

##  uq         max        neval
##  86599.9875 96736.750  100
##  5498.9710  46484.009  100
##  292.3135   473.927    100
```

## Rcpp的解决方案

### 代码块 005

```text
long add_cpp(long max_value) {
  long sum = 0;
  for(long i = 1; i <= max_value; ++i) {
    sum = sum + i;
  }
  return sum;
}
```

### 代码块 006

```r
lapply
## function (X, FUN, ...)
## {
##   FUN <- match.fun(FUN)
##   if (!is.vector(X) || is.object(X))
##     X <- as.list(X)
##     .Internal(lapply(X, FUN))
## }
```

### 代码块 007

```r
library(Rcpp)

# 创建一个C++函数
cppFunction('
  long add_cpp(long max_value) {
    long sum = 0;
    for(long i = 1; i <= max_value; ++i) {
     sum = sum + i;
    }
    return sum;
   }'
)
```

### 代码块 008

```r
add_cpp
## function (max_value)
## .Primitive(".Call")(<pointer: 0x10f52fbb0>, max_value)
```

### 代码块 009

```text
add_cpp(1e5)
## [1] 5000050000
```

### 代码块 010

```text
microbenchmark(loop = sum_with_loop_in_r(1e5),
  compiled = compiled_sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5),
  compiled_cpp = add_cpp(1e5))

## Unit: microseconds
## expr         min       lq         median
## loop         73049.461 76640.5945 79635.8810
## compiled     7359.040  7487.9655  7795.9125
## vectorized   804.773   932.9285   1031.9695
## compiled_cpp 79.573    88.2615    98.9385

## uq         max        neval
## 80676.6600 94618.174  100
## 12101.8610 135353.743 100
## 1373.8565  2148.409   100
## 105.2440   135.781    100
```

### 代码块 011

```cpp
# include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
long add_2_cpp(long max_value) {
    long sum = 0;
    for(long i = 1; i <= max_value ; i++) {
        sum = sum + i;
    }
    return sum;
}
```

### 代码块 012

```text
sourceCpp('Chapter_11/add_2_file.cpp')

add_2_cpp(100)
## [1] 5050
```

## 用RInside在C++中调用R

### 代码块 013

```cpp
#include <RInside.h>
int main(int argc, char *argv[]) {

 // create an embedded R instance
 RInside R(argc, argv);

 // assign a char* (string) to "txt"
 R["txt"] = "Hello, world!\n";

 // eval the init string, ignoring any returns
 R.parseEvalQ("cat(txt)");

 exit(0);
}
```

## 用testthat编写单元测试

### 代码块 014

```r
# 定义函数
convert_to_returns <- function(prices) {
  return(9)
}
```

### 代码块 015

```r
require(testthat)

# 用context()将所需的功能聚合起来
context("Price to log-return conversion")

# 用expect_that()定义我们期待的结果
test_that("convert_to_returns produces the correct values", {

  # 对于这些输入
  input_prices <- c(100, 101, 102, 103, 99)

  # 我们期待返回这些结果
  expected_returns <- c(0.009950331,
    0.009852296, 0.009756175, -0.039609138)

  # 验证实际返回值与期待值是否等同
  expect_equal(expected_returns,
    convert_to_returns(input_prices))
})
```

### 代码块 016

```text
## Error: Test failed: 'convert_to_returns produces
## the correct values'
## Not expected: expected_returns not equal to
## convert_to_returns(input_prices)
## Numeric: lengths (1, 4) differ.
```

### 代码块 017

```r
# 定义函数
convert_to_returns <- function(prices) {
  return(diff(log(prices)))
}
```

### 代码块 018

```r
# 验证错误信息
input_prices <- c(100)
msg <- "Not enough price entries."

expect_message(msg, convert_to_returns(input_prices))
## Error: expected_message no messages shown
```

### 代码块 019

```r
# 检查极端情况的函数
convert_to_returns <- function(prices) {
  if(length(prices) < 2) {
    message("Not enough price entries.")
  }
  return(diff(log(prices)))
}
```

### 代码块 020

```text
test_file("example_test_file.r")
```

## 使用knitr撰写文档

### 代码块 021

```tex
\documentclass{article}
\usepackage[T1]{fontenc}

\begin{document}

This is an example LaTeX document with some embedded R code woven in for convenience.

<<foo, fig.height = 4>>=
x = 1:10
y = x ^ 2

plot(x, y, main = "This is a graph")
@
行内表达可以使用\verb|\Sexpr{}|，比如$\pi=\Sexpr{pi}$，\Sexpr{2.3492e7}和\Sexpr{x / 2.0}。
\subsection*{A different subsection}
我们可以插入绘图而隐藏代码。这个功能由代码块的参数列表完成。

<<foo2, fig.height = 3, echo = FALSE>>=
x = 1:10
y = x ^ 3
plot(x, y, main = "This is a second graph")
@

任何R代码都可以在knitr提供的代码块中运行。下面一个例子加载\texttt{ggplot2},并创建一个漂亮的密度直方图。

<<foo3, fig.height = 6, tidy = FALSE >>=
require(ggplot2)
my_data = data.frame(returns = c(0.03, 0.04, 0.05,
0.032, 0.01, 0.23, 0.4, 0.05, 0.066, 0.5),
stock = c("SPY", "CVX", "CVX", "SPY",
"XOM", "XOM", "CVX", "SPY", "SPY", "XOM"))
ggplot(my_data, aes(x = returns, fill = stock)) +
geom_density(alpha = 0.2)
@
\end{document}
```

### 代码块 022

```r
x = 1:10
y = x ^ 2

plot(x, y, main = "这是一幅图")
```

### 代码块 023

```r
x = 1:10
y = x ^ 3
plot(x, y, main = "这是第二幅图")
```

### 代码块 024

```r
require(ggplot2)
## Loading required package: ggplot2
my_data = data.frame(returns = c(0.03, 0.04, 0.05,
0.032, 0.01, 0.23, 0.4, 0.05, 0.066, 0.5),
stock = c("SPY", "CVX", "CVX", "SPY",
"XOM", "XOM", "CVX", "SPY", "SPY", "XOM"))
ggplot(my_data, aes(x = returns, fill = stock)) +
geom_density(alpha = 0.2)
```
