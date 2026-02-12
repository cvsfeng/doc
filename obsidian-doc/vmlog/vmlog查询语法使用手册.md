# 范围查询
## 时间范围查询
### 时区解释
- **T**：是**分隔符**，代表 "Time"。
- **Z**：是**时区**，代表 "Zero" (UTC/GMT)。
#### 详细解释
##### 1. `T` (Time Separator)
- **含义：** 它代表 **Time**（时间）。
- **作用：** 它的主要作用是作为一个**分隔符**，用来把“日期部分”（YYYY-MM-DD）和“时间部分”（HH:MM:SS）区分开来。
- **为什么需要它：** 在计算机解析字符串时，T 能够清晰地告诉系统：“日期到此结束，后面开始是具体的时间”。
##### 2. `Z` (Zulu / Zero Time)
- **含义：** 它代表 **UTC**（协调世界时），也就是格林威治标准时间（GMT）。
- **来源：** Z 代表 **Zero** offset（零偏移量），即与 UTC 的时差为 0。在军事和航空领域，Z 读作 **Zulu**，所以这也常被称为“Zulu Time”。
- **重要性：** 如果时间字符串末尾有 `Z`，说明这个时间是**世界标准时间**，而不是你当地的时间。
#### 举个例子
假设你看到这样一个时间字符串：
> **`2023-10-05T08:00:00Z`**
它的意思是：
1. **日期：** 2023年10月5日
2. **T：** (分隔符)
3. **时间：** 上午 08点00分00秒
4. **Z：** **UTC 标准时间** (不是北京时间！)

> **注意转换：** 如果要把这个时间换算成**中国标准时间 (UTC+8)**，你需要给这个时间加上 8 个小时。 即：`2023-10-05 16:00:00` (北京时间)。
### 简单时间范围查询
```
{service.name="product-catalog"} and _time:["2026-02-02T00:00:00Z","2026-02-02T10:00:00Z"]
```
### 过往天数内的指定时间范围
这句话表示查询指定服务名的==过往2周中每天08:00~09:00==这个时间段范围的数据
```
{service.name="product-catalog"} and _time:2w and _time:day_range[08:00,09:00]
```
[在线示例](https://play-vmlogs.victoriametrics.com/select/vmui/?_gl=1%2At75nlg%2A_gcl_au%2AOTE0MTUzNjU5LjE3NjkxNTc3NjE.%2A_ga%2AMTU3NjY3MjUxNi4xNzY3Nzc1NjIw%2A_ga_N9SVT8S3HK%2AczE3NzAxNzI5MjckbzE0JGcxJHQxNzcwMTc0OTAyJGo0NiRsMCRoMA..#/?query=%7Bservice.name%3D%22product-catalog%22%7D+and+_time%3A2w+and+_time%3Aday_range%5B08%3A00%2C09%3A00%5D&g0.range_input=7d&g0.end_input=2026-02-04T07%3A25%3A50&g0.relative_time=none&view=group&groupBy=_stream)
![[Pasted image 20260204153136.png]]
### 排除过往指定天的指定时间段
这句话表示排除过往4天中==[08:00,09:00]==这个时间段的数据。
这里使用了 `not` ,也可以使用 `-`  
`4d`表示是4天，也可以使用`w`表示周
- d
这样写表示过往4天内，每天不在==[08:00,09:00]==这个时间范围的日志数据。offset表示偏移8小时（转换成北京时间）
```
{service.name="product-catalog"} and _time:4d and not _time:day_range[08:00,09:00] offset 8h
```
![[Pasted image 20260204154145.png]]
- w
 若是w参数即：`_time:2w`则表示查询过去一周每天在==[08:00,09:00]==这个时间范围的日志。注意这里去掉了==not==
 ```
 {service.name="product-catalog"} and _time:2w and _time:day_range[08:00,09:00] offset 8h
 ```
![[Pasted image 20260204154841.png]]

### 周范围选择器
这个时间选择器比较有意思，可以返回每周特定日期的日志，但仅支持==[start,end]==
比如：
- Sun或Sunday
- Mon或Monday
- Tue或Tuesday
- Wed或Wednesday
- Thu或Thursday
- Fri或Friday
- Sat或Saturday
这表示筛选出上两周不在这个时间范围内的数据。即：仅周日的数据
```
{service.name="product-catalog"} and _time:2w and -_time:week_range[Mon,Sat] offset 8h
```
![[Pasted image 20260204164918.png]]
![[Pasted image 20260204165003.png]]
也可以使用==()==即：不包含指定天数的
- 查询工作日、工作时间的数据
```
{service.name="product-catalog"} and _time:2w and _time:week_range[Mon,Fri] and _time:day_range[08:00,17:00] offset 8h
```
![[Pasted image 20260204165357.png]]
# stream过滤器
## `_stream_id`过滤器
### uniq by() vs fields
#### fields查询场景
```
{_stream_id:in(_msg:"Product" | fields _stream_id) } 
```
分析：
子查询内==`_msg:"Product" | fields _stream_id`==有大量重复的 `_stream_id`所以这种方式的数据处理方式肯定会较低
![[Pasted image 20260204182232.png]]
#### uniq by()
```
_msg:"Product" | uniq by(_stream_id)
stream_id:in(_msg:"Product" | uniq by(_stream_id))
```
通过上面的语句我们不难看出最后的查询结果集里面的数据只有4条，对于网络io来说这种方式肯定是最优的；
![[Pasted image 20260204182828.png]]
### todo  子查询uniq by真的比fields效率高吗？

# 文字过滤器
最简单的logsql查询包含一个要在日志消息搜索中的单子，比如：`error`
此查询匹配以下日志消息：
- error
- an error happened
- error:cannot open file
```
product
```
![[Pasted image 20260205160931.png]]
```
Product
```
![[Pasted image 20260205161009.png]]
不难发现，文字过滤器如果在不指定字段的前提下搜索时，默认使用的是==`_msg`==字段，且对大小写敏感；
如果想大小写不敏感进行搜索可使用：==**i()**==
## 仅查大写
```
_stream_id: in("0000000000000000a2dc4d3051d99980b795d92763f04fe0","00000000000000006215e92095062fd573719b5215588754")|Product
```
![[Pasted image 20260205161950.png]]
## 仅查小写
```
_stream_id: in("0000000000000000a2dc4d3051d99980b795d92763f04fe0","00000000000000006215e92095062fd573719b5215588754")|product
```
![[Pasted image 20260205162039.png]]
## 忽略大小写查询
```
_stream_id: in("0000000000000000a2dc4d3051d99980b795d92763f04fe0","00000000000000006215e92095062fd573719b5215588754")|i(product)
```
![[Pasted image 20260205161908.png]]
# 短语过滤器
所谓短语过滤器即：搜索包含特定短语的日志消息，需要将短语用引号括起来。短语可以包含任何字符，包括空格、标点符号、括号等。搜索时会考虑这些字符。例如：
**短语**：==**frontend 10.71.9.48:36272**==
```
"frontend 10.71.9.48:36272"
```
![[Pasted image 20260205162916.png]]
# 前缀过滤器
如果需要搜索包含特定前缀的单词/短语的日志，只需在查询短语中单词末尾加上==**`*`**== 
默认情况下，前缀过滤器应用于默认字段==**`_msg`**== 
```
pro*
```
![[Pasted image 20260205164236.png]]
# 模式匹配过滤器
这个过滤器挺有意思，支持4中模式：任何部分、全匹配、前缀匹配、末尾匹配；
具体详见官方文档 
==[模式匹配过滤器]([https://docs.victoriametrics.com/victorialogs/logsql/#stream-filter](https://docs.victoriametrics.com/victorialogs/logsql/#pattern-match-filter))==
# 精值过滤(exact filter)
```
_msg:="User browsing product: L9ECAV7KIM"
```
这里的==**:=**==为精值匹配模式
![[Pasted image 20260205173559.png]]
# 精值前缀过滤
这个过滤器有些纠结
==简单来说是指定前缀的所有字段==
## 精值匹配
```
_msg:="User browsing product"
```

![[Pasted image 20260205173841.png]]
## 精值模糊匹配
```
_msg:="User browsing product"*
```
![[Pasted image 20260205174015.png]]

# Multi-exact filter vs equals_common_case vs contains_common_case
==**以下过滤器可以无序**==
## multi-exact filter
简单来说就是in查询，只是这个in查询里面的值是不忽略大小写的即：精值in查询
但这个in查询好像不能有空格产生
```
_msg:in("User broWsing product: 2ZYFJ3GM2N","OLJCESPC7Z")
```
![[Pasted image 20260205182710.png]]
## equals_common_case filter
这个就有意思了，会忽略大小写，但是整体单词语义不可变
```
_msg:equals_common_("User broWsing product: 2ZYFJ3GM2N","OLJCESPC7Z")
```
![[Pasted image 20260205183503.png]]
## contains_common_case filter
这个就是包含指定短语和单词的都会被查询出来，并且会忽略大小写
```
_msg:contains_common_case("2ZYFJ3GM2N","OLJCESPC7Z")
```
![[Pasted image 20260206113220.png]]
## 结论
这三个过滤器，各有千秋。如果想精值匹配in查询，那么直接使用==**in**==即可。如果想忽略大小写精值匹配则使用==**equals_common_case**==如果忽略大小写，模糊匹配则使用==**contains_common_case**==

# 序列过滤器
有时候，需要查找特定顺序排列的单词或短语的日志消息。比如：`error` `open file`
```
_msg:seq("request","via","stream")
```
这个查询即可匹配
>[2026-02-02 09:38:41.348][8][info][main] [source/server/server.cc:503] ==request== header map: 656 bytes: :authority,:method,:path,:protocol,:scheme,accept,accept-encoding,access-control-request-headers,access-control-request-method,access-control-request-private-network,authentication,authorization,cache-control,cdn-loop,connection,content-encoding,content-length,content-type,expect,grpc-accept-encoding,grpc-timeout,if-match,if-modified-since,if-none-match,if-range,if-unmodified-since,keep-alive,origin,pragma,proxy-connection,proxy-status,referer,te,transfer-encoding,upgrade,user-agent,==via==,x-client-trace-id,x-envoy-attempt-count,x-envoy-decorator-operation,x-envoy-downstream-service-cluster,x-envoy-downstream-service-node,x-envoy-expected-rq-timeout-ms,x-envoy-external-address,x-envoy-force-trace,x-envoy-hedge-on-per-try-timeout,x-envoy-internal,x-envoy-ip-tags,x-envoy-is-timeout-retry,x-envoy-max-retries,x-envoy-original-path,x-envoy-original-url,x-envoy-retriable-header-names,x-envoy-retriable-status-codes,x-envoy-retry-grpc-on,x-envoy-retry-on,x-envoy-up==stream==-alt-stat-name,x-envoy-upstream-rq-per-try-timeout-ms,x-envoy-upstream-rq-timeout-alt-response,x-envoy-upstream-rq-timeout-ms,x-envoy-upstream-stream-duration-ms,x-forwarded-client-cert,x-forwarded-for,x-forwarded-host,x-forwarded-port,x-forwarded-proto,x-request-id
![[Pasted image 20260206114827.png]]
这样便什么匹配不出来
```
_msg:seq("stream","request","via")
```
![[Pasted image 20260206114758.png]]
# IPv4 range filter
这个过滤器真的是为了排查问题，且在运维体系相对比较规范的前提下设计的。但是在容器化部署的时代，这个功能的作用还没有想好场景；
```
user.ip:ipv4_range(127.0.0.1,127.255.255.255)
```

还可以基于网段，上述命令等同于
```
user.ip:ipv4_range("127.0.0.1/8")
```
# len_range filter
这个过滤器在某些统计场景下可能会比较有用，用于发现日志长度比较==**大**==的日志；
```
_msg:len_range(20,30)
```
上述含义表示过滤出长度在此范围内的==**`_msg`**==
![[Pasted image 20260206144428.png]]

```
_msg:len_range(20,Inf)
```
这表示不做上限控制
![[Pasted image 20260206144403.png]]
# eq_field过滤器
有时候需要查找给定字段中包含相同值的日志。这可以通过筛选器来实现：==**fiel1:eq_field(field2)**==
这个例子没有实验成功
```

```
# filed_names pipe
field_names 返回所有日志字段名称，以及每个字段名称对应的估计日志数量。例如：查询返回过去5分钟内所有字段名称及其对应的匹配日志数量：
```
_time:5m | field_names
```
![[Pasted image 20260209141926.png]]
# field_values pipe
返回给定field_name字段的所有值，查询返回每个值对应的日志数量。例如：查询返回`level`过去5分钟所有与该字段匹配的日志数量
```
_time:5m|field_values level
```
![[Pasted image 20260209142659.png]]
# fields pipe
默认所有的日志字段都会响应返回。可以使用 `| field1,field2,field3,field4`管道符选定指定的日志字段集。例如：以下查询仅选择最近5分钟日志中的`host,_msg`字段
```
_time:5m | fields os.type,_msg
```
![[Pasted image 20260210103313.png]]
# join pipe
## performance tips
- Make sure that the `<query>` in the `join` pipe returns relatively small number of results, since they are kept in RAM during execution of `join` pipe.
- [Conditional `stats`](https://docs.victoriametrics.com/victorialogs/logsql/#stats-with-additional-filters) is usually faster to execute. They usually require less RAM than the equivalent `join` pipe
# last pipe
`<q> |last N by(fields)`pip returns the last `N`logs form `<q>`query after sorting them by given fields.
For example，the following query returns the last 10 logs with the biggest value of `request_duration`field over the last 5 minutes:
```
_time:5m|last 10 by(request_duration)
```
It is possible to return up to `N`logs individually for each group of logs with the same set of fields,by enumerating the set of these fields in `parttion by(...)`

# limit pipe 
```
_time:5m|limit 10
```
# pack_json
`_time:5m|pack_json as field_name`pipe将查询返回的每个日志条目所有字段打包到josn对象中，并将其作为字符串存储在给定的`field_name`中；
如，以下查询将所有字段打包到一个json对象中，并将其存储在用于记录过去5分钟的`_msg`字段中：
```
_time:5m|pack_json as _msg
```
==**eg.**==
```
_time:5m | pack_json fields (collector, host.name) as baz
```
![[Pasted image 20260210182621.png]]
# pack_logfmt
==这个函数暂时没有想到使用场景==
```
_time:5m | pack_logfmt as _msg
```
![[Pasted image 20260210183032.png]]
# query_stats pipe
==**[统计型管道详见官网](https://docs.victoriametrics.com/victorialogs/logsql/#query_stats-pipe)**==
# rename pipe
如果需要重命名某些日志字段，可以使用`rename src1 as dst1`管道符
```
_time:5m | rename collector as c1
```
![[Pasted image 20260210185225.png]]
# replace pipe
`<q>|replace('old','new') at _msg`pipe将查询返回的所有日志中所有出现的子字符串替换`old`为给定的子字符串`new`

```
* | replace("Product","helloworld") at _msg
```
![[Pasted image 20260210185502.png]]
# conditional replace
如果replace管道必须应用于某些日志条目，则`if(filters)`后面添加replace。
```
_time:5m|replace if(user_type:=admin)("secret","****") at _msg
```
![[Pasted image 20260211102340.png]]
# regexp replace 和 conditional regexp replace 
正则替换和条件正则替换
# set_stream_fields
管道将给定的日志设置为字段`|set_stream_fields field1,field2,field3`
例如，如果将`_time:5m`过滤器返回的日志包含`host="foo"` and `path="/bar"`字段，则以下查询会将`_stream`field设置为`{host="foo",path="/bar"}`

```
_time:5m | set_stream_fields collector, _time
```

![[Pasted image 20260211104419.png]]
==conditional set_stream_fields==原理等同与上面
eg.
```
_time:5m|set_stream_fields if(host:="foobar") host,app
```

# sort pipe
默认情况下，出于性能考虑，日志以任意顺序选择。如果需要对日志进行排序，则可以使用`<q>|sort by(field1,field2)`管道符，根据给定字段，使用自然排序对`<q>`查询返回的日志进行排序。
例如：
```
_time:5m|sort by(_stream,_time)
```
在指定的日志字段后添加`desc` `--reverse`参数，即可按该字段的逆序排序。例如，以下查询将按`request_duration_seconds`字段逆序对日志进行排序：
```
_time:5m | sort by(reqeust_duration_seconds desc)
```
==**案例如下**==
```
_time:1h| fields _msg |uniq by(_msg)  |sort by(_msg desc)
```
![[Pasted image 20260211105808.png]]
也可以使用==order by ==与==**sort by**==查询等效
```
_time:1h| fields _msg |uniq by(_msg)  |order by(_msg desc)
```
![[Pasted image 20260211110129.png]]
## 更有意思的跳过查询
 如果需要跳过前几个排序结果，则`offset N` 可以将其添加到`sort`管道中。例如，以下查询会跳过`request_duration`字段值最大的前10条日志，然后返回最近5分钟内排序后的20条日志。
 ```
 _time:5m|order by(request_duration desc) offset 10 limit 10
 ```
### 使用场景
现实生活中比如在进行一些数据统计时，由于数据的首尾效应而引起的数据失真现象，那么就可以使用这种方式来去除几个最高值、去除几个最低值、随后进行数据统计；

## 对于sort by的注意事项
请注意，对大量日志进行排序可能速度较慢，并且会消耗大量额外内存。建议在排序前使用以下方法限制日志数量：
- limit n 在管道末端添加 `sort`
- 使用时间过滤器缩小选定的时间范围
- 使用更具体的筛选条件，因此他们选择的日志较少
- 通过管道限制所选字段的数量`fields`
# split pipe
这个管道挺有意思，可以按指定字符进行切割，然后再结合一些其它的管道进而实现字符统计的效果。
unroll展开包含分割效果的JSON数组非常方便。例如，以下查询返回过去5分钟内日志消息中出现频率最高的5个以逗号分隔的内容
```
 _time:5m |split "," from _msg as new_col |unroll new_col| top 5(new_col)
```
![[Pasted image 20260211112938.png]]
# stats pipe
该`<q>|stats ...`管道会计算`<q>`返回的日志各种统计信息。例如，以下LogsQL查询使用`count`stats函数来计算最近5分钟的日志数量：
```
_time:5m|stats count()
```
该`|stats ...`管道的基本格式如下：
```
...|stats
stats_func1() as result_name1,
stats_func2() as result_name2,
```
其中`stats_func*`是任何受支持的统计函数，而`result_name*`是用于存储相应统计函数结果的日志字段名称。`as`关键字是可选的。
例如，一下查询计算过去5分钟内日志的统计信息
- 借助==count==函数统计日志数量
- 借助stats函数统计唯一日志流的数量:==count_uniq==
```
 _time:5m |stats count(),count_uniq(_stream)
```

![[Pasted image 20260211114008.png]]
==可简化如下==
``` 
_time:5m |stats count(),count_uniq(_stream)
```
![[Pasted image 20260211113922.png]]
## stats-by-fields
有点像多阶段==**`group by`**==
以下LogsQL可用于计算每组日志字段的独立统计信息:
```
<q>|stats by(field1,field2,...,fieldM) 
stats_func1(...) as result_name1,
...
stats_funcN(...) as result_nameN,
```
这样会计算查询返回的日中查看到的`stats_func*`每一`(field1,field2,...,fieldM)`日志字段.
例如：
查询过去一小时按`os.type,service.name`分组数据总和以及分组过后唯一`_msg`
```
_time:1h|stats by (os.type,service.name)count()as total,count_uniq(_msg)
```
![[Pasted image 20260211142122.png]]
## stats-by-time-step
按时间段统计
以下语法可用于计算时间段分组的统计数据：
```
<q>|stats by(_time:step)
stats_func1(...) as result_name1,
...
stats_funcN(...) as result_nameN,
```
如：
以下查询会计算返回的日志中`stats_func*`每个step字段的值。持续时间可以任意设置。如：以下logsql查询返回过去5分钟内每分钟的日志数量和唯一ip地址：
```
_time:5m|stats by(_time:1m) count()as logs_total,count_uniq(collector) collector_cnt
```
![[Pasted image 20260211142848.png]]
## running_stats pipe
这个比较有意思，如果结合stats count可以实现每小时增量占比的效果
```
_time:1d|stats by(_time:1h) count() as hour_cnt | running_stats sum(hour_cnt) as running_hits
```
下图中的数据快照为：
```
[
  {
    "_time": "2026-02-10T06:00:00Z",
    "hour_cnt": "74995",
    "running_hits": "74995"
  },
  {
    "_time": "2026-02-10T07:00:00Z",
    "hour_cnt": "238808",
    "running_hits": "313803"
  },
  {
    "_time": "2026-02-10T08:00:00Z",
    "hour_cnt": "230967",
    "running_hits": "544770"
  },
  {
    "_time": "2026-02-10T09:00:00Z",
    "hour_cnt": "233989",
    "running_hits": "778759"
  }
]
```
数组元素中==**[1]**==running_hits = 313803 = 238808+74995
当前元素持续与上述所有元素做累和。
![[Pasted image 20260211144117.png]]
## stats-with-additional-filters
有时候需要计算不同匹配日志子集的统计信息。这里可以通过在统计函数和条件语句`if(<any_filter>)`之间插入一个条件来实现，其中条件语句可以包含任意过滤器，例如：
```
_time:5m | stats
  count() if (GET) gets,
  count() if (POST) posts,
  count() if (PUT) puts,
  count() total
```
如果输入的行数没有与给定的`if(...)`筛选条件匹配，则相应的统计函数将基于空的行集进行计算；
# top pipe
`<q>|top N by(field1,...,fieldN)`pipe返回查询的日志中匹配日志条目数量最多的日志字段的前`N`个集合。
例如：
```
_time:5m|top 7 by(_stream)
```
```
*|top 10 by(collector) | order by(hits)
```
![[Pasted image 20260211151557.png]]
# total_stats pipe
该`<q> | total_stats...`管道计算返回指定日志字段（全局）统计信息（例如全局计数或全局总和），并将这些统计信息存储在每个输入日志条目的指定日志字段中。
`total_stats`结合按时间划分的统计数据会很有用。例如，以下查询返回过去每一天每小时的日志数量，加上日志总数，然后计算每小时的日志百分比：
```
_time:1d|stats by(_time:hour) count()as hits | total_stats sum(hits) as total | math round((hits/total)*100) as percent
```
```
[
  {
    "_time": "2026-02-10T07:00:00Z",
    "hits": "80376",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T08:00:00Z",
    "hits": "230967",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T09:00:00Z",
    "hits": "233989",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T10:00:00Z",
    "hits": "228564",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T11:00:00Z",
    "hits": "230412",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T12:00:00Z",
    "hits": "234365",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T13:00:00Z",
    "hits": "240913",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T14:00:00Z",
    "hits": "225845",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T15:00:00Z",
    "hits": "215497",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T16:00:00Z",
    "hits": "209756",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T17:00:00Z",
    "hits": "210616",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T18:00:00Z",
    "hits": "217986",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T19:00:00Z",
    "hits": "230217",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T20:00:00Z",
    "hits": "223689",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T21:00:00Z",
    "hits": "219552",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T22:00:00Z",
    "hits": "216710",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-10T23:00:00Z",
    "hits": "210117",
    "percent": "5",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T00:00:00Z",
    "hits": "95007",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T01:00:00Z",
    "hits": "97931",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T02:00:00Z",
    "hits": "96454",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T03:00:00Z",
    "hits": "97030",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T04:00:00Z",
    "hits": "95601",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T05:00:00Z",
    "hits": "97113",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T06:00:00Z",
    "hits": "97454",
    "percent": "2",
    "total": "4399714"
  },
  {
    "_time": "2026-02-11T07:00:00Z",
    "hits": "63553",
    "percent": "1",
    "total": "4399714"
  }
]
```
![[Pasted image 20260211153946.png]]
# unpack_json pipe
`<q>|unpack_json from field_name`该管道将查询结果`{"K1":"V1",....,"Kn":"Vn"}`中的JSON解包为对应的输出字段名及其对应的值。他会使用列表中的名称覆盖现有字段，其它字段保持不变；此外`preserve_keys`的所有json字段扁平化;
```
* | unpack_json from _msg preserve_keys(kubernetes.container_name,kubernetes.pod_name)
```
# stats、total_stats 和 running_stats 管道函数详解
VictoriaLogs 的查询语言是 LogsQL（类似于 Loki 的 LogQL），这些管道函数（pipe functions）主要用于对日志流进行统计计算。它们在功能上有一定交集（如都支持 count、sum、min、max 等基本聚合），但设计目的、输出方式和适用场景不同；
## stats 管道函数
含义： 最常用的聚合统计函数，对匹配的日志流计算各种统计指标（如总数、平均值、唯一值计数、直方图等）。支持分组（by），可以按字段或时间桶进行聚合，输出是独立的聚合结果行（不是附加到原始日志）。
语法：
```
<查询过滤> | stats [by (字段1, 字段2, ...)] 函数1(参数) [AS 别名], 函数2(参数) [AS 别名], ...
```
- 支持函数：count()、avg()、sum()、min()、max()、median()、quantile()、histogram()、count_uniq()、rate() 等（最丰富）。
- by：可选，按字段分组（如 by (host)）或时间桶（如 by (_time:1h)）。
**示例**：

- 最近 5 分钟错误日志总数，按 level 分组：
```
_time:5m error | stats by (level) count() AS error_count
```
- 按小时统计日志数：
```
_time:1d | stats by (_time:1h) count() AS hourly_count
```
**使用注意**：
- 适合最终报告或可视化（Grafana 等）。
- 高基数字段（如 count_uniq）可能耗内存，可加 limit N 限制。
## total_stats 管道函数
**含义**： 计算**全局总计**（total/global stats），对整个日志流（或分组后）计算整体聚合值。结果会附加到每个输入日志行中（不是独立输出）。不支持复杂分组，主要用于获取“总和”作为上下文。
**语法**：
```
<查询过滤> | total_stats [by (字段1, ...)] 函数1(参数) [AS 别名], ...
```
- 支持函数：主要 count()、sum()、min()、max()（比 stats 少）。
- by：可选，按分组计算各自的总计。
**示例**：
- 结合 stats 计算每日总日志数（每小时占比）：
```
_time:1d | stats by (_time:1h) count() AS hourly_hits
| total_stats sum(hourly_hits) AS total_hits
| math round((hourly_hits / total_hits) * 100) AS percent
```
- 全局最大响应大小：
```
_time:5m | total_stats max(response_size) AS global_max_size
```
**使用注意**：
- 会加载所有匹配日志到内存，建议限制时间范围。
- 常与 stats by 时间桶 结合，用于计算相对指标（如占比）。
#### 3. running_stats 管道函数

**含义**： 计算**累积/运行统计**（running/cumulative stats），随着日志流顺序处理，逐步累加值（如运行总数、运行求和）。结果附加到每个日志行中，适合时间序列累积分析。
**语法**：
```
<查询过滤> | running_stats [by (字段1, ...)] 函数1(参数) [AS 别名], ...
```
- 支持函数：仅 count()、sum()、min()、max()（最少）。
- 需要先按时间排序（建议加 sort by (`_time`)）。
**示例**：
- 运行日志计数：
```
_time:5m | sort by (_time) | running_stats count() AS running_count
```
- 按主机累计错误数：
```
_time:1d error | sort by (_time) | running_stats by (host) count() AS running_errors
```
- 结合 stats 计算累计小时日志：
```
_time:1d | stats by (_time:1h) count() AS hourly_hits | running_stats sum(hourly_hits) AS cumulative_hits
```
**使用注意**：
- 会加载所有日志到内存，时间范围不要太大；
- 必须确保日志按时间顺序（否则累积不准）；
#### 三者比较（为什么有交集？不同场景怎么选？）
| 特性          | `stats`                         | `total_stats`                                  | `running_stats`                  |
| ----------- | ------------------------------- | ---------------------------------------------- | -------------------------------- |
| **主要目的**    | 静态聚合（分组统计）                      | 全局总计（整体求和）                                     | 动态累积（运行求和）                       |
| **支持函数**    | 最丰富（avg、quantile、histogram 等）   | 基础（count、sum、min、max）                          | 最少（仅 count、sum、min、max）          |
| **分组 (by)** | 全面支持（字段、时间桶）                    | 有限支持                                           | 支持（按组累积）                         |
| **输出方式**    | 独立聚合行                           | 附加到每行日志                                        | 附加到每行日志（累积值）                     |
| **内存消耗**    | 中等（取决于分组）                       | 高（加载全部日志）                                      | 高（加载全部日志）                        |
| **典型交集函数**  | count、sum、min、max               | count、sum、min、max                              | count、sum、min、max                |
| **适用场景**    | - 常规报告 - 分组指标（如每服务平均延迟） - 可视化图表 | - 需要整体总计（如每日总和） - 计算占比/百分比 - 与 `stats` 结合做相对指标 | - 累积趋势（如累计请求数） - 时间序列累计分析 - 运行监控 |
**为什么有交集？** 基本聚合（如 count/sum）是统计核心需求，但每个函数的处理方式不同：stats 是“汇总视图”，total_stats 是“全局上下文”，running_stats 是“逐步累加”。交集是为了在不同分析深度复用相同计算逻辑，避免重复实现。

**场景选择建议**：

- **想分组统计、复杂计算** → 用 stats（最常用）。
- **需要整体总和做相对计算（如错误占比）** → 用 total_stats（常后接 math）。
- **需要累计趋势（如运行总请求数曲线）** → 用 running_stats（需排序）。
- 常结合使用：先 stats by 时间桶 → 再 total_stats 或 running_stats 计算总/累积。
# quantile
```
_time:5m | stats
  quantile(0.5, request_duration_seconds) p50,
  quantile(0.9, request_duration_seconds) p90,
  quantile(0.99, request_duration_seconds) p99
```






















































