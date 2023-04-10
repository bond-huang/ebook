# AIX-系统性能
记录一些常见的AIX系统性能查看分析知识。
## CPU性能

## 参考文档
AIX官方性能数据收集脚本：[AIX MustGather: System Performance Analysis](https://www.ibm.com/support/pages/node/875894)

官方参考文档：
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