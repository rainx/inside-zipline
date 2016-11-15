# zipline应用在国内市场的限制

zipline可以很好的支持美国股票市场的应用，但是却无法直接使用在国内市场，主要有如下几个方面的限制	

## 数据方面

zipline 自带的几个bundle 都无法支持国内市场，其中quandl只包含美国的市场数据， yahoo的经过配置勉强可以下载到国内的股票信息。但是由于雅虎的国内行情的数据质量不敢恭维，历史数据经常会有缺失，所以感觉也不是特别靠谱的选择

## 交易日历 ( TradingCalendar )

交易日历是zipline系统里非常重要的部分，通过上一文章的稿子可以看到，很多其他的组件都和它有关联，给其他组件提供时间维度的索引，不管是ingest data bundle 还是运行算法，如果日历不匹配的话，那回测的结果肯定是不正确的，zipline默认使用的是美国纽交所的交易日历 （NYSE），这个显然是不能匹配国内股市的，除此之外，它还提供了 CME ([芝加哥商品交易所 ](http://www.baidu.com/link?url=Ysd27unYdDJZcKSxnEKAt8GjrM5XgNR_ZQ_yR5qWUQSQ0rdB-50pEQjn9ZJ63wgm)) ,ICE（洲际交易所），us_futures （美国期货），等不同交易市场的交易日历，同样也不太适合国内的市场。

## 基准数据和国债利率曲线

### 基准数据

在回测的时候，如果没有特别指定，zipline使用的美国的`标普500`作为基准，显然也无法适用于国内的市场，我们可以通过两种方式来改变默认的行为

* 一种是在初始化`TradingEnvironment`的时候使用`bm_symbol`参数来指定
* 另外一种是在策略代码里调用 `set_benchmark` api 来设置

注意，两种方法在数据源的获取上有所不同 ， `TradingEnvironment`的方式通过雅虎网站上的csv数据源获取的，而 `set_benchmark` 的方式是直接使用你本地的data bundle数据。

无论哪种方式，现有的zipline平台都不是特别合适，其中`TradingEnvironment`的方式由于是用yahoo上获取的数据源，经测试，雅虎的数据源的数据在国内的指数上，如`上证综指`部分时间点的数据有缺失，由于zipline在运行时对于数据的要求比较严格，会导致运行时抛出异常失效。对于`set_benchmark`的模式，由于它默认是不提供国内的行情数据的，所以也无法使用。

所以如果要讲zipline应用到国内的市场，需要做一些定制的开发。