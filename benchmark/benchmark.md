# 基准信息和国债收益率曲线

基准信息和国债收益率曲线用于计算Alpha，Beta，Shape, Sortino等风险指标的时候使用。默认情况下zipline使用`标普500`作为基准数据，并且使用美国的国债收益率曲线，这显然是有问题的，基于这个信息计算出来的数值也会存在较大偏差，所以我们需要引入国内的数据。

## 基准信息(benchmark)

之前我们提到，切换benchmark信息有两种方式：

- 一种是在初始化`TradingEnvironment`的时候使用`bm_symbol`参数来指定
- 另外一种是在策略代码里调用 `set_benchmark` api 来设置

对于方式2，如果你需要引入某个指数的数据作为基准， 需要我们再ingest信息的时候讲指数的信息也作为行情导入进来，如果是方式1，那么需要注意的是，zipline默认的loader只能通过雅虎网站上下载指数的行情信息，而我们之前也提到过，**雅虎的行情并不靠谱** (虽然国内的行情有一阵子是我再Yahoo China的时候维护的 T_T)，所以如果需要用这种方式导入，需要**实现我们自己的** loader并在`TradingEnvionment`初始化的时候传递进去。

下面就是一个例子

https://github.com/rainx/zipline_cn_databundle/blob/master/zipline_cn_databundle/loader.py

对于可以使用的指数信息，我维护了一个项目来生成这部分数据，这个项目会保持每个交易日晚上更新

https://github.com/rainx/cn_index_benchmark_for_zipline

## 国债收益率曲线

国库收益率曲线也可以通过loader进行定制，具体可以参考之前loader的代码，我这里通过从 [中债信息网](http://www.chinabond.com.cn/)获取数据，并进行整合，由于网站上的数据是通过excel的文件提供的，并且格式和最终zipline所需要的格式并不相同，我这里做了一个解析其内容的包：

https://github.com/rainx/cn_treasury_curve

关于数据的推导和整理过程，可以参考**[这个jupyter notebook文档](https://github.com/rainx/cn_treasury_curve/blob/master/notebooks/%E4%B8%AD%E5%80%BA%E6%95%B0%E6%8D%AE%E6%95%B4%E7%90%86.ipynb)**

这部分内容，也同样通过上面的loader整合到zipline中。

