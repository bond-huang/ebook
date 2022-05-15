# Redhat-常用命令
## 硬件相关
### cpu相关
#### 查看CPU信息
查看配置文件示例如下：
```
[root@redhat8 ~]# cat /proc/cpuinfo
processor	: 0
vendor_id	: AuthenticAMD
cpu family	: 23
model		: 96
model name	: AMD Ryzen 5 PRO 4650U with Radeon Graphics
...output omitted...
```
使用`lscpu`命令查看示例如下：
```
[root@redhat8 ~]# lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              2
On-line CPU(s) list: 0,1
...output omitted...
```
### 内存相关
#### 查看内存信息
命令如下：
```
[root@redhat8 ~]# cat /proc/meminfo
MemTotal:        1849432 kB
MemFree:          657832 kB
MemAvailable:    1148136 kB
Buffers:            2312 kB
Cached:           464084 kB
...output omitted...
```
#### free命令
命令示例如下：
```
[root@redhat8 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1849432      661268      657760        9776      530404     1148040
Swap:       2097148           0     2097148
[root@redhat8 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1806         645         642           9         517        1121
Swap:          2047           0        2047
```
### 网卡相关
#### 查看网卡信息
查看网口配置文件：
```
[root@redhat8 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens160
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens160"
UUID="b561c316-1e2c-4f92-9f28-fb4cc66f0c40"
DEVICE="ens160"
ONBOOT="yes"
```
## 待补充
