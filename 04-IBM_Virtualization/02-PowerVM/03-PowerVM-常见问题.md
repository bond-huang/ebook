# PowerVM-常见问题
VIOS系统中常见问题。
## HMC连接问题
### HMC查看虚拟网络报错
#### 报错示例一
报错示例：
```
"Error occurred while querying for SharedEthernetAdapter from 
VIOS <VIOS_PARTITION_NAME> with ID <VIOS_PARTITION_ID) in System <MANAGED_SYSTEM_NAME>- 
Unable to connect to Database."
```
官方描述：      
[HMC Enhanced GUI-"Error occurred while querying for SharedEthernetAdapter... Unable to connect to Database"](https://www.ibm.com/support/pages/node/961332?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

#### 报错示例二
报错示例：
```
Error occurred while querying for SharedEthernetAdapter from 
VIOS <PARTITION_NAME> with ID <PARTITION ID> in System 
<MACHINE_TYPE-MODEL*SERIAL_NO> - The system is currently 
too busy to complete the specified request. Please retry 
the operation at a later time. If the operation continues to fail, 
check the error log to see if the filesystem is full.
```
官方说明：    
[HMC Enhanced GUI - "Error occurred while querying for SharedEthernetAdapter...The system is currently too busy to complete the specified request."](https://www.ibm.com/support/pages/node/1073894?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

### 查看存储池报错
报错示例：
```
"Cannot connect to one or more Virtual I/O Servers.
Virtual I/O Server Error Details
Error occurred while querying for VirtualMediaRepository 
from VIOS <VIOS_PARTITION_NAME> with ID <VIOS_PARTITION_ID) 
in System <MANAGED_SYSTEM_NAME>- Unable to connect to Database."
```
官方描述：   
[HMC Enhanced GUI - "Error occurred while querying for VirtualMediaRepository from VIOS... Unable to connect to Database."](https://www.ibm.com/support/pages/node/1077855?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

### 解决方法
官方针对以上问题的解决方案：        
[When Using HMC GUI you see message "Unable to connect to the Database Error occurred "](https://www.ibm.com/support/pages/when-using-hmc-gui-you-see-message-unable-connect-database-error-occurred)

## VIOS升级问题
### root用户被删
通过updateios命令进行VIOS升级失败，出现类似以下错误： 
```
sysck: 3001-037 The name root is not a known user for file /usr/bin/rm_mlcache_file.
sysck: 3001-003 A value must be specified for owner for entry /usr/bin/rm_mlcache_file.
```
&#8195;&#8195;原因是root用户被删除了，可以尝试手动重新创建root用户，并确保所有用户属性都与运行中的VIOS的root用户相匹配。 或者从备份系统进行恢复。

官方说明：[updateios errors "sysck: 3001-037 The name root is not a known user for file"](https://www.ibm.com/support/pages/node/742815?mhsrc=ibmsearch_a&mhq=sysck%3A3001-038%203001-017)

### ntp.conf can not be a link
&#8195;&#8195;更新VIOS时出现“ sysck：3001-017”错误，可能是由于文件ntp.conf是“链接”而不是“文件”引起的。 updateios和alt_root_vg都将报告错误。 报错示例如下：
```
sysck: 3001-017 Errors were detected validating the files for package ios.cli.rte.

0503-464 installp: The installation has FAILED for the
"usr" part of the following filesets:
ios.cli.rte 6.1.x.yyy

ios.cli.rte 6.1.x.yyy USR APPLY FAILED
ios.cli.rte 6.1.x.yyy USR CLEANUP SUCCESS
```
确认方式：
```
$ ls -ld /home/padmin/config/ntp.conf
lrwxrwxrwx 1 root system 13 Nov 17 2014 /home/padmin/config/ntp.conf -> /etc/ntp.conf
```
解决方法：创建"/home/padmin/config/ntp.conf"文件作为"file"而不是动态"link".
```
# mv /home/padmin/config/ntp.conf /home/padmin/config/ntp.conf_link
```
或者
```
# rm /home/padmin/config/ntp.conf
# cp /etc/ntp.conf /home/padmin/config/ntp.conf
# chmod 760 /home/padmin/config/ntp.conf
# chown root:staff /home/padmin/config/ntp.conf
```
查看：
```
$ ls -ld /home/padmin/config/ntp.conf
-rwxrw---- 1 root staff 630 mmm dd yyyy /home/padmin/config/ntp.conf
```
升级或克隆方法：
```
$ updateios -dev <directory_with_VIO_update> -accept -install
$ alt_root_vg -bundle update_all -location <directory_vios_VIO_update> -target <hdisk#>
```
官方说明：[VIO updateios reports sysck 3001-017. File ntp.conf can not be a link.](https://www.ibm.com/support/pages/node/646359?mhsrc=ibmsearch_a&mhq=sysck%3A3001-038%203001-003)

## 待补充
