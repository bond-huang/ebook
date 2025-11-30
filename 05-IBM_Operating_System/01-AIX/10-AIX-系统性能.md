# AIX-系统性能
记录一些常见的AIX系统性能查看分析知识。
## CPU性能
## 常用命令
### iostat
iostat命令选项说明：
- `tin`：显示系统为所有 TTY 读取的字符总数
- `tout`：显示系统向所有 TTY 写人的字符总数
- `% user`：显示了在用户级(应用程序)执行时生成的CPU 使用率百分比
- `%  sys`：显示了在系统级(内核)执行时生成的CPU 使用率百分比
- `%  idle`：显示了在 CPU 空闲并且系统没有未完成的磁盘I/O请求时的时间百分比
- `% iowait`：显示了 CPU 空闲期间系统有未完成的磁盘I/O请求时的时间百分比
- `physc`：显示消耗的物理处理器的数量或尾数，仅当分区正在运行共享处理器运行时才显示此信息
- `%  entc`：显示消耗的授权容量的百分比，仅当分区正在使用共享处理器运行时才显示此信息。由于计算该数据所依据的时间基础会发生变化，因此授权容量百分比有时可能超过 100%。这种超过只在采样时间间隔很小时才会比较明显
- `%  rc`：显示消耗的处理器资源的百分比，仅对于已强制实行处理器资源限制的 WPAR，才会显示此信息

## 参考文档
AIX官方性能数据收集脚本：[AIX MustGather: System Performance Analysis](https://www.ibm.com/support/pages/node/875894)

官方参考文档：
- [AIX 7.2 The sar command](https://www.ibm.com/docs/en/aix/7.2?topic=monitoring-sar-command)
- [Disk I/O pacing](https://www.ibm.com/docs/en/aix/7.1?topic=performance-disk-io-pacing)
- [Recommended AIX Virtual Memory Manager settings for DB2 database product](https://www.ibm.com/support/pages/recommended-aix-virtual-memory-manager-settings-db2-database-product)
- [Archived | Part 3, Tuning swap space settings](https://developer.ibm.com/articles/au-aix7memoryoptimize3/)
- [Values for minperm and maxperm parameters](https://www.ibm.com/docs/zh/aix/7.1?topic=tuning-values-minperm-maxperm-parameters)
- [Archived | Part 2, Monitoring memory usage (ps, sar, svmon, vmstat) and analyzing the results](https://developer.ibm.com/articles/au-aix7memoryoptimize2/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)

其它参考文档：
- [Tuning the AIX file caches](https://www.stix.id.au/wiki/Tuning_the_AIX_file_caches)
- [修改aix操作系统参数maxclient%和maxperm%的一点记录](https://blog.51cto.com/u_14036245/4372242)
- [maxperm小记](https://blog.csdn.net/freedomx1oa/article/details/52162098)
- [AIX 中 Paging Space 使用率过高的分析与解决](https://developer.aliyun.com/article/527407)
- [BM AIX下的lru_file_repage参数是什么](http://blog.if98.com/328131696/manage/33920.html)
- [优化 AIX 7 内存性能](https://developer.aliyun.com/article/510149?spm=a2c6h.14164896.0.0.541054b3BmqP64)
- [aix +oracle 内存使用说明](https://www.talkwithtrend.com/Question/76266)

amepat -N

svmon -G

iostat -t 1 5
131 
lparstat 
7274
topas
4130
vmstat 


[IBM AIX: Performance Analysis Using iperf](https://www.ibm.com/support/pages/node/886387)