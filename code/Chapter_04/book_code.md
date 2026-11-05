# 第 4 章 统计和概率基础 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 总体和样本

### 代码块 001

```r
# 设置随机数种子
set.seed(100)
X <- rnorm(1000000, mean = 2.33, sd = 0.5)
mu <- mean(X)
sd <- sd(X)
hist(X, breaks = 100)
abline(v = mu, lwd = 3, lty = 2)
```

### 代码块 002

```text
set.seed(12)
rnorm(5)
## [1] -1.4805676 1.5771695 -0.9567445 -0.9200052 -1.9976421
```

### 代码块 003

```text
rnorm(5)
## [1] -0.2722960 -0.3153487 -0.6282552 -0.1064639  0.4280148
```

### 代码块 004

```r
sample5  <- sample(X, 5, replace = TRUE)
sample10 <- sample(X, 10, replace = TRUE)
sample50 <- sample(X, 50, replace = TRUE)

sample5
## [1] 2.497921 2.635927 2.291848 2.127974 2.268268

sample10
## [1] 2.064451 2.274464 2.468938 1.800007 2.557669
## [6] 2.535241 1.331020 1.159151 1.661762 2.285889

sample50
## [1] 2.581844 2.138331 3.003670 1.864148 2.049141
## [6] 2.808971 1.400057 2.527640 3.639216 3.311873

mean(sample5)
## [1] 2.364388

mean(sample10)
## 2.013859

mean(sample50)
## 2.447003

mean(sample(X, 1000, replace = TRUE))
## 2.323124

mean(sample(X, 10000, replace = TRUE))
## [1] 2.334109
```

## 中心极限定理在R里的实现

### 代码块 005

```r
mean_list <- list()
for(i in 1:10000) {
  mean_list[[i]] <- mean(sample(X, 10, replace = TRUE))
}

hist(unlist(mean_list), breaks = 500,
  xlab = "Mean of 10 samples from X",
  main = "Convergence of sample distribution",
  cex.main = 0.8)
abline(v = mu, lwd = 3, col = "white", lty = 2)
```

### 代码块 006

```r
population <- sample(c(0, 1), 100000, replace = TRUE)
hist(population, main = "Non-normal", cex.main = 0.8)
abline(v = mean(population), lwd = 3, lty = 3)
```

### 代码块 007

```r
mean_list <- list()
for(i in 1:10000) {
  mean_list[[i]] <- mean(sample(population, 10, replace = TRUE))
}

hist(unlist(mean_list), main = "Distribution of averages",
  cex.main = 0.8,
  xlab = "Average of 10 samples")
abline(v = 0.5, lwd = 3)
```

## 无偏性和有效性

### 代码块 008

```r
# 总体方差公式
population_variance <- function(x) {
  mean <- sum(x) / length(x)
  return(sum((x - mean) ^ 2) / length(x))
}

# 创建一个总体
population <- as.numeric(1:100000)
variance <- population_variance(population)

variance
## [1] 833333333
```

### 代码块 009

```r
output <- list()
for(i in 1:1000) {
  output[[i]] <- population_variance(sample(population,
                                            10, replace = TRUE))
}

variance_estimates <- unlist(output)
hist(variance_estimates, breaks = 100, cex.main = 0.9)
average_variance <- mean(variance_estimates)
abline(v = average_variance, , lty = 2, lwd = 2)
abline(v = variance, lwd = 2)

average_variance
## [1] 738123625
```

### 代码块 010

```r
# 无偏方差估计量的公式
sample_variance <- function(x) {
  mean <- sum(x) / length(x)
  return(sum((x - mean) ^ 2) / (length(x) - 1))
}

output <- list()

for( i in 1:1000 ) {
  output[[i]] <- sample_variance(sample(population,
                                        10, replace = TRUE))
}

sample_variance_estimates <- unlist(output)
average_sample_variance <- mean(sample_variance_estimates)

average_sample_variance
## [1] 836184961
```

## 概率分布

### 代码块 011

```r
plot(c(-1, 1), c(0.5, 0.5), type = "h", lwd = 3,
xlim = c(-2, 2), main = "Probability mass function of coin
toss",
ylab = "Probability",
xlab = "Random Variable",
cex.main = 0.9)
```

## 掷硬币模拟器

### 代码块 012

```r
outcomes <- sample(c(0, 1), 1000, replace = TRUE)
```

### 代码块 013

```r
set.seed(101) 
biased_outcomes <- sample(c(0, 1), 1000, 
  replace = TRUE, prob = c(0.4, 0.6))
```

### 代码块 014

```r
prob_estimate <- sum(biased_outcomes) / 
  length(biased_outcomes)
prob_estimate 
## [1] 0.603
```
