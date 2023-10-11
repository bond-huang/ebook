# Redhat-常用命令
## 系统管理
查看系统版本：
```
[root@redhat8 ~]# cat /etc/redhat-release
Red Hat Enterprise Linux release 8.0 (Ootpa)
```
### 服务管理
#### systemctl命令
下表是服务管理实用命令：

命令|任务描述
:---|:---
systemctl status UNIT|查看有关单元状态的详细信息
systemctl stop UNIT|在运行中的系统上停止一项服务
systemctl start UNIT|在运行中的系统上启动一项服务	
systemctl restart UNIT|在运行中的系统上重新启动一项服务
systemctl reload UNIT|重新加载运行中服务的配置文件
systemctl mask UNIT|彻底禁用服务，使其无法手动启动或在系统引导时启动
systemctl unmask UNIT|使屏蔽的服务变为可用
systemctl enable UNIT|将服务配置为在系统引导时启动
systemctl disable UNIT|禁止服务在系统引导时启动
systemctl list-dependencies UNIT|列出指定单元需要的单元

### 网络相关命令
#### IP命令
`IP`命令常用命令列表：

命令|用途
:---|:---
ip add show name|查看设备name和地址信息
ip -s link show name|显示关于网络性能的统计信息
ip route|显示IPv4路由信息
ip -6 route|显示IPv6路由信息

#### nmcli命令
`nmcli`命令常用命令列表：

命令|用途
:---|:---
nmcli dev status|显示所有网络接口的NetworkManager状态
nmcli con show|列出所有连接
nmcli con show name|列出name连接的当前设置
nmcli con add con-name name|添加一个名为name的新连接
nmcli con mod name|修改name连接
nmcli con reload|重新加载配置文件(在手动编辑配置文件之后使用)
nmcli con up name|激活name连接
nmcli dev dis dev|在网络接口dev上停用并断开当前连接
nmcli con del name|删除name连接及其配置文件

## 软件管理
### RPM相关命令
`RPM`查询命令摘要如下表所示：

命令|任务
:---|:---
rpm -qa|列出当前安装的所有RPM软件包
rpm -q NAME|显示系统上安装的NAME版本
rpm -qi NAME|显示有关软件包的详细信息
rpm -ql NAME|列出软件包中含有的所有文件
rpm -qc NAME|列出软件包中含有的配置文件
rpm -qd NAME|列出软件包中含有的文档文件
rpm -q --changelog NAME|显示软件包新发行版的简短原因摘要
rpm -q --scripts NAME|显示在软件包安装、升级或删除时运行的shell脚本

### Yum命令
可以根据名称或软件包组，查找、安装、更新和删除软件包：

命令|任务
:---|:---
yum list \[NAME-PATTERN]|按名称列出已安装和可用的软件包
yum group list|列出已安装和可用的组
yum search KEYWORD|按关键字搜索软件包
yum info PACKAGENAME|显示软件包的详细信息
yum install PACKAGENAME|安装软件包
yum group install GROUPNAME|安装软件包组
yum update|更新所有软件包
yum remove PACKAGENAME|删除软件包
yum history|显示事务历史记录

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
## 文件相关
### wc命令
命令参数如下
- `-c`或`--bytes`：显示文件Bytes字节数
- `-m`或`--chars`, 显示文件字符个数
- `-l`或`--lines`：显示文件行数
- `-w`或`--words`：显示文件单词数
- `-L`或`--max-line-length`：显示文件中最长行数的长度
- `--help`：帮助
- `--version`：显示版本信息

### rsync
&#8195;&#8195;`rsync`是一个强大的文件同步工具，可以用于本地和远程文件同步。以下是一些基本的用法示例：
#### 文件同步
从源目录到目标目录同步文件：
```sh
rsync -av /源目录/ /目标目录/
```
该命令会递归同步源目录中的文件和子目录到目标目录，并保持文件权限和时间戳。

本地文件同步，删除目标目录多余文件：
```sh
rsync -av --delete /源目录/ /目标目录/
```
这将确保目标目录与源目录完全一致，删除目标目录中多余的文件。

#### SSH进行远程同步
同步本地目录到远程服务器：
```sh
rsync -av -e ssh /源目录/ user@remoteip:/目标目录/
```
示例说明：
- `user`是远程服务器的用户名
- `remoteip`是远程服务器的地址
- 使用SSH协议进行安全传输

从远程服务器同步到本地：
```sh
rsync -av -e ssh user@remote:/源目录/ /目标目录/
```
#### 排除文件或目录
使用`--exclude` 参数来排除特定文件或目录：
```sh
rsync -av --exclude '文件或目录名' /源目录/ /目标目录/
```
### 其他用法
可以根据需求使用不同的选项和参数，如`-r`用于递归，`-u`用于仅同步更新的文件等。

## 待补充
