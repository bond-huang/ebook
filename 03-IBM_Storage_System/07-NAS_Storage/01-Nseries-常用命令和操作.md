# Nseries-基本操作
目前就接触过N3400和N6040，维护不多，操作不难但是长时间不用就忘了，记下来。
### 信息查看和收集
##### 基本检查
命令|用途
---|:---
rdfile /etc/messages|查看日志
environment status|查看系统环境状态
sysconfig -r|查看磁盘相关配置
sysconfig -a|查看所有配置
sysconfig -v|查看硬件状态
ifconfig -a |查看IP状态
vif status |查看VIF端口
aggr status |查看aggr状态
vol status |查看vol状态
lun status |查看lun状态
disk show -v|查看硬盘状态
cf status |查看cluster状态

##### 收集日志
确认autosupport状态是enable：
```
N3400b> options autosupport.enable
autosupport.enable           on 
```
触发最新autosupport信息
```
N3400b> options autosupport.doit 
```
同时收集/etc/messages*和/etc/log/auditlog文件。

### 硬盘更换
查看硬盘状态：
```
N3400b> disk show -v
  DISK       OWNER                  POOL   SERIAL NUMBER                          
  0b.38        N3400B    (84230013)   Pool0  ZAKUBKBH             
  0b.34        N3400A    (84228001)   Pool0  ZBGU7DUH             
  0b.35        N3400B    (84228013)   FAILED ZBG6H8NF   
```
查看所在槽位，可以看到位于1号柜子7号槽位：
```
aggr status -r
RAID Disk	Device	HA  SHELF BAY CHAN Pool Type  RPM  Used (MB/blks)    Phys (MB/blks)  failed  	    
0b.35 	0b    1   3   FC:A   -  ATA   7200 423111/866531584  423889/868126304 
```
拔出故障硬盘，换上新的硬盘，输入命令查看状态：
```
N3400b> disk show -v
  DISK       OWNER                  POOL   SERIAL NUMBER  
  0b.38        N3400B    (84230013)   Pool0  ZAKUBKBH             
  0b.34        N3400A    (84228001)   Pool0  ZBGU7DUH             
  0b.35        Not Owned              NONE   ZBBF08JG  
```
磁盘属于哪个控制器就在那个控制器下执行：
```
N3400b> disk assign 0b.35
N3400b> disk show -v 
  DISK       OWNER                  POOL   SERIAL NUMBER  
  0b.38        N3400B    (84230013)   Pool0  ZAKUBKBH             
  0b.34        N3400A    (84228001)   Pool0  ZBGU7DUH             
  0b.35        N3400B    (84230013)   Pool0  ZBBF08JG  
```
查看磁盘使用状态，可以看到是个spare备用盘：
```
N3400b> aggr status -r
```
更换完成。
