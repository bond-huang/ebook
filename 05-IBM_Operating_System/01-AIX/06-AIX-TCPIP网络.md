# AIX-TCPIP网络
## 配置IPv6
&#8195;&#8195;有客户近期需要配置IPv6，看了官网配置方案不难，给客户写了一份方案，这里也把方案记录下方便查阅，以后可能经常会用到。分两种情况，一种是已经有IPv4，一种是没有，两种配置差别不大，但是有些细节步骤不一样，还是分开写避免混淆。
### 从IPv4到IPv6的手动升级
当前系统已经配置了IPv4的情况下采用此方案。
#### 配置IPv6地址
输入命令查看当前配置：
```
bash-5.0# netstat -ni
Name  Mtu   Network     Address            Ipkts Ierrs    Opkts Oerrs  Coll
en0   1500  link#2      4e.8e.70.0.90.cf 403401863     0 408849341     0     0
en0   1500  9.200.104   9.200.104.23     403401863     0 408849341     0     0
lo0   16896 link#1                       151933830     0 151933834     0     0
lo0   16896 127         127.0.0.1        151933830     0 151933834     0     0
lo0   16896 ::1%1  
```
使用root用户权限，输入命令`autoconf6`来配置IPv6,然后查看配置：
```
bash-5.0# autoconf6
bash-5.0# netstat -ni
Name  Mtu   Network     Address            Ipkts Ierrs    Opkts Oerrs  Coll
en0   1500  link#2      4e.8e.70.0.90.cf 403917847     0 409372234     0     0
en0   1500  9.200.104   9.200.104.23     403917847     0 409372234     0     0
en0   1500  fe80::4c8e:70ff:fe00:90cf    403917847     0 409372234     0     0
sit0  1480  link#3      9.200.104.23            0     0        0     0     0
sit0  1480  ::9.200.104.23                      0     0        0     0     0
lo0   16896 link#1                       152128319     0 152128323     0     0
lo0   16896 127         127.0.0.1        152128319     0 152128323     0     0
lo0   16896 ::1%1 
```
输入命令启动ndpd-host守护进程：
```
bash-5.0# startsrc -s ndpd-host
```
#### 配置开机启用IPv6
重新启动计算机后，新配置的IPv6将被删除。要在每次重新启动时启用IPv6，操作步骤如下：

首先打开文件: /etc/rc.tcpip；

取消注释下列所示的行（对比当前配置）：
```
# Start up autoconf6 process
start /usr/sbin/autoconf6 "" 

# Start up ndpd-host daemon
start /usr/sbin/ndpd-host "$src_running"
```
在`start /usr/sbin/autoconf6 ""`行中加上 -A参数：
`start /usr/sbin/autoconf6 "" -A`

重新启动系统后，配置会自动配置。

#### 配置IPv6路由
使用root用户权限，输入命令`netstat -ni`查看已配置的IPv4；

然后输入命令`autoconf6`来配置IPv6,

输入以下命令，在属于两个子网的每个路由器的接口上手动配置全局地址（根据需求）：
```
bash-5.0# ifconfig en0 inet6 3ffe:0:0:aaaa::/64 eui64 alias
bash-5.0# ifconfig en1 inet6 3ffe:0:0:bbbb::/64 eui64 alias
```
要激活IPv6转发，输入如下命令：
```
bash-5.0# no -o ip6forwarding=1
```
启动ndpd-router守护进程：
```
bash-5.0# startsrc -s ndpd-router
```
#### 开机启用IPv6路由
首先打开文件: /etc/rc.tcpip；取消注释下面所示行（对比当前配置）：
```
# Start up autoconf6 process
start /usr/sbin/autoconf6 "" 
```
在上面注释的行后面加上如下内容（根据当前系统需求）：
```
# Configure global addresses for router
ifconfig en0 inet6 3ffe:0:0:aaaa::/64 eui64 alias
ifconfig en1 inet6 3ffe:0:0:bbbb::/64 eui64 alias
```
取消注释下面所示行（对比当前配置）：
```
# Start up ndpd-host daemon
start /usr/sbin/ndpd-host "$src_running"
```

### 未配置IPv4的时配置IPv6
#### 配置IPv6地址
使用root用户权限，输入命令`autoconf6 -A`来配置IPv6;

然后输入命令`netstat -ni`查看当前配置,输出示例如下：
```
Name  Mtu   Network     Address            Ipkts Ierrs    Opkts Oerrs  Coll
en0   1500  link#3      0.4.ac.17.b4.11          7     0       17     0     0
en0   1500  fe80::204:acff:fe17:b411             7     0       17     0     0
lo0   16896 link#1                             436     0      481     0     0
lo0   16896 127         127.0.0.1              436     0      481     0     0
lo0   16896 ::1                                436     0      481     0     0
```
输入命令启动ndpd-host守护进程：`startsrc -s ndpd-host`
#### 配置开机启用IPv6
重新启动计算机后，新配置的IPv6将被删除。要在每次重新启动时启用IPv6，操作步骤如下：

首先打开文件: /etc/rc.tcpip；

取消注释下列所示的行（对比当前配置）：
```
# Start up autoconf6 process
start /usr/sbin/autoconf6 "" 

# Start up ndpd-host daemon
start /usr/sbin/ndpd-host "$src_running"
```
在`start /usr/sbin/autoconf6 ""`行中加上 -A参数：
```
start /usr/sbin/autoconf6 "" -A
```
重新启动系统后，配置会自动配置。
#### 配置IPv6路由
1）	使用root用户权限，输入命令`autoconf6 -A`来配置IPv6：输出示例：
```
Name  Mtu   Network     Address            Ipkts Ierrs    Opkts Oerrs  Coll
en1   1500  link#2      0.6.29.dc.15.45          0     0        7     0     0
en1   1500  fe80::206:29ff:fedc:1545             0     0        7     0     0
en0   1500  link#3      0.4.ac.17.b4.11          7     0       17     0     0
en0   1500  fe80::204:acff:fe17:b411             7     0       17     0     0
lo0   16896 link#1                             436     0      481     0     0
lo0   16896 127         127.0.0.1              436     0      481     0     0
lo0   16896 ::1                                436     0      481     0     0
```
输入以下命令，在属于两个子网的每个路由器的接口上手动配置全局地址（根据需求）：
```
bash-5.0# ifconfig en0 inet6 3ffe:0:0:aaaa::/64 eui64 alias
bash-5.0# ifconfig en1 inet6 3ffe:0:0:bbbb::/64 eui64 alias
```
要激活IPv6转发，输入如下命令：
```
bash-5.0# no -o ip6forwarding=1
```
启动ndpd-router守护进程：
```
bash-5.0# startsrc -s ndpd-router
```
#### 开机启用IPv6路由
首先打开文件: /etc/rc.tcpip；取消注释下面所示行（对比当前配置）：
```
# Start up autoconf6 process
start /usr/sbin/autoconf6 ""
```
在`start /usr/sbin/autoconf6 ""`行中加上 -A参数：
```
start /usr/sbin/autoconf6 "" -A
```
在上面注释的行后面加上如下内容（根据当前系统需求）：
```
# Configure global addresses for router
ifconfig en0 inet6 3ffe:0:0:aaaa::/64 eui64 alias
ifconfig en1 inet6 3ffe:0:0:bbbb::/64 eui64 alias
```
取消注释下面所示行（对比当前配置）：
```
# Start up ndpd-host daemon
start /usr/sbin/ndpd-host "$src_running"
```
在引导时启用IP转发，运行如下命令：
```
no -r -o ip6forwarding=1
```
### 其它
要调出接口的子集，可以使用命令`autoconf6`的`-i`参数。示例显示接口en0和en1：
```
autoconf6 -i en0 en1
```
## 网络服务
### /etc/services文件
[AIX services File Format for TCP/IP](https://www.ibm.com/docs/en/aix/7.1?topic=formats-services-file-format-tcpip)
## 待补充

