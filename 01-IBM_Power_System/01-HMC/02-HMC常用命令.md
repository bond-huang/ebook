# HMC常用命令
HMC管理说实话用命令的很少，一般都是图形化界面，很方便；但是在一些对受管机器分区进行批量处理的时候，或者查看获取一些受管机器信息的时候，用命令还是比较方便的。
### 管理HMC命令
介绍一些常用管理HMC控制台的命令
##### hmcshutdown
关闭或重启HMC，示例：
```shell
#立即关机
hscroot@hmc:~> hmcshutdown -t now
#等1分钟后关闭
hscroot@hmc:~> hmcshutdown -t 1
#等1分钟后重启
hscroot@hmc:~> hmcshutdown -t 1 -r
```
##### lshmc
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
##### bkconsdata
备份硬件控制台HMC重要数据,示例：
```shell
#ftp方式备份
hscroot@hmc:~> bkconsdata -r ftp -h ftpserver -u ftpuser --passwd ftppassword
#nfs方式备份
hscroot@hmc:~> bkconsdata -r nfs -h 192.168.1.10 -l /tmp/hmc/backups
#dvd方式备份
hscroot@hmc:~> bkconsdata -r dvd
```
##### chhmc
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
在HMC可以用命令行对受管小型机进行相应配置操作，这对于查看小型机信息和进行批量配置时候很有用。
##### lsdump
列出受管系统的dump信息，示例：
```shell
#列出所有dump
hscroot@hmc:~>lsdump -h
#列出指定受管设备的dump
hscroot@hmc:~>lsdump -m <managed system>
```
##### lshwres
列出受管系统的硬件资源，示例：
```shell
#列出物理I/O相关信息
lshwres -r io --rsubtype unit -m system1
lshwres -r io --rsubtype bus -m <managed system> --filter "units=U787A.001.*******"
lshwres -r io --rsubtype slot -m system1 --filter "units=U787A.001.*******,"buses=2,3"" -F drc_index,description,<lpar_name>
lshwres -r io --rsubtype iopool -m system1 --level pool
lshwres -r io --rsubtype taggedio -m <managed system> --filter "lpar_ids=1"
#列出虚拟适配器相关信息
lshwres -r virtualio --rsubtype eth --level lpar -m system1
lshwres -r virtualio --rsubtype scsi -m system1 -F --header
lshwres -r virtualio --rsubtype slot -m system1 --level slot --filter "lpar_names=lpar1"
#列出内存相关信息
lshwres -r mem -m <managed system> --level sys
lshwres -r mem -m <managed system> --level lpar -R
lshwres -r mem -m system1 --level lpar --filter ""lpar_names=lpar_1,lpar2""
#列出处理器相关信息
lshwres -r proc -m <managed system> --level sys -F installed_sys_proc_units:configurable_sys_proc_units
lshwres -r proc -m system1 --level lpar
#列出hca相关信息
lshwres -r hca -m <managed system> --level sys
lshwres -r hca -m <managed system> --level lpar --filter "lpar_names=lpar1"
lshwres -r sni -m system1
```
