# HMC-常用命令
HMC管理说实话用命令的很少，一般都是图形化界面，很方便；但是在一些对受管机器分区进行批量处理的时候，或者查看获取一些受管机器信息的时候，用命令查看配置或者是写脚本去抓取数据还是比较方便的。
## 常用命令
### 管理HMC命令
介绍一些常用管理HMC控制台的命令
#### hmcshutdown
关闭或重启HMC，示例：
```shell
#立即关机
hscroot@hmc:~> hmcshutdown -t now
#等1分钟后关闭
hscroot@hmc:~> hmcshutdown -t 1
#等1分钟后重启
hscroot@hmc:~> hmcshutdown -t 1 -r
```
#### lshmc
列出HMC的设置，示例：
```shell
#列出HMC版本信息
hscroot@hmc:~> lshmc -V
#列出HMC VPD信息
hscroot@hmc:~> lshmc -v
#列出HMC远程访问信息
hscroot@hmc:~> lshmc -r
#列出HMC网络设置信息
hscroot@hmc:~> lshmc -n
#列出HMC硬件信息
hscroot@hmc:~> lshmc -h
```
#### bkconsdata
备份硬件控制台HMC重要数据,示例：
```shell
#ftp方式备份
hscroot@hmc:~> bkconsdata -r ftp -h ftpserver -u ftpuser --passwd ftppassword
#nfs方式备份
hscroot@hmc:~> bkconsdata -r nfs -h 192.168.1.10 -l /tmp/hmc/backups
#dvd方式备份
hscroot@hmc:~> bkconsdata -r dvd
```
#### chhmc
修改硬件管理控制台（HMC）设置，示例：
```shell
#修改控制名称
hscroot@hmc:~>chhmc -c network -s modify -h mynewhost
#修改控制台eth0 ip
hscroot@hmc:~>chhmc -c network -s modify -i eth0 -a 192.168.1.10 -nm 255.255.255.0
#修改ssh，http和web访问限制
hscroot@hmc:~>chhmc -c ssh -s enable
hscroot@hmc:~>chhmc -c http -s disable
hscroot@hmc:~>chhmc -c websm -s enable
#添加或删除ntp服务
hscroot@hmc:~>chhmc -c xntp -s add mytimeserver.company.com
hscroot@hmc:~>chhmc -c xntp -s add -a 192.168.1.32 -i eth0
hscroot@hmc:~>chhmc -c xntp -s remove mytimeserver.company.com
```
### 管理受管机器命令
所有操作都可以在HMC图形化管理界面进行操作，当需要在HMC可以用命令行对受管小型机进行相应配置操作，用命令对查看小型机信息和进行批量配置时候很有用。
####  mkvterm
打开受管系统中分区的虚拟终端会话，示例：
```shell
#打开指定受管系统分区id为1的分区的虚拟终端会话
hscroot@hmc:~>mkvterm -m <managed system> --id 1
```
通常采用vtmenu命令进去设备菜单，再选择对应的分区进入虚拟终端会话
####  rmvterm
关闭受管系统中分区的虚拟终端会话，示例：
```shell
#关闭指定受管系统分区id为1的分区的虚拟终端会话
hscroot@hmc:~>rmvterm -m <managed system> --id 1
```
分区的虚拟终端会话只允许开一个，如果想使用但是被占用，又不知道谁占用，可以用此命令关闭
#### lsdump
列出受管系统的dump信息，示例：
```shell
#列出所有dump
hscroot@hmc:~>lsdump -h
#列出指定受管设备的dump
hscroot@hmc:~>lsdump -m <managed system>
```
#### lssvcevents
列出可维护事件，示例：
```shell
#列出所有维护事件
hscroot@hmc:~>lssvcevents -t hardware -d 0
#列出控制台3天内的维护事件
hscroot@hmc:~>lssvcevents -t console -d 3
#列出指定受管系统打开的维护事件
hscroot@hmc:~>lssvcevents -t hardware -m <managed system> --filter "status=open"
```
#### lsled
列出受管系统的 LED 信息，示例：
```shell
#列出指定受管设备的物理attention LEDs状态
~>lsled -m <managed system> -r sa -t phys
#列出指定受管设备的指定Lpar警告灯状态
~>lsled -m <managed system> -r sa -t virtuallpar --filter ""lpar_names=lpar1,lpar2""
```
#### chled
列出受管系统的 LED 状态，示例：
```shell
#关闭指定受管设备的物理attention LED
~>chled -m <managed system> -r sa -t phys -o off
#打开指定lpar的attention LED
~>chled -m <managed system> -r sa -t virtuallpar -o on -p lpar3
#关闭lpar id 为2的lapr 的attention LED
~>chled -m <managed system> -r sa -t virtuallpar -o off --id 2
```
#### lssysconn
列出受管设备的连接信息，示例：
```shell
#列出所有受管设备的连接信息
~>lssysconn -r all
#列出所有受管设备的连接IP和状态
lssysconn -r all -F ipaddr:state
```
#### lshwres
列出受管系统的硬件资源，示例：
```shell
#列出物理I/O相关信息
~>lshwres -r io --rsubtype unit -m <managed system>
~>lshwres -r io --rsubtype bus -m <managed system> --filter "units=U787A.001.*******"
~>lshwres -r io --rsubtype slot -m <managed system> --filter "units=U787A.001.*******,"buses=2,3"" -F drc_index,description,<lpar_name>
~>lshwres -r io --rsubtype iopool -m <managed system> --level pool
~>lshwres -r io --rsubtype taggedio -m <managed system> --filter "lpar_ids=1"
#列出虚拟适配器相关信息
~>lshwres -r virtualio --rsubtype eth --level lpar -m <managed system>
~>lshwres -r virtualio --rsubtype scsi -m <managed system> -F --header
~>lshwres -r virtualio --rsubtype slot -m <managed system> --level slot --filter "lpar_names=lpar1"
#列出内存相关信息
~>lshwres -r mem -m <managed system> --level sys
~>lshwres -r mem -m <managed system> --level lpar -R
~>lshwres -r mem -m <managed system> --level lpar --filter ""lpar_names=lpar_1,lpar2""
#列出处理器相关信息
~>lshwres -r proc -m <managed system> --level sys -F installed_sys_proc_units:configurable_sys_proc_units
~>lshwres -r proc -m <managed system> --level lpar
#列出hca相关信息
~>lshwres -r hca -m <managed system> --level sys
~>lshwres -r hca -m <managed system> --level lpar --filter "lpar_names=lpar1"
```
#### chhwres
更改受管系统的硬件资源配置，示例：
```shell
#将指定受管系统指定分区的处理器移动到指定的分区
~>chhwres -r proc -m <managed system> -o m -p <partition name1> -t <partition name2> --procs 1
#移除指定受管系统指定分区中virtual slot号为3的虚拟适配器
~>chhwres -r virtualio -m <managed system> -o r -p <partition name> -s 3
```
#### lssyscfg
```shell
列出分区，分区配置文件；系统配置文件，受管系统信息；受管机架框或受管机柜信息，示例：
#列出所有受管系统的配置信息
~>lssyscfg -r sys
#列出指定受管系统的配置信息
~>lssyscfg -r sys -m <managed system>
#列出指定受管系统属性名称的标题
~>lssyscfg -r lpar -m <managed system> -F --header
#列出指定受管系统中指定分区的配置信息
~>lssyscfg -r lpar -m <managed system> --filter ""lpar_names=lpar1,lpar2,lpar3""
#列出指定受管系统中指定分区中指定的相关信息
~>lssyscfg -r lpar -m <managed system> --filter ""lpar_names=lpar1,lpar2,lpar3"" -F name,lpar_id,state
#列出指定受管系统中指定分区的profile信息
~>lssyscfg -r prof -m <managed system> --filter "lpar_names=lpar2"
#列出指定受管系统中指定分区中的指定profile信息
~>lssyscfg -r prof -m <managed system> --filter "lpar_ids=2,"profile_names=prof1,prof2""
#列出指定受管系统的profile信息
~>lssyscfg -r sysprof -m <managed system>
#列出指定受管系统中指定的某一个profile信息
~>lssyscfg -r sysprof -m <managed system> --filter "profile_names=sysprof1"
#列出所有managed frames信息
~>lssyscfg -r frame
#列出指定managed frames信息
~>lssyscfg -r frame -e <managed frames>
```
#### mksyscfg
为受管系统创建分区、分区概要文件或系统概要文件，示例：
```shell
#为指定受管系统创建分区
~>mksyscfg -r lpar -m <managed frames> -i "name=<partition name>,profile_name=<profile name>,lpar_env=aixlinux,min_mem=256,desired_mem=1024,max_mem=1024,proc_mode=ded,min_procs=1,desired_procs=1,max_procs=2,sharing_mode=share_idle_procs,auto_start=1,boot_mode=norm"
#为指定受管系统创建profile
~>mksyscfg -r prof -m <managed frames> -f /tmp/profcfg
#为指定受管系统中的指定分区创建profile
~>mksyscfg -r prof -m <managed frames> -o save -p <partition name> -n <profile name>
```
#### lspartition
列出分区信息：
```shell
# Lists partitions with RMC connection as known by LparCmdRM   
~> lspartition -dlpar
# Lists partitions with RMC connection as known by SFP's ServiceRM
~> lspartition -sfp
# 列出指定机器的分区信息
~> lspartition -c <Machine Type, Model, and Serial> 
# Invoked by InvScout directly for ext data.Return format: <LParID,IPaddress,active,hostname,OStype,OSlevel;>
~> lspartition -ix
```
#### lslparutil
列出受管系统和分区的利用率指标。该命令仅在Integrated Virtualization Manager环境中可操作:
```shell
# List configuration attributes for utilization monitoring 
~> lslparutil -r config
# List utilization data for logical partitions 
~> lslparutil -r lpar
```
## 使用示例
查看指定受管设备的lpar信息：
```shell
hscroot@TEST:~> lssyscfg -r lpar -m Server-9117-570-SN65YDR6E -F --header
name,lpar_id,lpar_env,state,resource_config,os_version,logical_serial_num,default_profile,
curr_profile,work_group_id,shared_proc_pool_util_auth,allow_perf_collection,power_ctrl_lpar_ids,boot_mode,ipl_source,lpar_keylock,auto_start,redundant_err_path_reporting,rmc_state,rmc_ipaddr,rmc_osshutdown_capable,dlpar_mem_capable,dlpar_proc_capable,dlpar_io_capable,sync_curr_profile,uuid
teacher02 9.210.114.218,10,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6EA,client08,cl
ient08,none,0,0,none,norm,null,norm,0,0,active,9.210.114.218,1,1,1,1,0,08A64825-D324-4FBB-A6D3-D44143ED8AF3
teacher01 9.220.154.227,9,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6E9,client07,cli
ent07,none,0,0,none,norm,null,norm,0,0,inactive,9.220.154.227,0,0,0,0,1,3E8A72FB-C815-4AAD-9754-89037455DCFC
...
hscroot@TEST:~> lssyscfg -r lpar -m Server-9117-570-SN65YDR6E -F
teacher02 9.210.114.218,10,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6EA,client08,client08,none,0,0,none,norm,null,norm,0,0,active,9.210.114.218,1,1,1,1,0,08A64825-D324-4FBB-A6D3-D44143
ED8AF3
teacher01 9.220.154.227,9,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6E9,client07,client07,none,0,0,none,norm,null,norm,0,0,inactive,9.220.154.227,0,0,0,0,1,3E8A72FB-C815-4AAD-9754-89037
455DCFC
...
```
列出RMC连接的分区信息：
```shell
hscroot@TEST:~> lspartition -dlpar                                       
<#1> Partition:<6*9117-570*65YDR6E, , 9.210.114.214>
       Active:<1>, OS:<AIX, 6.1, 6100-09-08-1642>, DCaps:<0xc3f>, CmdCaps:<0x3b, 0x3b>, Pi
nnedMem:<645>
<#2> Partition:<10*9117-570*65YDR6E, , 9.220.154.228>
       Active:<1>, OS:<AIX, 7.1, 7100-04-03-1642>, DCaps:<0xc3f>, CmdCaps:<0x1b, 0x1b>, Pi
nnedMem:<559>
```
## 用户权限
各用户命令使用权限参考官方文档：[HMC 任务，用户角色，标识和相关联的命令](https://www.ibm.com/support/knowledgecenter/zh/POWER7/p7ha1/hmctasksanduserroles.htm)
