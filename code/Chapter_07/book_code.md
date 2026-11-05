# 第 7 章 基于 Quantstrat 包的回溯测试 书稿代码摘录

本文件按完整中文书稿的出现顺序收录代码块，包含控制台输出和少量非 R 语言示例。可运行的原仓库代码请优先参考同目录的 `code.R`。

## 初始设置

### 代码块 001

```r
# 禁止警告
options("getSymbols.warning4.0" = FALSE)

# 清理工作区
rm(list = ls(.blotter), envir = .blotter)

# 设定货币种类和交易时区
currency('USD')
Sys.setenv(TZ = "UTC")

# 定义感兴趣的证券名
symbols <- c("XLB", #SPDR Materials sector
  "XLE", #SPDR Energy sector
  "XLF", #SPDR Financial sector
  "XLP", #SPDR Consumer staples sector
  "XLI", #SPDR Industrial sector
  "XLU", #SPDR Utilities sector
  "XLV", #SPDR Healthcare sector
  "XLK", #SPDR Tech sector
  "XLY", #SPDR Consumer discretionary sector
  "RWR", #SPDR Dow Jones REIT ETF
  "EWJ", #iShares Japan
  "EWG", #iShares Germany
  "EWU", #iShares UK
  "EWC", #iShares Canada
  "EWY", #iShares South Korea
  "EWA", #iShares Australia
  "EWH", #iShares Hong Kong
  "EWS", #iShares Singapore
  "IYZ", #iShares U.S. Telecom
  "EZU", #iShares MSCI EMU ETF
  "IYR", #iShares U.S. Real Estate
  "EWT", #iShares Taiwan
  "EWZ", #iShares Brazil
  "EFA", #iShares EAFE
  "IGE", #iShares North American Natural Resources
  "EPP", #iShares Pacific Ex Japan
  "LQD", #iShares Investment Grade Corporate Bonds
  "SHY", #iShares 1-3 year TBonds
  "IEF", #iShares 3-7 year TBonds
  "TLT" #iShares 20+ year Bonds
)

# 先SPDR ETFs（交易所交易基金），后iShares ETFs
if(!"XLB" %in% ls()) {
  # 如果数据不存在，从yahoo读取
  suppressMessages(getSymbols(symbols, from = from,
    to = to, src = "yahoo", adjust = TRUE))
}

# 定义证券类型
stock(symbols, currency = "USD", multiplier = 1)
```

## 第一个交易策略：简单趋势跟踪

### 代码块 002

```r
"lagATR" <- function(HLC, n = 14, maType, lag = 1, ...) {
   ATR <- ATR(HLC, n = n, maType = maType, ...)
   ATR <- lag(ATR, lag)
   out <- ATR$atr
   colnames(out) <- "atr"
   return(out)
}

"osDollarATR" <- function(orderside, tradeSize, pctATR,
                          maxPctATR = pctATR, data, timestamp,
                          symbol, prefer = "Open", portfolio, integerQty = TRUE, atrMod = "", rebal = FALSE, ...) {

  if(tradeSize > 0 & orderside == "short"){
    tradeSize <- tradeSize * -1
  }

  pos <- getPosQty(portfolio, symbol, timestamp)
  atrString <- paste0("atr", atrMod)
  atrCol <- grep(atrString, colnames(mktdata))
  
  if(length(atrCol) == 0) {
    stop(paste("Term", atrString,
      "not found in mktdata column names."))
  }

  atrTimeStamp <- mktdata[timestamp, atrCol]
  if(is.na(atrTimeStamp) | atrTimeStamp == 0) {
    stop(paste("ATR corresponding to", atrString,
      "is invalid at this point in time. Add a logical
      operator to account for this."))
  }

  dollarATR <- pos * atrTimeStamp
  desiredDollarATR <- pctATR * tradeSize
  remainingRiskCapacity <- tradeSize*maxPctATR – dollarATR

  if(orderside == "long"){
    qty <- min(tradeSize * pctATR / atrTimeStamp,
      remainingRiskCapacity / atrTimeStamp)
  } else {
    qty <- max(tradeSize * pctATR / atrTimeStamp,
      remainingRiskCapacity / atrTimeStamp)
  }
  if(integerQty) {
    qty <- trunc(qty)
  }
  if(!rebal) {
    if(orderside == "long" & qty < 0) {
       qty <- 0
    }
    if(orderside == "short" & qty > 0) {
       qty <- 0
    }
  }
  if(rebal) {
    if(pos == 0) {
     qty <- 0
    }
  }
  return(qty)
}
```

### 代码块 003

```r
require(quantstrat)
require(PerformanceAnalytics)

initDate = "1990-01-01"
from = "2003-01-01"
to = "2013-12-31"
options(width = 70)

# 再次执行策略，从下面一行开始运行
# demoData.R包括所有数据相关的模板代码
source("demoData.R")

# 交易规模和初始资本设定
tradeSize <- 10000
initEq <- tradeSize * length(symbols)

strategy.st <- "Clenow_Simple"
portfolio.st <- "Clenow_Simple"
account.st <- "Clenow_Simple"
rm.strat(portfolio.st)
rm.strat(strategy.st)

initPortf(portfolio.st, symbols = symbols,
initDate = initDate, currency = 'USD')

initAcct(account.st, portfolios = portfolio.st,
initDate = initDate, currency = 'USD', initEq = initEq)

initOrders(portfolio.st, initDate = initDate)

strategy(strategy.st, store=TRUE)
```

## 回溯测试第一个交易策略

### 代码块 004

```r
nLag = 252
pctATR = 0.02
period = 10

namedLag <- function(x, k = 1, na.pad = TRUE, ...) {
  out <- lag(x, k = k, na.pad = na.pad, ...)
  out[is.na(out)] <- x[is.na(out)]
  colnames(out) <- "namedLag"
  return(out)
}
```

### 代码块 005

```r
add.indicator(strategy.st, name = "namedLag",
  arguments = list(x = quote(Cl(mktdata)), k = nLag),
  label = "ind")

add.indicator(strategy.st, name = "lagATR",
  arguments = list(HLC = quote(HLC(mktdata)), n = period),
  label = "atrX")

test <- applyIndicators(strategy.st, mktdata = OHLC(XLB))
head(round(test, 2), 253)
```

### 代码块 006

```r
# 信号
add.signal(strategy.st, name = "sigCrossover",
  arguments = list(columns = c("Close", "namedLag.ind"),
  relationship = "gt"),
  label = "coverOrBuy")

add.signal(strategy.st, name = "sigCrossover",
  arguments = list(columns = c("Close", "namedLag.ind"),
  relationship = "lt"),
  label = "sellOrShort")
```

### 代码块 007

```text
# 看涨规则
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "coverOrBuy",
  sigval = TRUE, ordertype = "market",
  orderside = "long", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR,
  tradeSize = tradeSize, pctATR = pctATR,
  atrMod = "X"), type = "enter", path.dep = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "sellOrShort",
  sigval = TRUE, orderqty = "all",
  ordertype = "market", orderside = "long",
  replace = FALSE, prefer = "Open"),
  type = "exit", path.dep = TRUE)

# 看跌规则
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "sellOrShort",
  sigval = TRUE, ordertype = "market",
  orderside = "short", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR,
  tradeSize = -tradeSize, pctATR = pctATR,
  atrMod = "X"), type = "enter", path.dep = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "coverOrBuy",
  sigval = TRUE, orderqty = "all",
  ordertype = "market", orderside = "short",
  replace = FALSE, prefer = "Open"),
  type = "exit", path.dep = TRUE)
```

### 代码块 008

```r
# 获取起始时间
t1 <- Sys.time()
out <- applyStrategy(strategy = strategy.st,
  portfolios = portfolio.st)

# 记录结束时间
t2 <- Sys.time()
print(t2 - t1)
```

### 代码块 009

```text
## [1] "2007-10-22 00:00:00 XLY -655 @ 32.3578893111826"
## [1] "2007-10-22 00:00:00 XLY -393 @ 32.3578893111826"
## [1] "2007-10-23 00:00:00 XLY 393 @ 33.1349846702336"
## [1] "2007-10-23 00:00:00 XLY 358 @ 33.1349846702336"
## [1] "2007-10-25 00:00:00 XLY -358 @ 32.8639048938205"
## [1] "2007-10-25 00:00:00 XLY -333 @ 32.8639048938205"
## [1] "2009-09-30 00:00:00 XLY 333 @ 25.9947501843176"
## [1] "2009-09-30 00:00:00 XLY 449 @ 25.9947501843176"
## [1] "2009-10-02 00:00:00 XLY -449 @ 24.8800203565938"
```

## 评估策略表现

### 代码块 010

```r
 updatePortf(portfolio.st)
 dateRange <- time(getPortfolio(portfolio.st)$summary)[-1]
 updateAcct(portfolio.st, dateRange)
 updateEndEq(account.st)
```

### 代码块 011

```r
tStats <- tradeStats(Portfolios = portfolio.st, use = "trades",
  inclZeroDays = FALSE)
tStats[, 4:ncol(tStats)] <- round(tStats[, 4:ncol(tStats)], 2)

print(data.frame(t(tStats[,-c(1,2)])))
aggPF <- sum(tStats$Gross.Profits) / -sum(tStats$Gross.Losses)
aggCorrect <- mean(tStats$Percent.Positive)
numTrades <- sum(tStats$Num.Trades)
meanAvgWLR <- mean(tStats$Avg.WinLoss.Ratio[
  tStats$Avg.WinLoss.Ratio < Inf], na.rm = TRUE)
```

### 代码块 012

```text
##          XLK XLP XLU XLV XLY
## Num.Txns 105.00 33.00 41.00 93.00 69.00
## Num.Trades 53.00 17.00 21.00 45.00 35.00
## Net.Trading.PL 6016.85 23438.64 20018.73 24191.83 16349.64
## Avg.Trade.PL 113.53 1378.74 953.27 537.60 467.13
## Med.Trade.PL -97.78 -41.14 -114.53 -79.73 -136.29
## Largest.Winner 7203.51 5403.21 14892.92 1710.61 2287.43
## Largest.Loser -1541.41 -231.73 -895.47 -1196.50 -786.39
## Gross.Profits 16494.40 24645.83 23631.91 31237.12 22885.87
## Gross.Losses -10477.55 -1207.19 -3613.17 -7045.29 -6536.23
## Std.Dev.Trade.PL 1197.87 4154.99 3387.87 3619.68 3041.63
## Percent.Positive 35.85 35.29 33.33 37.78 31.43
## Percent.Negative 64.15 64.71 66.67 62.22 68.57
## Profit.Factor 1.57 20.42 6.54 4.43 3.50
## Avg.Win.Trade 868.13 4107.64 3375.99 1837.48 2080.53
## Med.Win.Trade 162.15 1287.61 2128.49 225.36 219.68
## Avg.Losing.Trade -308.16 -109.74 -258.08 -251.62 -272.34
## Med.Losing.Trade -172.44 -115.39 -193.79 -202.68 -209.24
## Avg.Daily.PL 51.89 428.46 894.51 3.13 -37.87
## Med.Daily.PL -98.89 -48.45 -130.50 -82.54 -141.43
## Std.Dev.Daily.PL 1121.49 1428.19 3464.89 503.18 579.12
## Ann.Sharpe 0.73 4.76 4.10 0.10 -1.04
## Max.Drawdown -8098.15 -3819.66 -8212.64 -5755.91 -7084.35
## Profit.To.Max.Draw 0.74 6.14 2.44 4.20 2.31
## Avg.WinLoss.Ratio 2.82 37.43 13.08 7.30 7.64
## Med.WinLoss.Ratio 0.94 11.16 10.98 1.11 1.05
## Max.Equity 6016.85 23457.62 21210.88 24220.69 16349.64
## Min.Equity -5258.58 -330.75 -709.58 -3914.56 -4354.96
## End.Equity 6016.85 23438.64 20018.73 24191.83 16349.64
```

### 代码块 013

```r
aggPF <- sum(tStats$Gross.Profits) / -sum(tStats$Gross.Losses)
## [1] 3.663545

aggCorrect <- mean(tStats$Percent.Positive)
## [1] 36.00233

<- sum(tStats$Num.Trades)
## [1] 1134

meanAvgWLR <- mean(tStats$Avg.WinLoss.Ratio[
  tStats$Avg.WinLoss.Ratio < Inf], na.rm = TRUE)
## [1] 9.871333
```

### 代码块 014

```r
instRets <- PortfReturns(account.st)

portfRets <- xts(rowMeans(instRets) * ncol(instRets),
  order.by = index(instRets))
portfRets <- portfRets[!is.na(portfRets)]
cumPortfRets <- cumprod(1 + portfRets)
firstNonZeroDay <- as.character(index(portfRets)[
  min(which(portfRets != 0))])

# 获取证券
getSymbols("SPY", from = firstNonZeroDay, to = to)
SPYrets <- diff(log(Cl(SPY)))[-1]
cumSPYrets <- cumprod(1 + SPYrets)
comparison <- cbind(cumPortfRets, cumSPYrets)
colnames(comparison) <- c("strategy", "SPY")
chart.TimeSeries(comparison, legend.loc = "topleft",
  colors=c("green", "red"))

# 计算风险测量
SharpeRatio.annualized(portfRets)
Return.annualized(portfRets)
maxDrawdown(portfRets)
```

### 代码块 015

```text
SharpeRatio.annualized(portfRets)
## [1] 0.6424366

Return.annualized(portfRets)
## [1] 0.1392711

maxDrawdown(portfRets)
## [1] 0.278221
```

### 代码块 016

```r
chart.Posn(portfolio.st, "XLB")
tmp <- namedLag(Cl(XLB), k = nLag)
add_TA(tmp$namedLag, col = "blue", on = 1)
```

## 第二个交易策略：累计康纳斯相对强弱指数（Connors RSI）

### 代码块 017

```r
 # 基于RSI函数计算康纳斯相对强弱指数
connorsRSI <- function(price, nRSI = 3, nStreak = 2,
  nPercentLookBack = 100 ) {
  priceRSI <- RSI(price, nRSI)
  streakRSI <- RSI(computeStreak(price), nStreak)
  percents <- round(runPercentRank(x = diff(log(price)),
    n = 100, cumulative = FALSE, exact.multiplier = 1) * 100)
  ret <- (priceRSI + streakRSI + percents) / 3
  colnames(ret) <- "connorsRSI"
  return(ret)
}

 # 计算连续的价格变化正负带
computeStreak <- function(priceSeries) {
  signs <- sign(diff(priceSeries))
  posDiffs <- negDiffs <- rep(0,length(signs))
  posDiffs[signs == 1] <- 1
  negDiffs[signs == -1] <- -1
  
  # 建立累积和的向量，累积和并不递增
  # na.loc（向后填充缺失值）之后将NA转换为0 
  posCum <- cumsum(posDiffs)
  posNAcum <- posCum
  posNAcum[posDiffs == 1] <- NA
  posNAcum <- na.locf(posNAcum, na.rm = FALSE)
  posNAcum[is.na(posNAcum)] <- 0
  posStreak <- posCum - posNAcum

  # 对负的累积和重复这一过程
  negCum <- cumsum(negDiffs)
  negNAcum <- negCum
  negNAcum[negDiffs == -1] <- NA
  negNAcum <- na.locf(negNAcum, na.rm = FALSE)
  negNAcum[is.na(negNAcum)] <- 0
  negStreak <- negCum - negNAcum

  streak <- posStreak + negStreak
  streak <- xts(streak, order.by = index(priceSeries))
  return (streak)
}

sigAND <- function(label, data=mktdata,
  columns, cross = FALSE) {
  ret_sig = NULL
  colNums <- rep(0, length(columns))
  for(i in 1:length(columns)) {
    colNums[i] <- match.names(columns[i], colnames(data))
  }
  ret_sig <- data[, colNums[1]]
  for(i in 2:length(colNums)) {
    ret_sig <- ret_sig & data[, colNums[i]]
  }
  ret_sig <- ret_sig * 1
  if (isTRUE(cross))
    ret_sig <- diff(ret_sig) == 1
  colnames(ret_sig) <- label
  return(ret_sig)
}

cumCRSI <- function(price, nCum = 2, ...) {
  CRSI <- connorsRSI(price, ...)
  out <- runSum(CRSI, nCum)
  colnames(out) <- "cumCRSI"
  out
}
```

### 代码块 018

```r
rm(list = ls(.blotter), envir = .blotter)
initDate = '1990-01-01'
from = "2003-01-01"
to = "2013-12-31"
initEq = 10000
currency('USD')
Sys.setenv(TZ="UTC")
source("demoData.R")
strategy.st <- "CRSIcumStrat"
portfolio.st <- "CRSIcumStrat"
account.st <- "CRSIcumStrat"
rm.strat(portfolio.st)
rm.strat(strategy.st)

initPortf(portfolio.st, symbols = symbols,
  initDate = initDate, currency = 'USD')

initAcct(account.st, portfolios = portfolio.st,
  initDate = initDate, currency = 'USD',
  initEq = initEq)

initOrders(portfolio.st, initDate = initDate)
strategy(strategy.st, store = TRUE)

# 参数
cumThresh <- 40
exitThresh <- 75
nCum <- 2
nRSI <- 3
nStreak <- 2
nPercentLookBack <- 100
nSMA <- 200
pctATR <- .02
period <- 10

# 指标
add.indicator(strategy.st, name = "cumCRSI",
  arguments = list(price = quote(Cl(mktdata)), nCum = nCum,
  nRSI = nRSI, nStreak = nStreak,
  nPercentLookBack = nPercentLookBack),
  label = "CRSIcum")

add.indicator(strategy.st, name = "connorsRSI",
  arguments = list(price = quote(Cl(mktdata)), nRSI = nRSI,
  nStreak = nStreak,
  nPercentLookBack = nPercentLookBack),
  label = "CRSI")

add.indicator(strategy.st, name = "SMA",
  arguments = list(x = quote(Cl(mktdata)), n = nSMA),
  label = "sma")

add.indicator(strategy.st, name = "lagATR",
  arguments = list(HLC = quote(HLC(mktdata)), n = period),
  label = "atrX")

# 信号
add.signal(strategy.st, name = "sigThreshold",
  arguments = list(column = "cumCRSI.CRSIcum",
  threshold = cumThresh, relationship = "lt", cross = FALSE),
  label="cumCRSI.lt.thresh")

add.signal(strategy.st, name = "sigComparison",
  arguments = list(columns = c("Close", "SMA.sma"),
  relationship = "gt"), label = "Cl.gt.SMA")

add.signal(strategy.st, name = "sigAND",
  arguments = list(columns = c("cumCRSI.lt.thresh",
  "Cl.gt.SMA"), cross = TRUE), label = "longEntry")

add.signal(strategy.st, name = "sigThreshold",
  arguments = list(column = "connorsRSI.CRSI",
  threshold = exitThresh, relationship = "gt",
  cross = TRUE), label = "longExit")

# 规则
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "longEntry", sigval = TRUE,
  ordertype = "market", orderside ="long", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR, tradeSize = tradeSize,
  pctATR = pctATR, atrMod = "X"), type = "enter", path.dep
  = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "longExit", sigval = TRUE,
  orderqty = "all", ordertype = "market", orderside = "long",
  replace = FALSE, prefer = "Open"), type = "exit", path.dep
  = TRUE)

# 应用策略
  t1 <- Sys.time()
  out <- applyStrategy(strategy = strategy.st,
  portfolios = portfolio.st)
  t2 <- Sys.time()
  print(t2 - t1)

# 建立分析
updatePortf(portfolio.st)
dateRange <- time(getPortfolio(portfolio.st)$summary)[-1]
updateAcct(portfolio.st,dateRange)
updateEndEq(account.st)
```

## 评估均值回归策略

### 代码块 019

```r
aggPF <- sum(tStats$Gross.Profits)/-sum(tStats$Gross.Losses)
  ## [1] 1.699368

aggCorrect <- mean(tStats$Percent.Positive)
  ## [1] 71.608

numTrades <- sum(tStats$Num.Trades)
  ## [1] 1500

meanAvgWLR <- mean(tStats$Avg.WinLoss.Ratio[
      tStats$Avg.WinLoss.Ratio < Inf], na.rm = TRUE)
  ## [1] 0.725
```

### 代码块 020

```r
dStats <- dailyStats(Portfolios = portfolio.st, use = "Equity")
rownames(dStats) <- gsub(".DailyEndEq", "", rownames(dStats))
print(data.frame(t(dStats)))
```

### 代码块 021

```text
##                  XLU XLV XLY
## Total.Net.Profit 2822.01 3844.99 4081.33
## Total.Days 384.00 314.00 344.00
## Winning.Days 206.00 168.00 191.00
## Losing.Days 178.00 146.00 153.00
## Avg.Day.PL 7.35 12.25 11.86
## Med.Day.PL 15.41 13.00 18.76
## Largest.Winner 725.62 690.22 632.56
## Largest.Loser -950.62 -821.22 -782.98
## Gross.Profits 23038.84 21038.16 24586.03
## Gross.Losses -20216.82 -17193.17 -20504.71
## Std.Dev.Daily.PL 155.96 166.35 178.05
## Percent.Positive 53.65 53.50 55.52
## Percent.Negative 46.35 46.50 44.48
## Profit.Factor 1.14 1.22 1.20
## Avg.Win.Day 111.84 125.23 128.72
## Med.Win.Day 92.74 95.73 102.38
## Avg.Losing.Day -113.58 -117.76 -134.02
## Med.Losing.Day -79.09 -79.56 -85.41
## Avg.Daily.PL 7.35 12.25 11.86
## Med.Daily.PL 15.41 13.00 18.76
## Std.Dev.Daily.PL.1 155.96 166.35 178.05
## Ann.Sharpe 0.75 1.17 1.06
## Max.Drawdown -1845.00 -2201.70 -2248.18
## Profit.To.Max.Draw 1.53 1.75 1.82
## Avg.WinLoss.Ratio 0.98 1.06 0.96
## Med.WinLoss.Ratio 1.17 1.20 1.20
## Max.Equity 4344.64 3892.77 4081.33
## Min.Equity -729.04 -1228.57 -631.80
## End.Equity 2822.01 3844.99 4081.33
```

### 代码块 022

```r
durationStatistics <- function(Portfolio, Symbols,
  includeOpenTrade = FALSE, ...) {

  tmp <- list()
  length(tmp) <- length(Symbols)
  for(Symbol in Symbols) {
  pts <- perTradeStats(Portfolio = Portfolio,
   Symbol = Symbol, includeOpenTrade = includeOpenTrade)
  pts$diff <- pts$End - pts$Start

durationSummary <- summary(as.numeric(pts$diff))
winDurationSummary <- summary(as.numeric(
  pts$diff[pts$Net.Trading.PL > 0]))
lossDurationSummary <- summary(as.numeric(
  pts$diff[pts$Net.Trading.PL <= 0]))
names(durationSummary) <-
  c("Min", "Q1", "Med", "Mean", "Q3", "Max")
names(winDurationSummary) <-
  c("Min", "Q1", "Med", "Mean", "Q3", "Max")
names(lossDurationSummary) <-
  c("Min", "Q1", "Med", "Mean", "Q3", "Max")
names(winDurationSummary) <-
  paste0("W", names(winDurationSummary))
names(lossDurationSummary) <-
  paste0("L", names(lossDurationSummary))

  dataRow <- data.frame(cbind(t(round(durationSummary)),
    t(round(winDurationSummary)),
    t(round(lossDurationSummary))))
   tmp[[Symbol]] <- dataRow
 }
 out <- do.call(rbind, tmp)
 return(out)
}
```

### 代码块 023

```r
durStats <- durationStatistics(Portfolio=portfolio.st,
  Symbols=sort(symbols))
print(t(durStats))
```

### 代码块 024

```text
##      XLU XLV XLY
##  Min 1   1   1
##  Q1  5   4   3
##  Med 8   6   6
##  Mean    10  8   7
##  Q3  14  10  10
##  Max 27  33  22
##  WMin    1   1   1
##  WQ1 2   3   2
##  WMed    6   5   5
##  WMean   6   6   6
##  WQ3 8   7   7
##  WMax    21  20  20
##  LMin    6   6   1
##  LQ1 12  9   8
##  LMed    16  16  12
##  LMean   16  15  11
##  LQ3 20  18  14
##  LMax    27  33  22
```

### 代码块 025

```r
# 市场风险
tmp <- list()
length(tmp) <- length(symbols)
for(i in 1:nrow(dStats)) {
  totalDays <- nrow(get(rownames(dStats)[i]))
  mktExposure <- dStats$Total.Days[i] / totalDays
  tmp[[i]] <- c(rownames(dStats)[i], round(mktExposure, 3))
}
mktExposure <- data.frame(do.call(rbind, tmp))
colnames(mktExposure) <- c("Symbol", "MktExposure")
```

### 代码块 026

```r
print(mktExposure)
##   Symbol MktExposure
## 1 EFA 0.113
## 2 EPP 0.122
## 3 EWA 0.137
## 4 EWC 0.137
## 5 EWG 0.126
## 6 EWH 0.136
## 7 EWJ 0.073
## 8 EWS 0.097
## 9 EWT 0.135
## 10 EWU 0.103
## 11 EWY 0.116
## 12 EWZ 0.108
## 13 EZU 0.123
## 14 IEF 0.155
## 15 IGE 0.144
## 16 IYR 0.127
## 17 IYZ 0.169
## 18 LQD 0.147
## 19 RWR 0.133
## 20 SHY 0.114
## 21 TLT 0.146
## 22 XLB 0.131
## 23 XLE 0.132
## 24 XLF 0.104
## 25 XLI 0.121
## 26 XLK 0.121
## 27 XLP 0.125
## 28 XLU 0.139
## 29 XLV 0.113
## 30 XLY 0.124

print(mean(as.numeric(as.character(mktExposure$MktExposure))))
## [1] 0.1257
```

### 代码块 027

```text
SharpeRatio.annualized(portfRets)
## [1] 0.6973019

Return.annualized(portfRets)
## [1] 0.03370045

maxDrawdown(portfRets)
## [1] 0.09120687
```

### 代码块 028

```r
  chart.Posn(portfolio.st, "XLB")
  TA_CRSI <- connorsRSI(Cl(XLB), nRSI = nRSI,
     nStreak = nStreak, nPercentLookBack = nPercentLookBack)
  add_TA(TA_CRSI, col = "red")

  TA_cumCRSI <- cumCRSI(price = Cl(XLB),
     nCum = nCum, nRSI = nRSI, nStreak = nStreak,
     nPercentLookBack = nPercentLookBack)
  add_TA(TA_cumCRSI, col = "blue")
 
  TA_lagATR <- lagATR(HLC = HLC(XLB), n = period)
  add_TA(TA_lagATR, col = "purple")

  TA_SMA <- SMA(Cl(XLB), n = nSMA)
  add_TA(TA_SMA, col = "blue", lwd = 2, on = 1)
```
