# Python-time模块
### time_时间的访问和转换
该模块提供了各种时间相关的函数。可以使用以下函数在时间表示之间进行转换：
从|到|使用
---|:---:|:---
自纪元以来的秒数|UTC 的 struct_time|gmtime()
自纪元以来的秒数|本地时间的 struct_time|localtime()
UTC 的 struct_time|自纪元以来的秒数|calendar.timegm()
本地时间的 struct_time|自纪元以来的秒数|mktime()

##### time.time()
返回以浮点数表示的从 epoch 开始的秒数的时间值。
示例如下：
```python
>>> time.time()
1391426626.4728231
```
##### time.asctime([t])
转换由gmtime()或 localtime()所返回的表示时间的元组或struct_time为以下形式的字符串: 'Tue Jul 11 10:13:29 2000'。日期字段的长度为两个字符，如果日期只有一个数字则会以零填充，例如: 'Tue Jul  4 10:13:29 2000'
如果未提供 t，则会使用 localtime()所返回的当前时间。asctime()不会使用区域设置信息。
示例如下：
```python
>>> import time
>>> time.asctime()
'Fri Jul 19 09:08:38 2019'
```
##### time.gmtime([secs])
将以自epoch开始的秒数表示的时间转换为UTC的 struct_time，其中 dst 标志始终为零。如果未提供 secs 或为None，则使用time()所返回的当前时间。
示例如下：
```python
>>> import time
>>> time.gmtime(1592316546.3528154)
time.struct_time(tm_year=2020, tm_mon=6, tm_mday=16, tm_hour=14, tm_min=9, tm_sec=6, tm_wd
ay=1, tm_yday=168, tm_isdst=0)
```
##### time.localtime([secs])
与gmtime()相似但转换为当地时间。如果未提供secs或为None，则使用由time()返回的当前时间。当DST适用于给定时间时，dst标志设置为1。
示例如下：
```python
>>> import time
>>> time.localtime(1592316546.3528154)
time.struct_time(tm_year=2020, tm_mon=6, tm_mday=16, tm_hour=9, tm_min=9, tm_sec=6, tm_wda
y=1, tm_yday=168, tm_isdst=1)
```
##### time.mktime(t)
是localtime()的反函数。参数是struct_time或者完整的9元组（因为需要dst标志如果它是未知的则使用 -1 作为dst标志，它表示local的时间，而不是UTC。它返回一个浮点数，以便与time()兼容。
示例如下：
```python
>>> import time
>>> a = time.localtime(1592316546.3528154)
>>> time.mktime(a)
1592316546.0
```
 时间戳转换时候一秒以内的小数将被忽略

##### time.sleep(secs)
暂停执行调用线程达到给定的秒数。参数可以是浮点数，以指示更精确的睡眠时间。实际的暂停时间可能小于请求的时间，因为任何捕获的信号将在执行该信号的捕获例程后终止 sleep() 。
示例如下：
```python
>>> import time
>>> time.sleep(3)
```
##### time.strftime(format,[t])
转换一个元组或 struct_time 表示的由 gmtime() 或 localtime() 返回的时间到由 format 参数指定的字符串。如果未提供 t ，则使用由 localtime() 返回的当前时间。 format 必须是一个字符串。
以下指令可以嵌入 format 字符串中。它们显示时没有可选的字段宽度和精度规范，并被 strftime() 结果中的指示字符替换：
指令|含义
---|:---
%a|本地化的缩写星期中每日的名称。
%A|本地化的星期中每日的完整名称。
%b|本地化的月缩写名称。
%B|本地化的月完整名称。
%c|本地化的适当日期和时间表示。
%d|十进制数 [01,31] 表示的月中日。
%H|十进制数 [00,23] 表示的小时（24小时制）。
%I|十进制数 [01,12] 表示的小时（12小时制）。
%j|十进制数 [001,366] 表示的年中日。
%m|十进制数 [01,12] 表示的月。
%M|十进制数 [00,59] 表示的分钟。
%p|本地化的 AM 或 PM 。
%S|十进制数 [00,61] 表示的秒。
%U|十进制数 [00,53] 表示的一年中的周数（星期日作为一周的第一天）作为。在第一个星期日之前的新年中的所有日子都被认为是在第0周。
%w|十进制数 [0(星期日),6] 表示的周中日。
%W|十进制数 [00,53] 表示的一年中的周数（星期一作为一周的第一天）作为。在第一个星期一之前的新年中的所有日子被认为是在第0周。
%x|本地化的适当日期表示。
%X|本地化的适当时间表示。
%y|十进制数 [00,99] 表示的没有世纪的年份。
%Y|十进制数表示的带世纪的年份。
%z|时区偏移以格式 +HHMM 或 -HHMM 形式的 UTC/GMT 的正或负时差指示，其中H表示十进制小时数字，M表示小数分钟数字 [-23:59, +23:59] 。
%Z|时区名称（如果不存在时区，则不包含字符）。
%%|字面的 '%' 字符。

注释:
- 当%p与 strptime() 函数一起使用时，如果使用 %I 指令来解析小时， %p 指令只影响输出小时字段。
- %S范围真的是 0 到 61 ；值 60 在表示 leap seconds 的时间戳中有效，并且由于历史原因支持值 61 。
- 当%W与 strptime() 函数一起使用时， %U 和 %W 仅用于指定星期几和年份的计算。

示例如下：
```python
>>> import time
>>> time.gmtime(1592316546.3528154)
time.struct_time(tm_year=2020, tm_mon=6, tm_mday=16, tm_hour=14, tm_min=9, tm_sec=6, tm_wd
ay=1, tm_yday=168, tm_isdst=0)>>> a = time.gmtime(1592316546.3528154)
>>> time.strftime('%a %d %b %Y %H:%M:%S +0000',a)
'Tue 16 Jun 2020 14:09:06 +0000'
```

##### time.strptime(string,[format])
根据格式解析表示时间的字符串。
示例如下：
```python
>>> import time
>>> time.strptime('12 Jul 15','%d %b %y')
time.struct_time(tm_year=2015, tm_mon=7, tm_mday=12, tm_hour=0, tm_min=0, tm_sec=0, tm_wda
y=6, tm_yday=193, tm_isdst=-1)
```
##### class time.struct_time
返回的时间值序列的类型为 gmtime() 、 localtime() 和 strptime() 。它是一个带有 named tuple 接口的对象：可以通过索引和属性名访问值。 存在以下值：
索引|属性|值
---|:---:|:---
0|tm_year|（例如，1993）
1|tm_mon|range [1, 12]
2|tm_mday|range [1, 31]
3|tm_hour|range [0, 23]
4|tm_min|range [0, 59]
5|tm_sec|range [0, 61]； 见 strftime() 介绍中的 (2)
6|tm_wday|range [0, 6] ，周一为 0
7|tm_yday|range [1, 366]
8|tm_isdst|0, 1 或 -1；如下所示
N/A|tm_zone|时区名称的缩写
N/A|tm_gmtoff|以秒为单位的UTC以东偏离

### datetime_基本的日期和时间类型
