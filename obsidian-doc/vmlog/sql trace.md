记录内容：
```
{
	remote_ip://远程访问ip,            
	dw_username:"dw_xx"//dw_用户账号   
	tenant_info://租户信息
	sql_orgional:""原始sql内容,
	sql_simple:"xxx"//sql不含查询时间范围,且用占位符将参数替换
	sql_range_time:"x"//sql查询时间范围，起始时间与结束时间中间采用「#」进行分割
	sql_fingerprint:"xxx"//针对原始sql进行sql占位符替换包括时间。随后针对已替换的内容取md5
	sql_exec_start_time:""执行起始时间
	sql_exec_end_time:""执行结束时间
	exec_duration_ms://执行时长 = sql_exec_end_time - sql_exec_start_time
	effect_rows:""执行结果行数
	effect_bytes:""//执行结果数据量（字节）
	[opt]scanned_total_rows:"扫描总行数"
	[opt]scanned_total_bytes:扫描总数据量（字节）
	[opt]error_info:"错误信息"
	[opt]cpu_time_ms:"" cpu耗时
	[opt]peak_memory_bytes:""峰值内存使用，记录查询在任意时刻占用的最大内存量
	
	creaet_time://当前时间，即当前数据创建的时间
	update_time://更新时间，即后续数据若产生更新，则记为数据更新时间
}
```

