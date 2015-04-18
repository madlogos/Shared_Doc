# 沪牌拍卖价格分析
madlogos  
Friday, April 17, 2015  



# 分析说明 Intro

沪牌拍卖要拼人品。最低成交价往往出现在最后30秒内。但偶尔也有例外，最低成交价出现在较早的时段，这意味着有人为干预。这就为预估价格、提前伏击创造了机会。

数据来自网络，包括每月放牌数(`Plate`)、参拍人数(`Bidder`)、最低成交价(`MinPrice`)和平均价(`AvgPrice`)。

用`Rstudio 0.98.1103`的`rmarkdown`完成(`R 3.2RC`)。代码隐藏未显示。



# 基本情况 Basics

从2002年开始，中签率周期波动。2013年开始，急剧下滑。

## 中签率趋势 Plate vs Bid

![](Shanghai_Car_Plate_files/figure-html/basic_chart-1.png) 

![](Shanghai_Car_Plate_files/figure-html/basic_chart-2.png) 

## 均价趋势 Avg Price Trend
2002年至今合计，按月平均，均价波动不大。

![](Shanghai_Car_Plate_files/figure-html/monthly_avg_price-1.png) 

但平均成交价一直在波动升高。每年的成交均价几乎是线性升高的。

![](Shanghai_Car_Plate_files/figure-html/avg_price_trend-1.png) 

![](Shanghai_Car_Plate_files/figure-html/avg_price_trend-2.png) 

历史上，6-7月是一个低谷，长假是高峰。这个趋势在2014年以后不见了。
而拍牌人数在春季升高，夏季到顶，秋季会回落。但中签率近几年始终很低。这些规律对拍牌命中帮助不大(-_-)。

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-1.png) 

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-2.png) 

![](Shanghai_Car_Plate_files/figure-html/trend_by_month-3.png) 

# 线性拟合预测 Linear Model

懒得做时间序列，就直接暴力线性拟合了。用2011年以后的数据。

## 拟合4个模型 Fit 4 Models
为什么拟合4个？因为也就这么多变量了。。。


Table: Adj $R^{2}$ of Models

Model                                                      $R^{2} Adj$
--------------------------------------------------------  ------------
AvgPrice~PrevPrice                                              0.8170
AvgPrice~PrevPrice+PrevBidder+Plate                             0.8131
AvgPrice~PrevPrice+PrevBidder+Plate+Bidder                      0.8090
AvgPrice~PrevPrice+PrevBidder+Plate+Bidder+PrevMinPrice         0.8116





Table: Parameter Estimate of Model 1-4

Variable             Model 1        Model 2        Model 3        Model 4
-------------  -------------  -------------  -------------  -------------
(Intercept)     1.470494e+04   7175.3251272   7075.6278084   7417.0583037
Bidder                   NaN            NaN      0.0029741     -0.0345129
Plate                    NaN      0.9644115      0.9768321      0.5601606
PrevBidder               NaN      0.0320525      0.0292701      0.0629747
PrevMinPrice             NaN            NaN            NaN     -1.2137834
PrevPrice       7.924684e-01      0.7597233      0.7593850      2.0173086

4个模型的$R^{2}$ Adj差不多。单用前期均价已经能预报了。

## 模型残差 Residuals


Table: Std Dev of Residuals, Model 1-4 (last 20 records)

Model      sd.Residual
--------  ------------
Model 1       2868.360
Model 2       2687.104
Model 3       2688.873
Model 4       2989.987

而用最后20次拍牌结果验证，残差的方差也差不多。实践下来，还是模型4更接近一些。

## 尝试预测 Predict Attempt

本轮投标152298人，放牌8288张，中签率。估计实际均价：

+ 模型1：**75422.28**

+ 模型2：**76685.44**

+ 模型3：**76828.5**

+ 模型4：**79173.45**

而平均价和最低价之间的差越来越小，成交区间非常窄。

![](Shanghai_Car_Plate_files/figure-html/dif-1.png) 

Table: Summary of Differenece between Avg And Min Price by Year

 Year     Min     P25   Median      Mean      P75     Max
-----  ------  ------  -------  --------  -------  ------
 2002     334   527.8    620.5   2948.00    993.8   27750
 2003     745   932.8   1412.0   2353.00   1938.0    9928
 2004     333   851.5   1404.0   3163.00   1750.0   23430
 2005     384   509.8    693.0   1991.00   1768.0   10900
 2006     252   437.2    673.5   1288.00   1019.0    4601
 2007     323   454.0    514.0   1281.00   1375.0    6042
 2008     359   644.5    869.0   2434.00   2068.0   15270
 2009     231   411.2    463.0    674.80    719.5    2300
 2010     271   351.5    385.5    936.20    718.2    5570
 2011     208   341.2    432.0    632.60    628.0    1935
 2012   -2633   377.5    502.5    468.90    650.8    2427
 2013      92   176.5    231.0    396.40    365.2    1423
 2014       1    75.0     91.5     98.08    118.0     185
 2015     118   167.0    216.0    188.00    223.0     230

![](Shanghai_Car_Plate_files/figure-html/dif-2.png) 

所以基本就是估计均价上下浮动500块进行伏击。希望真的符合实际的情况（毛）。

# 时间序列分析 Time-series Analysis
> 4月18日拍牌失败。只好再接再厉。这里更新一下算命技术，看会不会准些。//摊手

## 差分 Differences

![](Shanghai_Car_Plate_files/figure-html/ts-1.png) 

![](Shanghai_Car_Plate_files/figure-html/ts-2.png) 

![](Shanghai_Car_Plate_files/figure-html/ts-3.png) 


二阶差分后看起来平稳一点，就它了。(实际上差分到10阶还是有奇异点，就随它去任性吧。)

## 确定模型参数 ARIMA parameters

### 自相关图ACF

![](Shanghai_Car_Plate_files/figure-html/ts-4.png) 

Table: ACF

  i         Lag          ACF
---  ----------  -----------
  0   0.0000000    1.0000000
  1   0.0833333   -0.5364979
  2   0.1666667    0.0295103
  3   0.2500000   -0.0368472
  4   0.3333333    0.0423904
  5   0.4166667    0.0025149
  6   0.5000000    0.0082030
  7   0.5833333   -0.0003025
  8   0.6666667   -0.0454090
  9   0.7500000    0.0953208
 10   0.8333333   -0.1232463
 11   0.9166667    0.1202756
 12   1.0000000   -0.0859678
 13   1.0833333    0.0550577
 14   1.1666667   -0.0895252
 15   1.2500000    0.1425013
 16   1.3333333   -0.1079814
 17   1.4166667    0.0392051
 18   1.5000000   -0.0531547
 19   1.5833333    0.0996016
 20   1.6666667   -0.0772887

滞后2阶后自相关值即不超过边界值。故自相关选2阶。

### 偏相关图PACF

![](Shanghai_Car_Plate_files/figure-html/ts-5.png) 



Table: PACF

  i         Lag         PACF
---  ----------  -----------
  1   0.0833333   -0.5364979
  2   0.1666667   -0.3627220
  3   0.2500000   -0.3393394
  4   0.3333333   -0.2797851
  5   0.4166667   -0.2361004
  6   0.5000000   -0.1894344
  7   0.5833333   -0.1405862
  8   0.6666667   -0.1926987
  9   0.7500000   -0.0572385
 10   0.8333333   -0.1651563
 11   0.9166667   -0.0545309
 12   1.0000000   -0.0857903
 13   1.0833333   -0.0355571
 14   1.1666667   -0.1606541
 15   1.2500000   -0.0144222
 16   1.3333333   -0.0526999
 17   1.4166667   -0.0253542
 18   1.5000000   -0.1251225
 19   1.5833333    0.0037262
 20   1.6666667   -0.0566721

滞后7阶后偏相关值大体不再超出边界值。（就算超过也就当它随机误差了。）故偏相关选7阶。于是ARIMA(p,d,q)参数是**arima(2,2,7)**

## ARIMA模型拟合 Fit ARIMA


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


Table: Predict AvgPrice using ARIMA(2,2,7)

 Month   Predicted Mean   Predicted Std. Error
------  ---------------  ---------------------
     3         76225.93               4910.561
     4         74907.90               6178.273
     5         76137.00               6973.847
     6         77767.21               7436.199
     7         76713.81               7937.007


### 检验 Test

![](Shanghai_Car_Plate_files/figure-html/arima_test-1.png) 

自相关图显示滞后1-20阶，样本自相关值均不超过显著边界；Ljung-Box检验所有p值均大于0.05。在滞后1-20阶，都没有证据表明预测误差是非零自相关的。

# 结论 Summary
时间序列做下来，五月份价格是75000上下5000块。等于没说，最后成交区间根本就在上限那里。反正就是看命吧。
