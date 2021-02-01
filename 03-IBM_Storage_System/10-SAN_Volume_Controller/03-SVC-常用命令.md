# SVC-常用命令
&#8195;&#8195;SVC命令使用比较少，偶尔也会用到。SVC在7.8及以前版本中，没有awk命令，没有grep命令，有sed命令，但是在更老的7.2版本中，sed命令也没有。V8版本及以后版本目前还没尝试有没有这些命令。     
SVC命令Knowledge Center：[Command-line interface](https://www.ibm.com/support/knowledgecenter/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_clicommandscontainer_229g0r.html)
## Volume命令
### lsvdisk
显示系统可以识别的卷的简要列表或详细视图，Syntax如下：
```
>>- lsvdisk -- --+--------------------------------+-- ---------->
                 '- -filtervalue -- attrib=value -'
>--+----------+-- --+----------+-- ----------------------------->
   '- -nohdr -'     '- -bytes -'
>--+-----------------------+-- -- --+-----------------+--------->
   '- -delim -- delimiter -'        '- -filtervalue? -'
>--+---------------+-------------------------------------------><
   +- object_id ---+
   '- object_name -'
```
使用示例如下：
```
IBM_2145:SVC_Cluster:superuser>lsvdisk -delim : 0
id:0
name:E850C_LP1
IO_group_id:0
IO_group_name:io_grp0
status:online
mdisk_grp_id:1
mdisk_grp_name:Pool1
capacity:200.00GB
...
tier_capacity:55.78GB
compressed_copy:no
uncompressed_used_capacity:59.77GB
parent_mdisk_grp_id:1
parent_mdisk_grp_name:Pool1
encrypt:no
IBM_2145:SVC_Cluster:superuser>lsvdisk -delim :
id:name:IO_group_id:IO_group_name:status:mdisk_grp_id:mdisk_grp_name:capacity:type:FC_id:FC_name:RC_id:RC_name:vdisk_UID:fc_map_count:copy_count:fast_write_state:se_copy_count:RC_change:compressed_copy_count:parent_mdisk_grp_id:parent_mdisk_grp_name:formatting:encrypt:volume_id:volume_name:function
0:E850C_LP1:0:io_grp0:online:1:Pool1:200.00GB:striped:::::60050768018186A3000000000000001D:0:1:not_empty:1:no:0:1:Pool1:no:no:0:E850C_LP1:
1:E850C_LP2:0:io_grp0:online:1:Pool1:200.00GB:striped:::::60050768018186A3000000000000001E:0:1:not_empty:1:no:0:1:Pool1:no:no:1:E850C_LP2:
2:E850C_LP3:0:io_grp0:online:1:Pool1:200.00GB:striped:::::60050768018186A30000000000000006:0:1:not_empty:1:no:0:1:Pool1:no:no:2:E850C_LP3:
...
```
一个卷的信息非常多，如果只需取部分数据，可以使用脚本：
```sh
for i in {0..5};do lsvdisk -delim : $i |
sed -n '/^id:\|^name:\|^capacity:\|^real_capacity:/p'; done |
sed '/^id/{x;p;x;}'|
sed '/^id/{N;s/\n/ /}'|
sed '/^id/{N;s/\n/ /}'|
sed '/^id/{N;s/\n/ /}'|
sed '/^id/{N;s/\n/ /}'
```
上面脚本运行后示例：
```
id:0 name:E850C_LP1 capacity:200.00GB real_capacity:63.78GB 
id:1 name:E850C_LP2 capacity:200.00GB real_capacity:29.30GB 
id:2 name:E850C_LP3 capacity:200.00GB real_capacity:22.70GB 
id:3 name:E850C_LP4 capacity:200.00GB real_capacity:30.16GB 
id:4 name:E850C_LP5 capacity:200.00GB real_capacity:10.91GB 
id:5 name:E850C_LP6 capacity:200.00GB real_capacity:15.31GB 
```
上面脚本把同一卷结果输出在一行，按照原有的格式可以用如下脚本：
```sh
for i in {0..3}; do lsvdisk -delim : $i |
sed -n '/^id:\|^name:\|^capacity:\|^real_capacity:/p'; done |
sed '/^id/{x;p;x;}'  
```
上面脚本运行后示例：
```
id:0
name:E850C_LP1
capacity:200.00GB
real_capacity:63.78GB

id:1
name:E850C_LP2
capacity:200.00GB
real_capacity:29.30GB

id:2
name:E850C_LP3
capacity:200.00GB
real_capacity:22.70GB

id:3
name:E850C_LP4
capacity:200.00GB
real_capacity:30.16GB
```
官方参考链接：[lsvdisk](https://www.ibm.com/support/knowledgecenter/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lsvdisk_21pdwu.html)
### lsvdisksyncprogress
显示卷拷贝同步的进度，示例如下：
```
IBM_2145:SVC_Cluster1:superuser>lsvdisksyncprogress 0
vdisk_id vdisk_name copy_id progress estimated_completion_time
0        vdisk0     0       100
0        vdisk0     1       100     
```
官方参考链接：[lsvdisksyncprogress](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lsvdisksyncprogress_3r3rw0.html)
### lshostvdiskmap
显示映射到主机的卷的列表，Syntax如下：
```
>>- lshostvdiskmap -- --+----------+-- ------------------------->
                        '- -nohdr -'
>--+-----------------------+-- --+-------------+---------------><
   '- -delim -- delimiter -'     +- host_id ---+
                                 '- host_name -'
```
示例如下：
```
IBM_2145:SVC_Cluster1:superuser>lshostvdiskmap 0
id name  SCSI_id vdisk_id vdisk_name vdisk_UID                        IO_group_id IO_group_name
0  TSTDB 0       0        tstheart   600507680180863E5000000000000000 0           io_grp0
0  TSTDB 1       1        tstdata1   600507680180863E5000000000000001 0           io_grp0
0  TSTDB 2       2        tstdata2   600507680180863E5000000000000002 0           io_grp0
0  TSTDB 3       247      tstdata3   600507680180863E5000000000000146 0           io_grp0
0  TSTDB 4       248      tstdata4   600507680180863E5000000000000147 0           io_grp0
```
官方参考链接：[lshostvdiskmap](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lshostvdiskmap_21pele.html)
## Host命令
### lshost
生成一个包含有关系统可见的所有主机的简要信息以及有关单个主机的详细信息的列表，Syntax如下：
```
>>- lshost-- --+-----------------------------------+-- --------->
               '- -filtervalue -- attribute=value -'
>--+----------+-- --+-----------------------+-- -- ------------->
   '- -nohdr -'     '- -delim -- delimiter -'
>--+-----------------+--+---------------+----------------------><
   '- -filtervalue? -'  +- object_id ---+
                        '- object_name -'
```
使用示例：
```
IBM_2145:SVC_Cluster1:superuser>lshost
id  name                        port_count iogrp_count status
0   HOST1                       4          4           offline
1   host22                      4          4           offline
2   host33                      4          4           offline
3   host                        4          4           online
```
显示所有offline的主机：
```
IBM_2145:SVC_Cluster1:superuser>lshost -filtervalue status=offline
id  name                   port_count iogrp_count status
0   HOST1                       4          4           offline
1   host22                      4          4           offline
2   host33                      4          4           offline
```
官方参考链接：[lshost](https://www.ibm.com/support/knowledgecenter/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lshost_21pdxx.html)
## IO Group命令
### lsiogrp
显示对系统可视的输入/输出 (I/O) 组的简明列表或详细视图，示例如下：
```
IBM_2145:SVC_Cluster1:superuser>lsiogrp
id name            node_count vdisk_count host_count 
0  io_grp0         2          804         244        
1  io_grp1         0          0           244        
2  io_grp2         0          0           244        
3  io_grp3         0          0           244        
4  recovery_io_grp 0          0           0          
IBM_2145:SVC_Cluster1:superuser>lsiogrp 0
id 0
name io_grp0
node_count 2
vdisk_count 804
host_count 244
flash_copy_total_memory 20.0MB
flash_copy_free_memory 20.0MB
remote_copy_total_memory 50.0MB
remote_copy_free_memory 19.7MB
mirroring_total_memory 62.0MB
mirroring_free_memory 0.0MB
raid_total_memory 40.0MB
raid_free_memory 40.0MB
maintenance no
compression_active no
accessible_vdisk_count 804
compression_supported yes
```
官方参考链接：[lsiogrp](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lsiogrp_21pdk1.html)

## 待补充
