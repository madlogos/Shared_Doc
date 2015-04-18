# 沪牌拍卖价格分析
madlogos  
Friday, April 17, 2015  



# 分析说明 Intro

沪牌拍卖要拼人品。最低成交价往往出现在最后30秒内。但偶尔也有例外，最低成交价出现在较早的时段，这意味着有人为干预。这就为预估价格、提前伏击创造了机会。

数据来自网络，包括每月放牌数(`Plate`)、参拍人数(`Bidder`)、最低成交价(`MinPrice`)和平均价(`AvgPrice`)。

用`Rstudio 0.98.1103`的`rmarkdown`完成(`R 3.2RC`)。代码隐藏未显示。


```r
wb <- XLConnect::loadWorkbook(
    "C:/Users/madlogos/SkyDrive/协作文件夹/House/沪牌历史价格.xlsx")
d <- XLConnect::readWorksheet(wb,"Sheet1")
d$Year <- as.numeric(format(d$Date,"%Y"))
d$Month <- as.numeric(format(d$Date,"%m"))
d$Rate <- d$Plate/d$Bidder
d <- d[1:(nrow(d)-1),]
```

# 基本情况 Basics

从2002年开始，中签率周期波动。2013年开始，急剧下滑。

## 中签率趋势 Plate vs Bid


```r
gd<-melt(d[,c("Date","Plate","Bidder")],id="Date")
names(gd) <- c("Date","Type","Number")
g <- ggplot(gd,aes(x=Date,y=Number,group=Type))+
    geom_line(aes(color=Type))+
    ggtitle("Plate vs Bidder")+theme_bw()
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/basic_chart-1.png) 

```r
cat("\n\n")
```

```r
g <- ggplot(d,aes(x=Date,y=Rate))+
    geom_line(color="darkblue")+
    ggtitle("Bid Success Rate")+theme_bw()
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/basic_chart-2.png) 

## 均价趋势 Avg Price Trend
2002年至今合计，按月平均，均价波动不大。


```r
g <-ggplot(d,aes(x=Month,y=AvgPrice))+stat_smooth(method="lm")+
    geom_point()+theme_bw()+ggtitle("Avg Price by Month")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/monthly_avg_price-1.png) 

但平均成交价一直在波动升高。每年的成交均价几乎是线性升高的。


```r
g <- ggplot(d,aes(Date,AvgPrice))+geom_line(color="darkblue")+
    theme_bw()+ggtitle("Avg Price Trend")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/avg_price_trend-1.png) 

```r
cat("\n\n")
```

```r
g <-ggplot(d,aes(x=Year,y=AvgPrice))+stat_smooth(method="lm")+
    geom_point()+theme_bw()
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/avg_price_trend-2.png) 

历史上，6-7月是一个低谷，长假是高峰。这个趋势在2014年以后不见了。
而拍牌人数在春季升高，夏季到顶，秋季会回落。但中签率近几年始终很低。这些规律对拍牌命中帮助不大(-_-)。


```r
d$Yr <- as.factor(d$Year)
g <-ggplot(d,aes(x=Month,y=AvgPrice,group=Yr))+theme_bw()+
    geom_line(aes(color=Yr))+ggtitle("Year-specific Avg Price by Month")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-1.png) 

```r
cat("\n\n")
```

```r
g <-ggplot(d,aes(x=Month,y=Bidder,group=Yr))+theme_bw()+
    geom_line(aes(color=Yr))+ggtitle("Year-specific Bidder Number by Month")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-2.png) 

```r
cat("\n\n")
```

```r
g <-ggplot(d,aes(x=Month,y=Rate,group=Yr))+theme_bw()+
    geom_line(aes(color=Yr))+ggtitle("Year-specific Bid Success Rate by Month")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-3.png) 

# 线性拟合预测 Linear Model

懒得做时间序列，就直接暴力线性拟合了。用2011年以后的数据。

## 拟合4个模型 Fit 4 Models
为什么拟合4个？因为也就这么多变量了。。。


```r
ds <- subset(d,Year>=2011)
m1 <- lm(AvgPrice~PrevPrice,ds)
r <- summary(m1)
r2 <- round(r$adj.r.squared,4)
m2 <- lm(AvgPrice~PrevPrice+PrevBidder+Plate,ds)
r <- summary(m2)
r2 <- c(r2,round(r$adj.r.squared,4))
m3 <- lm(AvgPrice~PrevPrice+PrevBidder+Plate+Bidder,ds)
r <- summary(m3)
r2 <- c(r2,round(r$adj.r.squared,4))
m4 <- lm(AvgPrice~PrevPrice+PrevBidder+Plate+Bidder+PrevMinPrice,ds)
r <- summary(m4)
r2 <- c(r2,round(r$adj.r.squared,4))

r.mtx <- data.frame(
    Model=c("AvgPrice~PrevPrice",
            "AvgPrice~PrevPrice+PrevBidder+Plate",
            "AvgPrice~PrevPrice+PrevBidder+Plate+Bidder",
            "AvgPrice~PrevPrice+PrevBidder+Plate+Bidder+PrevMinPrice"),
    R.Sqaure.Adj=r2)
knitr::kable(r.mtx,format="markdown",caption="Adj $R^{2}$ of Models",
             col.names=c("Model","$R^{2} Adj$"))
```



|Model                                                   | $R^{2} Adj$|
|:-------------------------------------------------------|-----------:|
|AvgPrice~PrevPrice                                      |      0.8170|
|AvgPrice~PrevPrice+PrevBidder+Plate                     |      0.8131|
|AvgPrice~PrevPrice+PrevBidder+Plate+Bidder              |      0.8090|
|AvgPrice~PrevPrice+PrevBidder+Plate+Bidder+PrevMinPrice |      0.8116|

```r
k1 <- m1$coefficients
k2 <- m2$coefficients
k3 <- m3$coefficients
k4 <- m4$coefficients
cat("\n\n")
```

```r
tk1 <- as.data.frame(k1)
tk1$var <- row.names(tk1)
tk1$model <- 1
tk2 <- as.data.frame(k2) 
tk2$var<-row.names(tk2)
tk2$model <- 2
tk3 <- as.data.frame(k3)
tk3$var<-row.names(tk3)
tk3$model <- 3
tk4 <- as.data.frame(k4)
tk4$var<-row.names(tk4)
tk4$model <- 4
names(tk1) <- c("value","Var","Model")
names(tk2) <- c("value","Var","Model")
names(tk3) <- c("value","Var","Model")
names(tk4) <- c("value","Var","Model")
tk <- rbind(tk1,tk2,tk3,tk4)
dtk <- as.data.frame(dcast(tk,Var~Model,mean))
names(dtk)<- c("Variable","Model 1","Model 2","Model 3","Model 4")
knitr::kable(dtk,format="markdown",caption="Parameter Estimate of Model 1-4")
```



|Variable     |      Model 1|      Model 2|      Model 3|      Model 4|
|:------------|------------:|------------:|------------:|------------:|
|(Intercept)  | 1.470494e+04| 7175.3251272| 7075.6278084| 7417.0583037|
|Bidder       |          NaN|          NaN|    0.0029741|   -0.0345129|
|Plate        |          NaN|    0.9644115|    0.9768321|    0.5601606|
|PrevBidder   |          NaN|    0.0320525|    0.0292701|    0.0629747|
|PrevMinPrice |          NaN|          NaN|          NaN|   -1.2137834|
|PrevPrice    | 7.924684e-01|    0.7597233|    0.7593850|    2.0173086|

4个模型的$R^{2}$ Adj差不多。单用前期均价已经能预报了。

## 模型残差 Residuals


```r
rs1<-tail(m1$residuals,n=20)
rs2<-tail(m2$residuals,n=20)
rs3<-tail(m3$residuals,n=20)
rs4<-tail(m4$residuals,n=20)
dtrs<-data.frame(Model=c("Model 1","Model 2","Model 3","Model 4"),
                 sd.Residual=c(sd(rs1,na.rm=T),sd(rs2,na.rm=T),
                               sd(rs3,na.rm=T),sd(rs4,na.rm=T)))
knitr::kable(dtrs,format="markdown",caption="Std Dev of Residuals, Model 1-4 (last 20 records)")
```



|Model   | sd.Residual|
|:-------|-----------:|
|Model 1 |    2868.360|
|Model 2 |    2687.104|
|Model 3 |    2688.873|
|Model 4 |    2989.987|

而用最后20次拍牌结果验证，残差的方差也差不多。实践下来，还是模型4更接近一些。

## 尝试预测 Predict Attempt

本轮投标152298人，放牌8288张，中签率。估计实际均价：


```r
n <- nrow(d)
nbid <- 152298
nplate <- 8288
cat("+ 模型1：")
```

+ 模型1：

```r
forecast=as.numeric(k1[1])+d$PrevPrice[n][1] * as.numeric(k1[2])
cat("**",forecast,"**",sep="")
```

**75422.28**

```r
cat("\n\n")
```

```r
cat("+ 模型2：")
```

+ 模型2：

```r
forecast=as.numeric(k2[1])+d$PrevPrice[n][1] * as.numeric(k2[2]) +
                    d$PrevBidder[n][1] * as.numeric(k2[3])+
                    nplate * as.numeric(k2[4])
cat("**",forecast,"**",sep="")
```

**76685.44**

```r
cat("\n\n")
```

```r
cat("+ 模型3：")
```

+ 模型3：

```r
forecast=as.numeric(k3[1])+d$PrevPrice[n][1] * as.numeric(k3[2]) +
                    d$PrevBidder[n][1] * as.numeric(k3[3])+
                    nplate * as.numeric(k3[4]) + nbid * as.numeric(k3[5])
cat("**",forecast,"**",sep="")
```

**76828.5**

```r
cat("\n\n")
```

```r
cat("+ 模型4：")
```

+ 模型4：

```r
forecast=as.numeric(k4[1]) + d$PrevPrice[n][1] * as.numeric(k4[2]) + 
    d$Bidder[n][1] * as.numeric(k4[3]) + nplate * as.numeric(k4[4]) + 
    nbid * as.numeric(k4[5]) + d$MinPrice[n][1] * as.numeric(k4[6])
cat("**",forecast,"**",sep="")
```

**79173.45**

而平均价和最低价之间的差越来越小，成交区间非常窄。


```r
d$Dif<-d$AvgPrice-d$MinPrice
g <- ggplot(d,aes(Date,Dif))+geom_line(color="darkblue")+theme_bw()+
        ggtitle("Difference bewteen Avg And Min Price by Day")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/dif-1.png) 

```r
s <- as.data.frame(aggregate(Dif~Year,data=d,summary))
gd <- data.frame(Year=s[,1],Min=s[,2][,1],P25=s[,2][,2],Median=s[,2][,3],
                 Mean=s[,2][,4],P75=s[,2][,5],Max=s[,2][,6])
knitr::kable(gd,format="markdown",
             caption="Summary of Differenece between Avg And Min Price by Year")
```



| Year|   Min|   P25| Median|    Mean|    P75|   Max|
|----:|-----:|-----:|------:|-------:|------:|-----:|
| 2002|   334| 527.8|  620.5| 2948.00|  993.8| 27750|
| 2003|   745| 932.8| 1412.0| 2353.00| 1938.0|  9928|
| 2004|   333| 851.5| 1404.0| 3163.00| 1750.0| 23430|
| 2005|   384| 509.8|  693.0| 1991.00| 1768.0| 10900|
| 2006|   252| 437.2|  673.5| 1288.00| 1019.0|  4601|
| 2007|   323| 454.0|  514.0| 1281.00| 1375.0|  6042|
| 2008|   359| 644.5|  869.0| 2434.00| 2068.0| 15270|
| 2009|   231| 411.2|  463.0|  674.80|  719.5|  2300|
| 2010|   271| 351.5|  385.5|  936.20|  718.2|  5570|
| 2011|   208| 341.2|  432.0|  632.60|  628.0|  1935|
| 2012| -2633| 377.5|  502.5|  468.90|  650.8|  2427|
| 2013|    92| 176.5|  231.0|  396.40|  365.2|  1423|
| 2014|     1|  75.0|   91.5|   98.08|  118.0|   185|
| 2015|   118| 167.0|  216.0|  188.00|  223.0|   230|

```r
gd <- data.frame(Year=s[1],Median=s[,2][,3])
g <- ggplot(gd,aes(y=Median,x=Year))+geom_line(color="darkblue")+theme_bw()+
    ggtitle("Median Difference between Avg And Min Price by Year")
print(g)
```

![](Shanghai_Car_Plate_files/figure-html/dif-2.png) 

所以基本就是估计均价上下浮动500块进行伏击。希望真的符合实际的情况（毛）。

# 时间序列分析 Time-series Analysis
> 4月18日拍牌失败。只好再接再厉。这里更新一下算命技术，看会不会准些。//摊手


```r
cat("## 差分 Differences")
```

## 差分 Differences

```r
cat("\n\n")
```

```r
ts <- ts(d$AvgPrice,start=c(2002,1),frequency=12) #2002年1月开始
plot.ts(ts,main="Time-series of AvgPrice")
```

![](Shanghai_Car_Plate_files/figure-html/ts-1.png) 

```r
cat("\n\n")
```

```r
ts.df1 <- diff(ts,differences=1)
plot.ts(ts.df1,main="Time-series of AvgPrice, 1' Diff")
```

![](Shanghai_Car_Plate_files/figure-html/ts-2.png) 

```r
cat("\n\n")
```

```r
ts.df2 <- diff(ts,differences=2)
plot.ts(ts.df2,main="Time-series of AvgPrice, 2' Diff")
```

![](Shanghai_Car_Plate_files/figure-html/ts-3.png) 

```r
cat("\n\n\n")
```

```r
cat("二阶差分后看起来平稳一点，就它了。(实际上差分到10阶还是有奇异点，就随它去任性吧。)")
```

二阶差分后看起来平稳一点，就它了。(实际上差分到10阶还是有奇异点，就随它去任性吧。)

```r
cat("\n\n")
```

```r
cat("## 确定模型参数 ARIMA parameters")
```

## 确定模型参数 ARIMA parameters

```r
cat("\n\n")
```

```r
cat("### 自相关图ACF")
```

### 自相关图ACF

```r
cat("\n\n")
```

```r
acf.result<-acf(ts.df2,lag.max=20)
```

![](Shanghai_Car_Plate_files/figure-html/ts-4.png) 

```r
acf.dt<-data.frame(i=0:20,Lag=acf.result$lag,ACF=acf.result$acf)
knitr::kable(acf.dt,format="markdown",caption="ACF")
```



|  i|       Lag|        ACF|
|--:|---------:|----------:|
|  0| 0.0000000|  1.0000000|
|  1| 0.0833333| -0.5364979|
|  2| 0.1666667|  0.0295103|
|  3| 0.2500000| -0.0368472|
|  4| 0.3333333|  0.0423904|
|  5| 0.4166667|  0.0025149|
|  6| 0.5000000|  0.0082030|
|  7| 0.5833333| -0.0003025|
|  8| 0.6666667| -0.0454090|
|  9| 0.7500000|  0.0953208|
| 10| 0.8333333| -0.1232463|
| 11| 0.9166667|  0.1202756|
| 12| 1.0000000| -0.0859678|
| 13| 1.0833333|  0.0550577|
| 14| 1.1666667| -0.0895252|
| 15| 1.2500000|  0.1425013|
| 16| 1.3333333| -0.1079814|
| 17| 1.4166667|  0.0392051|
| 18| 1.5000000| -0.0531547|
| 19| 1.5833333|  0.0996016|
| 20| 1.6666667| -0.0772887|

```r
cat("滞后2阶后自相关值即不超过边界值。故自相关选2阶。")
```

滞后2阶后自相关值即不超过边界值。故自相关选2阶。

```r
cat("\n\n")
```

```r
cat("### 偏相关图PACF")
```

### 偏相关图PACF

```r
cat("\n\n")
```

```r
pacf.result<-pacf(ts.df2,lag.max=20)
```

![](Shanghai_Car_Plate_files/figure-html/ts-5.png) 

```r
cat("\n\n")
```

```r
pacf.dt<-data.frame(i=1:20,Lag=pacf.result$lag,PACF=pacf.result$acf)
knitr::kable(pacf.dt,format="markdown",caption="PACF")
```



|  i|       Lag|       PACF|
|--:|---------:|----------:|
|  1| 0.0833333| -0.5364979|
|  2| 0.1666667| -0.3627220|
|  3| 0.2500000| -0.3393394|
|  4| 0.3333333| -0.2797851|
|  5| 0.4166667| -0.2361004|
|  6| 0.5000000| -0.1894344|
|  7| 0.5833333| -0.1405862|
|  8| 0.6666667| -0.1926987|
|  9| 0.7500000| -0.0572385|
| 10| 0.8333333| -0.1651563|
| 11| 0.9166667| -0.0545309|
| 12| 1.0000000| -0.0857903|
| 13| 1.0833333| -0.0355571|
| 14| 1.1666667| -0.1606541|
| 15| 1.2500000| -0.0144222|
| 16| 1.3333333| -0.0526999|
| 17| 1.4166667| -0.0253542|
| 18| 1.5000000| -0.1251225|
| 19| 1.5833333|  0.0037262|
| 20| 1.6666667| -0.0566721|

```r
cat("滞后7阶后偏相关值大体不再超出边界值。（就算超过也就当它随机误差了。）故偏相关选7阶。")
```

滞后7阶后偏相关值大体不再超出边界值。（就算超过也就当它随机误差了。）故偏相关选7阶。

```r
cat("于是ARIMA(p,d,q)参数是**arima(2,2,7)**")
```

于是ARIMA(p,d,q)参数是**arima(2,2,7)**

## ARIMA模型拟合 Fit ARIMA


```r
arima<-arima(ts,order=c(2,2,7))
arima
```

```
## 
## Call:
## arima(x = ts, order = c(2, 2, 7))
## 
## Coefficients:
##           ar1      ar2      ma1     ma2      ma3     ma4     ma5     ma6
##       -0.7921  -0.9349  -0.4468  0.0893  -1.0965  0.2767  0.1514  0.1109
## s.e.   0.0523   0.0434   0.0988  0.1006   0.1027  0.1212  0.0945  0.1013
##           ma7
##       -0.0850
## s.e.   0.0887
## 
## sigma^2 estimated as 23712656:  log likelihood = -1550.72,  aic = 3121.44
```

### 预测 ARIMA Prediction


```r
forecast <- predict(arima,5)
pred <- cbind(Month=tail(d$Month,1):(tail(d$Month,1)+4),
              as.data.frame(forecast[1]),as.data.frame(forecast[2]))
names(pred) <- c("Month","Predicted Mean", "Predicted Std. Error")
knitr::kable(pred,format="markdown",caption="Predict AvgPrice using ARIMA(2,2,7)")
```



| Month| Predicted Mean| Predicted Std. Error|
|-----:|--------------:|--------------------:|
|     3|       76225.93|             4910.561|
|     4|       74907.90|             6178.273|
|     5|       76137.00|             6973.847|
|     6|       77767.21|             7436.199|
|     7|       76713.81|             7937.007|


```r
cat("\n\n")
```

```r
cat("### 检验 Test")
```

### 检验 Test

```r
cat("\n\n")
```

```r
tsdiag(arima,gof.lag=20)
```

![](Shanghai_Car_Plate_files/figure-html/arima_test-1.png) 

```r
cat("\n\n")
```

```r
cat("自相关图显示滞后1-20阶，样本自相关值均不超过显著边界；Ljung-Box检验所有p值均大于0.05。在滞后1-20阶，都没有证据表明预测误差是非零自相关的。")
```

自相关图显示滞后1-20阶，样本自相关值均不超过显著边界；Ljung-Box检验所有p值均大于0.05。在滞后1-20阶，都没有证据表明预测误差是非零自相关的。

# 结论 Summary
时间序列做下来，五月份价格是75000上下5000块。等于没说，最后成交区间根本就在上限那里。反正就是看命吧。
