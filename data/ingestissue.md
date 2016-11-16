# zipline ingest的时候遇到的坑

这个是之前做程序的时候随手记录的，本来不想写出来，但是想到也许有人也遇到类似的问题，就记一下，99.999%的人应该可以略过了。



在dailywriter 写入的时候 传入的 sid, data tuple 里的 data 是一个df ，这个df 必须是有唯一类型的， 不能是组合类型，否则ingest 的时候会报如下错误，原因是 winsorise_uint32 会将所有大于uint32的内容过滤掉，其中用到 df[boolean_df] = 0 这个不允许用混合类型的df



```python
Traceback (most recent call last):
  File "/usr/local/bin/zipline", line 11, in <module>
    load_entry_point('zipline==1.0.2+103.g9d7049a.dirty', 'console_scripts', 'zipline')()
  File "/usr/local/lib/python3.4/site-packages/click/core.py", line 716, in __call__
    return self.main(*args, **kwargs)
  File "/usr/local/lib/python3.4/site-packages/click/core.py", line 696, in main
    rv = self.invoke(ctx)
  File "/usr/local/lib/python3.4/site-packages/click/core.py", line 1060, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "/usr/local/lib/python3.4/site-packages/click/core.py", line 889, in invoke
    return ctx.invoke(self.callback, **ctx.params)
  File "/usr/local/lib/python3.4/site-packages/click/core.py", line 534, in invoke
    return callback(*args, **kwargs)
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/__main__.py", line 306, in ingest
    show_progress,
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/bundles/core.py", line 451, in ingest
    pth.data_path([name, timestr], environ=environ),
  File "/usr/local/lib/python3.4/site-packages/zipline_cn_databundle-0.2-py3.4.egg/zipline_cn_databundle/squant_source.py", line 108, in squant_bundle
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 265, in write
    return self._write_internal(it, assets)
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 327, in _write_internal
    for asset_id, table in iterator:
  File "/usr/local/lib/python3.4/site-packages/click/_termui_impl.py", line 259, in next
    rv = next(self.iter)
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 258, in <genexpr>
    ((sid, to_ctable(df, invalid_data_behavior)) for sid, df in data),
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 166, in to_ctable
    # we already have a ctable so do nothing
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 169, in to_ctable
    winsorise_uint32(raw_data, invalid_data_behavior, 'volume', *OHLC)
  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 117, in winsorise_uint32

  File "/usr/local/lib/python3.4/site-packages/zipline-1.0.2+103.g9d7049a.dirty-py3.4-macosx-10.10-x86_64.egg/zipline/data/us_equity_pricing.py", line 159, in winsorise_uint32
    df[mask] = 0
  File "/usr/local/lib/python3.4/site-packages/pandas/core/frame.py", line 2354, in __setitem__
    self._setitem_frame(key, value)
  File "/usr/local/lib/python3.4/site-packages/pandas/core/frame.py", line 2390, in _setitem_frame
    self._check_inplace_setting(value)
  File "/usr/local/lib/python3.4/site-packages/pandas/core/generic.py", line 2781, in _check_inplace_setting
    raise TypeError('Cannot do inplace boolean setting on '
TypeError: Cannot do inplace boolean setting on mixed-types with a non np.nan value
```

部分数据缺失问题

如果信息在`tradingcalendar`有效地日期没有数据，则`ingest`之后会造成错乱

如 深发展 `000001.sz`

在通达信的数据中，查询

```
2010-12-16  16.62  16.72  16.43  16.45  22055606

2010-12-17  16.45  16.50  16.29  16.35  17302092

2010-12-20  16.39  16.44  15.99  16.13  25620545

2010-12-21  16.17  16.65  16.07  16.57  30685363

2010-12-22  16.61  16.63  16.20  16.23  22703588

2010-12-24  16.20  16.60  16.15  16.44  28612724

2010-12-27  16.62  16.73  16.02  16.07  30865798

```

无法发现 12-23 日的数据，

导致使用zipline的BcolzReader的时候，日期会错乱

```python
In [32]: reader2.get_value(1202, '2010-12-21', 'close')
Out[32]: 16.57

In [33]: for i in range(1,9):
    ...:     date = '2010-12-2' + str(i)
    ...:     v = (date, reader2.get_value(1202, date, 'close'))
    ...:     print(v)
    ...:
('2010-12-21', 16.57)
('2010-12-22', 16.23)
('2010-12-23', 16.44)
('2010-12-24', 16.07)
```

在网易上查看，也无此信息

[http://quotes.money.163.com/trade/lsjysj_000001.html?year=2010&season=4](http://quotes.money.163.com/trade/lsjysj_000001.html?year=2010&season=4)

奇怪的是，雅虎有对应的数据，可能是前向或者后向填充的数据

Bcolz Data的meta信息内容

```python
{
  "first_trading_day":1292889600,
  "end_session_ns":1477872000000000000,
  "calendar_name":"SHSZ",
  "start_session_ns":1292889600000000000,
  "first_row":{},
  "calendar_offset":{},
  "last_row":{}
}
```

其中  `calendar_name` 对应日历的名字

`first_row`` last_row` , `calendar_offset` 都是 以字符串格式定义的sid为key的字典

`table`里面是针对每个column每个股票前后连接起来的一个结构

`first_row` 定义了该sid对应股票在table里的起始位置

`last_row` 定义了该sid对应股票在table里的起始结束位置

`calendar_offset` 是股票起始数据日期对于当前 `start_session_ns` 的偏移量

这里有一个坑是first_row和last_row之间的日期不能有空的数据，及时股票停牌，也要填充上数据，否则数据索引的时候会出错

研究zipline的处理方式

在quandl模块里

```python
sessions = calendar.sessions_in_range(start_session, end_session)

raw_data = raw_data.reindex(
    sessions.tz_localize(None),
    copy=False,
).fillna(0.0)
```

reindex 并fillna(0.0)

修改之后，调用 

```python
In [151]: reader2.get_value(1202, '2010-12-23', 'close')
Out[151]: -1
```

查看历史信息

```python
def handle_daily_data(context, data):
    sym = symbol('000001.SZ')
    # Save values for later inspection
    record(price=data.current(sym, 'price'),)

In [23]: df.price
Out[23]:
2010-12-21 07:00:00    16.57
2010-12-22 07:00:00    16.23
2010-12-23 07:00:00    16.23
2010-12-24 07:00:00    16.44
2010-12-27 07:00:00    16.07
2010-12-28 07:00:00    15.95
2010-12-29 07:00:00    15.77
2010-12-30 07:00:00    15.67
2010-12-31 07:00:00    15.79
Name: price, dtype: float64
```

