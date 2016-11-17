# 整合在一起

接下来，我们就把我们之前做的整合在一起，看看如何进行国内股票的回测 （注意，下面的软件包中部分依赖的内容并未发布到github上，下面的说明只是示意，如果要让系统运行起来，需要自行开发部分代码，主要是数据获取部分的代码）

首先，安装`cn-stock-holidays` `cn-treasury-curve` `zipline-cn-databundle` 等软件包

## ingest 过程



配置 `~/.zipline/extension.py`注册bundle

```python
from zipline.data.bundles import register
from zipline_cn_databundle.squant_source import squant_bundle
import pandas as pd
from cn_stock_holidays.zipline.default_calendar import shsz_calendar

register('cn_squant',
        squant_bundle,
        'SHSZ',
        pd.Timestamp('2008-12-19', tz='utc'),
        pd.Timestamp('2016-10-31', tz='utc')
        )
```

由于我的程序里面使用了通达信的数据，在通达信的客户端下载数据，复制通达信目录下`vipdoc`目录下 `sh` 和 `sz `目录到某个位置，并在环境变量里设置，adjustment和assets meta 信息，由于使用了公司的私有数据库，无法对外公开，所以大家自行解决吧。

运行ingest

```python
zipline ingest cn_squant
```

## 运行策略

由于zipline默认的运行过程没有办法支持自定义的loader ，所以这里我们自己来实现 `TradingAlgorithm`的`run` 过程。

```python
from zipline.data.bundles import register
from zipline_cn_databundle.squant_source import squant_bundle
import pandas as pd
import os

from zipline.api import (
    schedule_function,
    symbol,
    order_target_percent,
    date_rules,
    record
)
import re
from zipline.algorithm import TradingAlgorithm
from zipline.finance.trading import TradingEnvironment
from zipline.utils.calendars import get_calendar, register_calendar
from zipline.finance import trading
from zipline.utils.factory import create_simulation_parameters
from zipline.data.bundles.core import load
from zipline.data.data_portal import DataPortal

from zipline_cn_databundle.loader import load_market_data

# register SHSZ

from cn_stock_holidays.zipline.default_calendar import shsz_calendar


bundle = 'cn_squant'

start_session_str = '2011-01-05'

register(
        bundle,
        squant_bundle,
        "SHSZ",
        pd.Timestamp(start_session_str, tz='utc'),
        pd.Timestamp('2016-10-31', tz='utc')
        )


bundle_data = load(
    bundle,
    os.environ,
    None,
)

prefix, connstr = re.split(
    r'sqlite:///',
    str(bundle_data.asset_finder.engine.url),
    maxsplit=1,
)

env = trading.environment = TradingEnvironment(asset_db_path=connstr,
                                               trading_calendar=shsz_calendar,
                                               bm_symbol='000001.SS',
                                               load=load_market_data)


first_trading_day = \
    bundle_data.equity_minute_bar_reader.first_trading_day
data = DataPortal(
    env.asset_finder, shsz_calendar,
    first_trading_day=first_trading_day,
    equity_minute_reader=bundle_data.equity_minute_bar_reader,
    equity_daily_reader=bundle_data.equity_daily_bar_reader,
    adjustment_reader=bundle_data.adjustment_reader,
)


def initialize(context):
    schedule_function(handle_daily_data, date_rules.every_day())

def handle_daily_data(context, data):
    sym = symbol('000001.SZ')

    # 计算均线
    short_mavg = data.history(sym, 'close', 5, '1d').mean()
    long_mavg = data.history(sym, 'close', 10, '1d').mean()

    # 交易逻辑
    if short_mavg > long_mavg:
        # 满仓
        order_target_percent(sym, 1)
    elif short_mavg < long_mavg:
        # 清仓
        order_target_percent(sym, 0)

    # Save values for later inspection
    record(价格=data.current(sym, 'price'),
           short_mavg=short_mavg,
           long_mavg=long_mavg)

if __name__ == '__main__':
    sim_params = create_simulation_parameters(
        start=pd.to_datetime(start_session_str + " 00:00:00").tz_localize("Asia/Shanghai"),
        end=pd.to_datetime("2012-01-01 00:00:00").tz_localize("Asia/Shanghai"),
        data_frequency="daily", emission_rate="daily", trading_calendar=shsz_calendar)

    algor_obj = TradingAlgorithm(initialize=initialize,
                                 handle_data=None,
                                 sim_params=sim_params,
                                 env=trading.environment,
                                 trading_calendar=shsz_calendar)
    # not use run method of TradingAlgorithm
    #perf_manual = algor_obj.run(data)
    #perf_manual.to_pickle('/tmp/perf.pickle')

    algor_obj.data_portal = data
    algor_obj._assets_from_source = \
            algor_obj.trading_environment.asset_finder.retrieve_all(
                    algor_obj.trading_environment.asset_finder.sids
                    )
    algor_obj.perf_tracker = None
    try:
        perfs = []
        for perf in algor_obj.get_generator():
                perfs.append(perf)
        daily_stats.to_pickle('/tmp/perf.pickle')
    finally:
        algor_obj.data_portal = None
```



其中 `initialize` 和 `handle_daily_data` 是我们策略的主要代码，我们将运行结果保存在 `perf.pickle` 文件中，后续如果要进一步进行分析，可以直接载入到Dataframe中进行分析，这里的输出结果和zipline 的 `-o`选项输出的内容是一致的。

至此，一个相对完整的运行使用zipline运行国内市场数据的过程也完成了，同事估计大家也能对`zipline`有一个简单的了解，后面有时间的话，我会完善`zipline-cn-databundle`包，尽量使用国内公开数据，是大家可以直接使用它进行回测。