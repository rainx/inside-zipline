# zipline的目录结构

下面列出了zipline主要的目录和文件结构和它的说明

```
├── ci    -  持续集成相关
├── conda - 生成conda 包先关
├── docs - 文档
│   ├── notebooks - notebook代码
│   └── source  - 教程和what’s new
├── etc - 依赖配置和一些 hook shell 脚本
├── tests - 测试代码
├── zipline - 代码主目录
│   ├── __init__.py - 集中引入包内容
│   ├── __main__.py - 主程序入口
│   ├── _protocol.pyx - current, history, can_trade 之类的一些数据操作接口的实现
│   ├── _version.py - 版本管理相关的
│   ├── algorithm.py - 策略算法的主逻辑抽象，算法最后会被实例化为TradingAlgorithm的实例或者继承它， 并且里面进行了主要的api的定义，zipine的 cli会调用它的run方法启动回测
│   ├── api.py 常用api
│   ├── api.pyi 常用api说明
│   ├── assets - 资产类抽象 里面封装了常用的资产如股票Equity，期货Funtrue, 作为Asset的子类，并且封装了其数据库操作（这里是sqlite)
│   ├── data - 数据相关，所有的数据操作封装为dataportal 
│   │   ├── __init__.py
│   │   ├── _adjustments.pyx - 除权除息等信息的读取
│   │   ├── _equities.pyx - 从bcolz里面获取行情的索引的抽象
│   │   ├── _minute_bar_internal.pyx - 分钟bar相关的索引
│   │   ├── bar_reader.py - BarReader接口定义
│   │   ├── benchmarks.py - 从雅虎获取基准数据
│   │   ├── bundles - 官方的提供的data bundle 
│   │   ├── continuous_future_reader.py 
│   │   ├── data_portal.py DataPortal定义，整合了所有的reader,writer，等，是biplane获取数据的入口，提供reader,writer数据的简单高层封装
│   │   ├── dispatch_bar_reader.py - 结合trading calendar 读取asset的bar信息
│   │   ├── history_loader.py  - asset 历史信息的获取, 包括附加复权信息
│   │   ├── loader.py - loader 封装了基准信息和国债收益率曲线
│   │   ├── minute_bars.py - 分钟线reader/writer相关的抽象
│   │   ├── resample.py - 把分钟线数据resample为日线数据
│   │   ├── session_bars.py -  SessionBarReader
│   │   ├── treasuries.py - 国债收益率曲线
│   │   ├── treasuries_can.py - 加拿大国债收益率曲线
│   │   └── us_equity_pricing.py - 主要是针对Equity的日线读取，adjustment数据读取，
│   ├── dispatch.py - 分发逻辑
│   ├── errors.py - 异常的抽象
│   ├── examples - 一些例子
│   ├── finance - 主要抽象了交易和财务相关的逻辑，这些接口大多会出现在zipline或者quantopian的代码策略代码里，可以进行import 
│   │   ├── __init__.py 
│   │   ├── asset_restrictions.py - 资产交易限制
│   │   ├── blotter.py - 账号？
│   │   ├── cancel_policy.py - 取消策略
│   │   ├── commission.py - 佣金
│   │   ├── constants.py - 一些常亮定义
│   │   ├── controls.py - 分控相关
│   │   ├── execution.py - 订单类型
│   │   ├── order.py - 订单逻辑
│   │   ├── performance - 收益
│   │   ├── risk - 风险相关
│   │   ├── slippage.py - 滑点
│   │   ├── trading.py - TradingEnvironment， SimulationParameters的抽象，如果使用自己的loader， TradingCalendar 则需要自己初始化这个对象 
│   │   └── transaction.py - Transaction - 交易的抽象
│   ├── gens - 应该是集合了大部分的generator , 主要是回测过程的generator
│   │   └── tradesimulation.py - 回测主要过程的generator
│   ├── lib - 一些主要用的的数据结构和算法
│   ├── pipeline - pipeline相关的逻辑
│   ├── protocol.py - Order Account, Portfolio, Position等的抽象
│   ├── resources - etc相关的一些资源
│   ├── sources - 基准数据源等
│   ├── test_algorithms.py - 测试策略...
│   ├── testing - 测试
│   ├── utils - 一些工具类， 其中 run_algo.py, tradingcalendar.py 相关的需要重点关注下
```



##  