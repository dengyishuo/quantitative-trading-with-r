# 第 2 章 交易工具 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## R语言

### 代码块 001

```asm
.section .rodata
string:
.ascii "Hello, world!\n"
length:
.quad . -string
.section .text
.globl _start
_start:
movq $4, %rax
movq $1, %rbx
movq $string, %rcx
movq length, %rdx
int $0x80
movq %rax, %rbx
movq $1, %rax
int $0x80
```

### 代码块 002

```brainfuck
++++++++++[>+++++++>
++++++++++>+++>+>+++
+<<<<<-]> ++.>+.+++++++
..+++.>>>++++.<<++.<+++
+++++.--------.+++.------.-----
---.>+.>.
```

### 代码块 003

```cpp
#include <iostream>
int main()
{
std::cout << "Hello, world!" << std::endl;
return 0;
}
```

### 代码块 004

```text
println("Hello, world!")
```

### 代码块 005

```text
println("Hello, world!")
```

### 代码块 006

```text
print "Hello, world!"
```

### 代码块 007

```r
cat("Hello, world!\n")
```

## R入门

### 代码块 008

```text
1 + 1
## [1] 2
sqrt(2)
## [1] 1.414214
20 + (26.8 * 23.4) / 2 + exp(1.34) * cos(1)
## [1] 335.6234
sin(1)
## [1] 0.841471
5^4
## [1] 625
sqrt(-1 + 0i)
## [1] 0+1i
```

### 代码块 009

```r
integrand <- function(x) 1 / ((x + 1) * sqrt(x))
integrate(integrand, lower = 0, upper = Inf)
## 3.141593 with absolute error < 2.7e-05
```

### 代码块 010

```r
x <- 3
x <- x + 1
z <- x ^ 2
z <- "hello quants"
y <- "a"
Z <- sqrt(2)
new.X <- 2.3
```

### 代码块 011

```text
5 + 4
## [1] 9
```

## c()对象

### 代码块 012

```r
first_vector <- c(1, 2, 3, 4, 5, 6)
second_vector <- c("a", "b", "hello")
third_vector <- c("a", 2, 23)
```

### 代码块 013

```text
first_vector
## [1] 1 2 3 4 5 6
third_vector
## [1] "a" "2" "23"
```

### 代码块 014

```r
new_vector <- c(first_vector, 7, 8, 9)
new_vector
## [1] 1 2 3 4 5 6 7 8 9
```

### 代码块 015

```r
# 提取第四个元素
example_1 <- new_vector[4]
# 提取第五个和第八个元素
example_2 <- new_vector[c(5, 8)]
example_2
## [1] 5 8
```

### 代码块 016

```r
x <- c(1, 5, 10, 15, 20)
## [1] 1 5 10 15 20
x2 <- 2 * x
## [1] 2 10 20 30 40
x3 <- x ^ 2
## [1] 1 25 100 225 400
x4 <- x / x2
## [1] 0.5 0.5 0.5 0.5 0.5
x5 <- round(x * (x / x2) ^ 3.5 + sqrt(x4), 3)
## [1] 0.795 1.149 1.591 2.033 2.475
x6 <- round(c(c(x2[2:4], x3[1:2]), x5[4]), 2)
## [1] 10.00 20.00 30.00 1.00 25.00 2.03
```

## matrix()对象

### 代码块 017

```r
my_matrix <- matrix(c(1, 2, 3, 4, 5, 6),
nrow = 2, ncol = 3)
my_matrix
## [,1] [,2] [,3]
## [1,] 1 3 5
## [2,] 2 4 6
```

### 代码块 018

```r
my_matrix <- matrix(c(1, 2, 3, 4, 5, 6),
nrow = 2, ncol = 3, byrow = TRUE)
my_matrix
## [,1] [,2] [,3]
## [1,] 1 2 3
## [2,] 4 5 6
```

### 代码块 019

```r
dimnames(my_matrix) <- list(c("one", "hello"),
c("column1", "column2", "c3"))
my_matrix
## column1 column2 c3
## one 1 2 3
## hello 4 5 6
```

### 代码块 020

```text
attributes(my_matrix)
## $dim
## [1] 2 3
## $dimnames
## $dimnames[[1]]
## [1] "one" "hello"
## $dimnames[[2]]
## [1] "column1" "column2" "c3"
```

### 代码块 021

```r
ans <- my_matrix[1, 3]
ans
## [1] 3
# 矩阵也适用向量化运算。
new_matrix_1 <- my_matrix * my_matrix
new_matrix_1
## [,1] [,2] [,3]
## [1,] 1 4 9
## [2,] 16 25 36
new_matrix_2 <- sqrt(my_matrix)
new_matrix_2
## [,1] [,2] [,3]
## [1,] 1 1.414214 1.732051
## [2,] 2 2.236068 2.449490
```

### 代码块 022

```r
mat1 <- matrix(rnorm(1000), nrow = 100)
round(mat1[1:5, 2:6], 3)
## [,1] [,2] [,3] [,4] [,5]
## [1,] -1.544 1.281 1.397 0.407 -0.459
## [2,] 0.483 0.046 -1.817 -0.289 0.597
## [3,] 0.405 1.045 -0.726 -0.163 0.258
## [4,] 0.141 -0.294 -1.225 -0.217 -0.771
## [5,] -0.537 0.226 0.126 -1.584 -1.237
mat2 <- mat1[1:25, ] ^ 2
head(round(mat2, 0), 9)[,1:7]
## [,1] [,2] [,3] [,4] [,5] [,6] [,7]
## [1,] 1 2 2 2 0 0 7
## [2,] 0 0 0 3 0 0 0
## [3,] 0 0 1 1 0 0 1
## [4,] 0 0 0 2 0 1 4
## [5,] 1 0 0 0 3 2 1
## [6,] 2 1 3 1 1 1 1
## [7,] 0 0 0 0 0 1 0
## [8,] 1 2 0 0 1 2 0
## [9,] 0 0 3 0 2 2 0
```

## data.frame()对象

### 代码块 023

```r
df <- data.frame(price = c(89.2, 23.2, 21.2),
symbol = c("MOT", "AAPL", "IBM"),
action = c("Buy", "Sell", "Buy"))
df
## price symbol action
## 1 89.2 MOT Buy
## 2 23.2 AAPL Sell
## 3 21.2 IBM Buy
```

### 代码块 024

```r
df3 <-data.frame(price = c(89.2, 23.2, 21.2),
symbol = c("MOT", "AAPL", "IBM"),
action = c("Buy", "Sell", "Buy"),
stringsAsFactors = FALSE)
class(df3$symbol)
## [1] "character"
```

### 代码块 025

```r
price <- df[1, 1]
price
## [1] 89.2
df2 <- data.frame(col1 = c(1, 2, 3),
col2 = c(1, 2, 3, 4))
## Error in data.frame(col1 = c(1,2,3),
## col2 = c(1,2,3,4)) : arguments imply
## differing number of rows: 3, 4
```

### 代码块 026

```r
symbols <- df$symbol
symbols
## [1] MOT AAPL IBM
## Levels: AAPL IBM MOT
```

### 代码块 027

```text
class(symbols)
## [1] "factor"
```

### 代码块 028

```r
symbols <- df3$symbol
symbols
## [1] "MOT" "AAPL" "IBM"
```

## list()对象

### 代码块 029

```r
my_list <- list(a = c(1, 2, 3, 4, 5),
b = matrix(1:10, nrow = 2, ncol = 5),
c = data.frame(price = c(89.3, 98.2, 21.2),
stock = c("MOT", "IBM", "CSCO")))
my_list
## $a
## [1] 1 2 3 4 5
## $b
## [,1] [,2] [,3] [,4] [,5]
## [1,] 1 3 5 7 9
## [2,] 2 4 6 8 10
## $c
## price stock
## 1 89.3 MOT
## 2 98.2 IBM
## 3 21.2 CSCO
```

### 代码块 030

```r
first_element <- my_list[[1]]
first_element
## [1] 1 2 3 4 5
class(first_element)
## [1] "numeric"
```

### 代码块 031

```r
second_element <- my_list[["b"]]
second_element
## [,1] [,2] [,3] [,4] [,5]
## [1,] 1 3 5 7 9
## [2,] 2 4 6 8 10
class(second_element)
## [1] "matrix"
```

### 代码块 032

```r
part_of_list <- my_list[c(1, 3)]
part_of_list
## $a
## [1] 1 2 3 4 5
## $c
## price stock
## 1 89.3 MOT
## 2 98.2 IBM
## 3 21.2 CSCO
class(part_of_list)
## [1] "list"
```

### 代码块 033

```r
size_of_list <- length(my_list)
size_of_list
## [1] 3
```

## new.env()对象

### 代码块 034

```r
env <- new.env()
env[["first"]] <- 5
env[["second"]] <- 6
env$third <- 7
```

### 代码块 035

```text
env
## <environment: 0x101ef2f18>
```

### 代码块 036

```text
ls(env)
## [1] "first" "second" "third"
```

### 代码块 037

```text
get("first", envir = env)
## 5
```

### 代码块 038

```text
rm("second", envir = env)
ls(env)
## [1] "first" "third"
```

### 代码块 039

```r
env_2 <- env
env_2$third <- 42
get("third", envir = env)
## [1] 42
```

## 使用plot()函数

### 代码块 040

```r
# 创建一个向量，并绘制图形
x <- c(1, 2, 3.2, 4, 3, 2.1, 9, 19)
plot(x) 
```

### 代码块 041

```r
# 转换为线图
plot(x, type = "l")
```

### 代码块 042

```r
# 设置画布
plot(rnorm(1000), main = "Some returns", cex.main = 0.9,
xlab = "Time", ylab = "Returns")
# 添加网格线
grid()
# 绘制横线和竖线
abline(v = 400, lwd = 2, lty = 1)
abline(h = 2, lwd = 3, lty = 3)
```

### 代码块 043

```r
# 创建两行两列的画图窗口
par(mfrow = c(2, 2))
# 绘制散点图
plot(rnorm(100), main = "Graph 1")
# 绘制折线图
plot(rnorm(100), main = "Graph 2", type = "l")
# 绘制阶梯图
plot(rnorm(100), main = "Graph 3", type = "s")
abline(v = 50, lwd = 4)
# 绘制火柴棒图
plot(rnorm(100), type = "h", main = "Graph 4")
# 重置绘画窗口
par(mfrow = c(1, 1))
```

### 代码块 044

```r
plot(rnorm(100), main = "A line plot",
cex.main = 0.8,
xlab = "x-axis",
ylab = "y-axis",
type = "l")
# 添加文字
mtext("Some text at the top", side = 3)
#  x = 40和y = -1 处添加图例
legend(40, -1, "A legend")
```

### 代码块 045

```text
formals(plot.default)
## $x
## $y
## NULL
## $type
## [1] "p"
## ...
```

## 函数式编程

### 代码块 046

```r
ans <- sum(1:100)
ans
## [1] 5050
```

### 代码块 047

```r
answer <- 0
for(i in 1:100) {
answer <- answer + i
}
answer
## [1] 5050
```

## 编写R函数

### 代码块 048

```r
crack_eggs <- function(number_of_eggs) {
# 这是一段判断鸡蛋是否磕碎的代码
# 如果鸡蛋被磕碎了，设定have_all_eggs_been_cracked为TRUE,
# 否则，为FALSE
return(have_all_eggs_been_cracked)
}
```

### 代码块 049

```r
# 生成100个随机数
x <- rnorm(100, mean = 0, sd = 1)

# 返回向量的长度
length(x)

# 计算均值
mean(x)

# 计算标准差
sd(x)

# 计算中位数
median(x)

# 返回向量的最大值和最小值
range(x)

# 对向量求和
sum(x)

# 对向量累计求和
cumsum(x)

# 提取向量的前3个元素
head(x, 3)

# 计算向量x的描述性统计量
summary(x)

# 对x升序排序
sort(x, decreasing = TRUE)

# 计算向量x的差分序列
diff(x)

# 创建1-10的整数序列
1:10

# 创建起始值为1，终值为10，步长为0.1的序列
seq(1, 10, 0.1)

# 输出字符串“hello”
print("hello")
```

## 分支和循环

### 代码块 050

```r
# 定义一个布尔变量
my_boolean <- 1 == 2

if (my_boolean) {
  print("not correct")
} else {
  print("XYZ")
}
```

### 代码块 051

```r
for(i in 1:5) {
  cat(i, "\n")
}
## 1
## 2
## 3
## 4
## 5

some_list <- list()
for(z in c("hello", "goodbye")) {
  some_list[[z]] <- z
}

some_list
## $hello
## [1] "hello"

## $goodbye
```

## 推荐的代码风格

### 代码块 052

```r
# 数据求和
x<-0;for(i in 1:10){x=x+1};x

# 数据求和
x <- 0
for(i in 1:10) {
  x <- x + 1
}
x
```

## 两两相关示例

### 代码块 053

```r
filter_and_sort_symbols <- function(symbols) {
  # 函数名字: filter_and_sort_symbols
  # 函数功能: 将股票代码转换为大写字母
  # 并删除无效代码
  # 输入参数: 参数symbols对应一个股票代码向量
  # 输出结果: 有效的股票代码
  
  # 将股票代码转换为大写字母
  symbols <- toupper(symbols)
  
  # 过滤有效的代码
  valid <- regexpr("^[A-Z]{2,4}$", symbols)
  
  # 返回有效代码
  return(sort(symbols[valid == 1]))
}
```

### 代码块 054

```r
filter_and_sort_symbols(c("MOT", "cvx", "123", "Gog2", "XLe")) 
## "MOT" "CVX" "XLE"
```

### 代码块 055

```r
extract_prices <- function(filtered_symbols, file_path) {
  # 函数名字: extract_prices
  # 函数功能: 从指定文件读取价格数据
  # 输入参数: filtered_symbols为一系列股票代码,
  #         file_path为股票价格数据的存储路径
  # 输出结果: prices为包含所有股票价格数据的数据框
  
  # 读取.csv格式的价格数据
  all_prices <- read.csv(file = file_path, header = TRUE,
                         stringsAsFactors = FALSE)
  
  # 设置行名
  rownames(all_prices) <- all_prices$Date
  
  # 删除原始日期列
  all_prices$Date <- NULL
  
  # 提取对应股票的价格数据
  valid_columns <- colnames(all_prices) %in% filtered_symbols
  
  return(all_prices[, valid_columns])
}
```

### 代码块 056

```r
A <- c(1, 2, 5, 6, 9)
B <- c(0, 3, 6, 9, 10)

A %in% B
## [1] FALSE FALSE FALSE  TRUE  TRUE
```

### 代码块 057

```r
filter_prices <- function(prices) {
  # 函数名字: filter_prices
  # 函数功能: 识别包含缺失值的行
  # 函数参数: prices，包含股票价格的数据框
  # 函数输出: missing_rows，包含缺失值的行序号，以及记录是否为完整行的布尔向量
  valid_rows <- complete.cases(prices)
  
  # 识别包含缺失值的行序
  missing_rows <- which(valid_rows == FALSE)
  
  return(missing_rows)
}
```

### 代码块 058

```r
compute_pairwise_correlations <- function(prices) {
  # 函数名字: compute_pairwise_correlations
  # 函数功能: 计算收益率序列的两两相关系数，并绘制两两相关散点图
  # 函数参数: prices，包含股票价格数据的数据框
  # 函数输出: correlation_matrix，相关系数矩阵
  
  # 计算收益率
  returns <- apply(prices, 2, function(x) diff(log(x)))
  
  # 绘制收益率序列的两两散点图
  pairs(returns, main = "Pairwise return scatter plot")
  
  # 计算两两相关系数
  correlation_matrix <- cor(returns, use = "complete.obs")
  
  return(correlation_matrix)
}
```

### 代码块 059

```r
# 用户输入的股票代码
symbols <- c("IBM", "XOM", "2SG", "TEva",
             "G0og", "CVX", "AAPL", "BA")

# 包含股票价格数据的数据库路径
file_path <- "Chapter_02/prices.csv"

# 对股票代码进行过滤和排序
filtered_symbols <- filter_and_sort_symbols(symbols)
filtered_symbols
## [1] "AAPL" "BA"   "CVX"  "IBM"  "TEVA" "XOM"

# 提取价格数据
prices <- extract_prices(filtered_symbols, file_path)

# 过滤价格数据
missing_rows <- filter_prices(prices)
missing_rows
## integer(0)

# 计算相关性
correlation_matrix <- compute_pairwise_correlations(prices)
correlation_matrix
```
