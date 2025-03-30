布林带与无套利区间结合下的ETF期现套利策略
## 一、策略理论
## 1. 股指期货与交易型开放式指数基金ETF
ETF 是一种在证券交易所上市交易的基金，通常跟踪某个指数、行业或资产类别。股指期货是以股票指数为标的的标准化期货合约，买卖双方约定在未来某一时间按约定价格交割。在ETF期现套利模型中，我们将ETF基金来作为现货。与股票相比，其优势在于流动性强，不存在当天无法交易等特殊情况的风险。
## 2. 简单对冲
首先我们需要讨论在理论条件下，对冲时的股指期货与ETF的交易数量，在这里不进行复杂的推导。以沪深300股指期货（IF）为例，申明$`IFprice`$为股指期货的价格，$`ETFprice`$为ETF基金的价格。假设当前指数价格为$`P`$，ETF价格与指数点位的乘数关系为$`u`$，则每单位ETF的价值为$`\frac{P}{u}  `$，股指期货合约乘数为$`k`$，则每单位IF的市值为$`P \times k`$。对冲的核心是使1单位股指期货的市值变动与n单位ETF的市值变动完全抵消，则有$`n \times \frac{P}{u} = P \times k`$，可以得到每对冲1单位的ETF，需要反向操作n单位的IF，其中：
<p align="center">$n = k \times u$</p>

## 3. 价差与布林带
股指期货与ETF的价差$`spread = IFprice -  ETFprice \times u`$。在股指期货刚上市的第一天，不进行交易，而是对价差数据进行收集。可以得到共240条的分钟频率的价差数据。然后求得价差$`spread`$的平均值$`avg`$与标准差$`std`$，那么上市第二天的第一分钟的布林带中轨$`BOLL_{mid} = avg`$，布林带上轨$`BOLL_{up} = avg + std`$，布林带下轨为$`BOLL_{low} = avg - std`$。此后，将昨日第一个价差数据剔除，加入当日新一分钟的价差数据，并重新对布林带进行计算，这就是布林带滚动数据。
布林带可以用来刻画数据的变动幅度，价差的波动大多数位于布林带之间。因此，当价差大于布林带上轨时，布林带判断此时价格出现异常，存在套利空间，进行正向套利，即卖空IF，同时买入同等价值的ETF进行对冲。待到价差回归到布林带上轨时，进行平仓，结束正向套利。同理，当价差小于布林带下轨时，可以进行反向套利，即买入股指期货，同时卖空同等价值的ETF进行对冲。待到价差回归到布林带下轨时，进行平仓，结束反向套利。
由于我国对卖空股票的限制很多，且回测平台暂不支持ETF的卖空，因此代码中只进行正向套利。
## 4. 手续费与无套利区间
在高频交易下，价格变动频繁，且期货平今仓手续费率极高，很容易导致理论利润小于手续费支出的情况，导致最终的亏损。因此，要对理论收益与手续费进行对比，如果理论收益大于手续费，则该交易是可行的。
对于IF，假设平非今仓手续费率为$`α_1`$每点，平今仓手续费率为$`α_0`$每点。ETF的交易手续费为$`β`$。注意，交易手续费在开仓和平仓时分别收取。同样采用简化模型，由于开仓与平仓时，价格变化不大，在计算手续费时可以将开仓与平仓价格近似等同。平非今仓情况下，手续费$`cost_1 = 2α_1\times IFprice + 2 k\times u\timesβ\times ETFprice`$。同理平今仓手续费为$`cost_0 =α_0\times IFprice+α_1\times IFprice + 2k\times u\times β\times ETFprice`$。
## 5. 交易时机
设计一个简单的交易策略。一般而言，平今仓的手续费要比非平金仓的手续费率要高得多，即$`α_1 < α_0`$，因此易得$`cost_1 < cost_0`$ 。交易逻辑为，当仓位为0，$`spread > up`$，且$`profit > cost_otherday`$，开仓交易。如果说$`profit > cost_0`$时，可以进行当天平仓。否则最早为明天进行平仓。当仓位不为0，$`spread < up`$时，平仓。

## 二、其他信息
### 1. 交易标的
本策略为ETF期现套利，选用的交易标的分别为沪深300股指期货（代码为IF2004），以及多支沪深300ETF基金，例如广发沪深300ETF、博时沪深300ETF、华夏沪深300ETF等。代码中以南方沪深300ETF为例（代码为159925.XSHE）。
### 2. 回测时间
为了研究意义更加明确，对股指期货的整个过程进行回测。以IF2004为例，其上市日期为2020-02-24，交割日期为2020-04-17，选定为回测区间
### 3. 数据来源
可从米筐平台直接调用，数据频次为日频

## 三、回测结果
本文尝试了多个股指期货与相应ETF的投资组合，其中部分回测结果如下：
### 1. 南方沪深300ETF（159925.XSHE），IF2004
<img src="https://github.com/user-attachments/assets/a3f36161-a90b-41e1-9b1d-ac9d1fb226f6" width="200" />
回测期间内，一共产生了27次交易信号（具体记录可见源码），回测收益为8.5%，年画收益为71.7%。套利策略的特点是风险低，可以看到策略的回撤较小，收益比较稳定。
### 2. 广发沪深300ETF（159925.XSHE），IF2004
![image](https://github.com/user-attachments/assets/e9da3de0-2cfe-4509-96f4-d15cb754a0ce)
### 3. 华夏沪深300ETF（510330.XSHG），IF2004
![image](https://github.com/user-attachments/assets/d3a0c119-02f7-44bb-99e6-bada1ae611ce)
### 4. 嘉实沪深300ETF（159919.XSHE），IF2004
![image](https://github.com/user-attachments/assets/18af659c-3603-40bd-8939-494391516355)
### 5. 工银瑞信沪深300ETF（510350.XSHG），IF2004
![image](https://github.com/user-attachments/assets/cdf9a088-6ad9-40cf-9d77-ed4096fe9e8b)
### 6. 博时上证50ETF（510710.XSHG），IH2004
![image](https://github.com/user-attachments/assets/433bbcfa-1d65-4f20-9c35-c7a4b2ce0b6f)
### 7. 华夏上证50ETF（510050.XSHG），IH2004
![image](https://github.com/user-attachments/assets/021b49a9-9360-45bb-9dbc-e227fcc60424)
### 8. 工银瑞信上证50ETF（510850.XSHG），IH2004
![image](https://github.com/user-attachments/assets/f5145973-52b5-46b2-8448-7dca27701fcc)
### 9. 万家上证50ETF（510680.XSHG），IH2004
![image](https://github.com/user-attachments/assets/997ae39a-76e4-4268-8d20-b042fc012286)
### 10. 华夏上证50ETF（510100.XSHG），IH2004
![image](https://github.com/user-attachments/assets/f7d1c921-5cc9-48ef-b90f-c7d5cd0f53cf)
可以看到，该策略是完全可行的，在涵盖市面上大部分ETF的情况下，最低收益率仅为-0.6%（华夏沪深300ETF）。且作为套利模型，在风险极低，回撤极低的情况下，部分组合还出现了16.4%的收益率（万家上证50ETF）
![image](https://github.com/user-attachments/assets/d4d0acc9-5d9c-4263-ad01-5c2f1ffc12c3)
## 四、改进
对冲交易时考虑滑点等复杂模型，加入更完善的信号机制等

