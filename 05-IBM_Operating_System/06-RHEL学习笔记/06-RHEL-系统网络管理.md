# RHEL-系统网络管理
## 描述网络概念
### TCP/IP 网络模型
&#8195;&#8195;TCP/IP网络模型是一种简化的四层抽象集合，用于描述不同的协议如何进行互操作，以便计算机通过互联网将流量从一台计算机发送到另一台计算机。由`RFC 1122 互联网主机要求--通信层`规定。这四层是：
- 应用：每一应用程序具有用于通信的规范，以便客户端和服务器可以跨平台通信。常用的协议有SSH（远程登录）、HTTPS（安全Web）、NFS或CIFS（文件共享），以及SMTP（电子邮件递送）等。
- 传输：传输协议有TCP和UDP。应用协议使用TCP或UDP端口。`/etc/services`文件中可以找到常用和已注册的端口列表。协议及传输说明：
    - TCP是可靠连接导向型通信
    - UDP属于无连接数据报协议
    - 当数据包在网络上发送时，服务端口和IP地址组合形成套接字。每个数据包具有一个源套接字和目标套接字。此信息可以在监控和过滤时使用
- 互联网：互联网或网络层将数据从源主机传送到目标主机。IPv4和IPv6协议是互联网层协议。每一主机具有IP 地址和前缀，用于确定网络地址。路由器用于连接网络
- 链路：链路或介质存取层提供与物理介质的连接。最常见的网络类型是有线以太网(802.3)和无线局域网(802.11)。每一物理设备具有一个硬件地址(MAC)，用于标识局域网络段中数据包的目的地

### 描述网络接口名称
&#8195;&#8195;系统上的每个网络端口都有一个名称，可以使用该名称来配置和识别它。旧版RHEL将`eth0`、`eth1`和`eth2`等名称用于各个网络接口：
- 名称`eth0`是操作系统检测到的第一个网络端口，`eth1`则是第二个，以此类推。
- 随着设备的添加和移除，检测设备并给它们命名的机制可能会改变哪个接口获得哪个名称
- 此外，PCIe标准无法保证在启动时检测PCIe设备的顺序；鉴于设备或系统启动期间的变化，可能会意外改变设备命名

&#8195;&#8195;较新版本的RHEL采用另一种命名体系。系统将基于固件信息、PCI总线拓扑及网络设备的类型来分配网络接口名称，而非基于检测顺序。网络接口名称以接口类型开头：
- 以太网接口以`en`开头
- WLAN接口以`wl`开头
- WWAN接口以`ww`开头 

在类型之后，接口名称的其余部分将基于服务器固件所提供的信息，或由PCI拓扑中设备的位置确定：
- `oN`表示这是一个板载设备，且服务器的固件提供设备的索引编号`N`。例如`eno1`代表板载以太网设备`1`
- `sN`表示该设备位于PCI热插拔插槽`N`中。例如`ens3`代表PCI热插拔插槽`3`中的以太网卡
- `pMsN`表示这是一个位于插槽`N`中总线`M`上的PCI设备。例如`wlp4s0`代表位于插槽`0`中PCI总线`4`上的 `WLAN`卡：
    - 如果该卡是一个多功能设备（可能是有多个端口的以太网卡，或是具有以太网外加其他一些功能的设备），设备名称中就可能会添加`fN`
    - 例如`enp0s1f0`代表插槽`1`中总线`0`上的以太网卡的功能`0`。可能还有一个名为`enp0s1f1`的接口，它代表了同一设备的功能`1`

### IPv4网络
IPv4是当今互联网上使用的主要网络协议。
#### IPv4地址
&#8195;&#8195;`IPv4`地址是一个32位数字，通常使用点号分隔的四个十进制八位字节（取值范围从0到255）表示。此类地址分网络部分和主机部分：
- 位于同一子网中的所有主机可以在彼此之间直接通信，无需路由器，这些主机具有相同的网络部分
- 网络部分用于标识子网
- 同一子网中的任何两台主机都不能具有相同的主机部分
- 主机部分用于标识子网中的特定主机

&#8195;&#8195;在现代互联网中，`IPv4`子网的大小是可变的。要分清`IPv4`地址中的网络部分和主机部分，管理员必须知道分配给子网的子网掩码。子网掩码指明有多少位的IPv4地址属于子网。可供主机部分使用的位数越多，子网中就能有越多的主机：
- 有时，将子网中可能达到的最低地址（主机部分的二进制值全为零）称为网络地址
- 在IPv4中，子网中可能达到的最高地址（主机部分的二进制值全为一）用于广播消息，该地址称为广播地址

子网掩码可用两种格式表示：
- 较早的子网掩码语法中将24位用于网络部分，即`255.255.255.0`
- 较新的语法称为`CIDR`表示法，它指定了一个网络前缀`/24`。两种格式都传达同样的信息，即IP地址中有多少前导位组成其网络地址

&#8195;&#8195;特殊地址`127.0.0.1`始终指向本地系统`localhost`，而网络`127.0.0.0/8`属于本地系统，所以它能够使用网络协议与自己通信。
#### 地址计算示例
计算`192.168.1.108/24`的网络地址：

说明|十进制|二进制
:---:|:---:|:---:
主机地址|192.168.1.107|11000000.10101000.00000001.01101100
网络前缀|/24(255.255.255.0)|11111111.11111111.11111111.00000000
网络地址|192.168.1.0|11000000.10101000.00000001.00000000
广播地址|192.168.1.255|11000000.10101000.00000001.11111111

计算`10.1.1.18/8`的网络地址：

说明|十进制|二进制
:---:|:---:|:---:
主机地址|10.1.1.18|00001010.00000001.00000001.00010010
网络前缀|/8(255.0.0.0)|11111111.00000000.00000000.00000000
网络地址|10.0.0.0|00001010.00000000.00000000.00000000
广播地址|10.255.255.255|00001010.11111111.11111111.11111111

计算`172.16.180.20/19`的网络地址：

说明|十进制|二进制
:---:|:---:|:---:
主机地址|172.168.181.23|10101100.10101000.10110100.00010100
网络前缀|/19(255.255.224.0)|11111111.11111111.11100000.00000000
网络地址|172.168.160.0|10101100.10101000.10100000.00000000
广播地址|172.168.191.255|10101100.10101000.10111111.11111111

#### IPv4路由
&#8195;&#8195;不管使用`IPv4`还是`IPv6`，网络流量都需要以主机到主机和网络到网络的形式进行传输。每一主机具有一个路由表，该表告诉主机如何路由特定网络的通信：
- 路由表条目将列出目标网络、用于向其发送流量的接口，以及任何中间路由器的IP地址（用于将消息中继到最终目的地）。与网络流量目的地相符的路由表条目用于路由该流量。如果两个条目匹配，则使用前缀最长的那一个
- 如果网络流量不匹配更为具体的路由，路由表通常具有一个代表整个IPv4互联网的默认路由条目：`0.0.0.0/0`。此默认路由指向可通达的子网上的路由器（也就是说，在主机路由中具有更具体路由的子网上）
- 如果路由器收到的流量并非将其作为寻址目标，则路由器不会像普通主机那样忽略该流量，而是根据自己的路由表转发该流量。这种处理方式可能会将流量直接发送到目标主机（如果路由器恰巧与目标位于同一子网中），也可能转发到其他路由器。这种转发过程会一直进行，直到流量到达最终目标

路由表示例：
目的地|接口|路由器（若需要）
:---:|:---:|:---:
192.0.2.0/24|wlo1	 
192.168.5.0/24|enp3s0	 
0.0.0.0/0(默认)|enp3s0|192.168.5.254

示例说明：
- 从此主机发往IP地址`192.0.2.100`的流量将通过`wlo1`无线接口直接传输到该目的地，它与`192.0.2.0/24`路由的匹配度最高
- 发往IP地址`192.168.5.100`的流量将通过`enp3s0`以太网接口直接传输到该目的地，它与`192.168.5.0/24`路由的匹配度最高
- 发往IP地址`10.2.24.100`的流量将从`enp3s0`以太网接口发送到`192.168.5.254`的路由器，该路由器将该通信转发到其最终目的地。该流量与`0.0.0.0/0`路由的匹配度最高，因为此主机的路由表中没有更加具体的路由。该路由器将使用自身的路由表来判断流量需要转发到的下一个位置

#### IPv4地址和路由配置
配置方式：
- 可以在引导时从`DHCP`服务器自动配置其`IPv4`网络设置。本地客户端守护进程查询链路以获取服务器和网络设置，并获得租约以便在特定时间内使用这些设置。如果客户端未定期请求续订租约，则可能会丢失其网络配置设置
- 用户可以将服务器配置为使用静态网络配置。在这种情况下，网络设置读取自本地配置文件。用户必须从网络管理员处获取正确的设置，并根据需要手动更新它们，以避免与其他服务器冲突

### IPv6网络
&#8195;&#8195;`IPv6`旨在最终取代`IPv4`网络协议。`IPv6`可以在双栈模型中与`IPv4`并行使用，在这种配置中，网络接口可以同时具有`IPv6`地址和`IPv4`地址，RHEL默认在双栈模式下运行。
#### IPv6地址
&#8195;&#8195;`IPv6`地址是一个128位数字，通常表示为八组以分号分隔的四个十六进制半字节。每个半字节均表示`4`位的 `IPv6`地址，因此每个组表示`16`位的`IPv6`地址。示例：
```
2001:0db8:0000:0010:0000:0000:0000:0001
``` 
&#8195;&#8195;为了便于编写`IPv6`地址，不需要编写分号分隔的组中的前导零。但是，每个冒号分隔的组中必须至少写入一个十六进制数字。示例：
```
2001:db8:0:10:0:0:0:1
```
&#8195;&#8195;由于带有很长的零字符串的地址很常见，一组或多组连续零可以通过正好一个`::`块来合并。上面示例合并后示例如下：
```
2001:db8:0:10::1
``` 
有关编写始终可读的地址的一些提示如下：
- 抑制组中的前导零
- 使用`::`来尽可能地缩短
- 如果地址包含两个连续的零组，且长度相同，则最好将每个组最左边的零组缩短为`::`，最右边的组缩短为`:0:`
- 尽管允许这样做，但不要使用`::`来缩短一组零。应改为使用`:0:`，而将`::`留给连续的零组
- 始终对十六进制数字使用小写字母`a`到`f`

&#8195;&#8195;如果在`IPv6`地址后面包括TCP或UDP网络端口，请始终将`IPv6`地址括在方括号中，以便端口不会被误认为是地址的一部分。示例如下：
```
[2001:db8:0:10::1]:80
```
#### IPv6子网划分
&#8195;&#8195;普通的`IPv6`单播地址分为两部分：网络前缀和接口`ID`。网络前缀标识子网。同一子网上的任何两个子网接口都不能具有相同接口`ID`，接口`ID`可标识子网上的特定接口。   
&#8195;&#8195;与`IPv4`不同的是，`IPv6`具有一个标准的子网掩码`/64`，用于几乎所有的普通地址。在此情况下，地址的一半是网络前缀，另一半是接口`ID`。这意味着单个子网可以根据需要容纳任意数量的主机。    
&#8195;&#8195;通常，网络提供商将为组织分配一个较短的前缀，如`/48`。这会保留其余网络部分以用于通过这一分配的前缀来指定子网（长度始终为`/64`）。对于`/48`分配，将保留`16`位以用于子网（最多65536个子网）。

常用IPv6地址和网络：

IPv6地址或网络|用途|描述
:---:|:---:|:---
::1/128|本地主机|等效于127.0.0.1/8的IPv6，在回环接口上设置
::|未指定的地址|等效于0.0.0.0的IPv6。对于网络服务，这可能表示其正在侦听所有已配置的IP地址
::/0|默认路由（IPv6 互联网）|等效于 0.0.0.0/0 的IPv6。路由表中的默认路由与此网络匹配；此网络的路由器是在没有更好路由的情况下发送所有流量的位置
2000::/3|全局单播地址|“普通”的IPv6地址目前由IANA从该空间进行分配。等同于范围从`2000::/16`到 `3fff::/16`的所有网络
fd00::/8|唯一本地地址(RFC 4193)|IPv6没有RFC1918专用地址空间的直接等效对象，尽管这很接近。站点可以使用这些以在组织中自助分配可路由的专用IP地址空间，但是这些网络不能在全局Internet上使用。站点必须随机从该空间中选择一个/48，但是它可以正常将分配空间划分为/64网络
fe80::/10|本地链路地址|每个IPv6接口自动配置一个本地链路地址，该地址仅在fe80::/64网络中的本地链路有效。但是，整个fe80::/10范围保留供本地链路以后使用
ff00::/8|多播|等效于224.0.0.0/4的IPv6。多播用于同时传输到多个主机，并且在IPv6中特别重要，因为其没有广播地址

&#8195;&#8195;`IPv6`中的本地链路地址是一个无法路由的地址，仅用于与特定网络链路上的主机进行通信。系统上的每个网络接口都通过`fe80::/64`网络上的本地链路地址来自动配置。为确保其唯一性，本地链路地址的接口 `ID`是通过网络接口的以太网硬件地址来构建的。将`48`位`MAC`地址转换为`64`位接口`ID`的一般方法是反转`7`位的`MAC`地址，然后在其两个中间字节之间插入`ff:fe`。
- 网络前缀：fe80::/64
- MAC 地址：00:11:22:aa:bb:cc
- 本地链路地址：fe80::211:22ff:feaa:bbcc/64 

&#8195;&#8195;其他计算机的本地链路地址可以由相同链路上的其他主机像普通地址那样使用。由于每个链路具有`fe80::/64`网络，不能使用路由表来正确地选择出站接口。在地址的结尾必须使用作用域标识符来指定与本地链路地址进行通信时使用的链路。作用域标识符由`%`以及后跟的网络接口名称组成。

&#8195;&#8195;要使用`ping6`对本地链路地址`fe80::69af:c449:76d3:770`进行ping操作（使用连接到ens160网络接口的链路），正确的命令语法及示例如下所示： 
```
[root@redhat8 ~]# ping6 fe80::69af:c449:76d3:770%ens160
PING fe80::69af:c449:76d3:770%ens160(fe80::69af:c449:76d3:770%ens160) 56 data bytes
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=2 ttl=64 time=0.180 ms
^C
--- fe80::69af:c449:76d3:770%ens160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 0.066/0.123/0.180/0.057 ms
```
&#8195;&#8195;多播允许一个系统将流量发送到多个系统接收的特殊IP地址。它与广播不同，因为只有网络上的特定系统才能接收流量。它也与`IPv4`中的广播不同，因为某些多播流量可能会路由到其他子网，具体取决于网络路由器和系统的配置：
- 多播在`IPv6`中比在`IPv4`中扮演着更重要的角色，因为`IPv6`中没有广播地址
- `IPv6`中的一个重要多播地址是`ff02::1`，即`all-nodes`本地链路址。对此地址进行 Ping 操作会将流量都发送到链路上的所有节点
- 与本地链路地址一样，需要使用作用域标识符来指定链路作用域多播地址（从ff02::/8开始） 

示例如下：
```
[root@redhat8 ~]# ping6 ff02::1%ens160
PING ff02::1%ens160(ff02::1%ens160) 56 data bytes
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=1 ttl=64 time=0.080 ms
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=2 ttl=64 time=0.358 ms
^C
--- ff02::1%ens160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 4ms
rtt min/avg/max/mdev = 0.080/0.219/0.358/0.139 ms
```
#### IPv6地址配置
&#8195;&#8195;IPv6也支持手动配置以及两种动态配置方法，其中一种便是`DHCPv6`。与IPv4一样，可以随意选择静态`IPv6`地址的接口`ID`。在`IPv4`中，网络上有两个地址无法使用：子网中的最低地址和子网中的最高地址。在`IPv6`中，以下接口`ID`是保留的，无法用于主机上的普通网络地址。
- 由链路上的所有路由器使用的全零标识符`0000:0000:0000:0000`（“子网路由器任意广播”）。（对于 `2001:db8::/64`网络，这可能是地址`2001:db8::`）
- 标识符`fdff:ffff:ffff:ff80`到`fdff:ffff:ffff:ffff`

由于没有广播地址，`DHCPv6`的工作原理与适用于IPv4的DHCP有所不同：
- 主机将`DHCPv6`请求从其本地链路地址发送到`ff02::1:2`上的端口`547/UDP`，即`all-dhcp-servers`本地链路多播组
- 然后`DHCPv6`服务器通常向客户端的本地链路地址上的端口`546/UDP`发送一个包含相应信息的回复。

除了`DHCPv6`之外，`IPv6`也支持另外一个动态配置方法，称为无状态地址自动配置(SLAAC)：
- 使用`SLAAC`时，主机通常使用本地链路`fe80::/64`地址来调出其接口
- 主机随后向`ff02::2`（即全路由器本地链路多播组）发送一个“路由器请求”
- 本地链路上的`IPv6`路由器以网络前缀以及其他可能的信息来响应主机的本地链路地址
- 主机随后将该网络前缀与其通常构建的接口ID（构建方式与本地链路地址相同）配合使用
- 路由器定期发送多播更新（“路由器播发”）以确认或更新其提供的信息

&#8195;&#8195;红帽企业Linux中的`radvd`软件包允许基于RHEL的`IPv6`路由器通过路由器播发提供`SLAAC`。`SLAAC`注意事项如下：
- 配置为通过`DHCP`获取`IPv4`地址的典型RHEL8计算机通常也配置为使用`SLAAC`来获取`IPv6`地址。当网络中添加了`IPv6`路由器时，这可能导致计算机意外获取`IPv6`地址
- 部分`IPv6`部署将`SLAAC`与`DHCPv6`组合，`SLAAC`仅用于提供网络地址信息，而`DHCPv6`仅用于提供其他信息（如要配置的DNS服务器和搜索域）

### 主机名和IP地址
&#8195;&#8195;如果用户总是必须使用IP地址连接用户的服务器，这会很不方便。人们通常更愿意使用名称而不是一长串难记的数字。因此，Linux有多种机制可以将主机名映射到IP地址，统称为名称解析。
- 一种方法是在各个系统上的`/etc/hosts`文件中为每个名称设置一个静态条目。这需要手动更新每台服务器的文件副本
- 对于大多数主机，可以借助称为域名系统(DNS)的网络服务，从主机名查找地址（或从地址查找主机名）：
    - DNS是提供主机名到IP地址映射的分布式服务器网络
    - 为使名称服务起作用，主机需要指向某一个名称服务器。该名称服务器只需可供主机访问即可。这通常通过`DHCP`或`/etc/resolv.conf`文件中的静态设置来配置

## 验证网络配置
### 收集网络接口信息
#### 识别网络接口
`ip link`命令将列出系统上可用的所有网络接口：
```
[root@redhat8 ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT gro
up default qlen 1000    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT
 group default qlen 1000    link/ether 00:0c:29:ee:ed:0e brd ff:ff:ff:ff:ff:ff
```
示例中，服务器有两个网络接口：
- `lo`：这是连接到服务器本身的环回设备
- `ens160`：一个以太网接口

#### 显示IP地址
使用`ip`命令来查看设备和地址信息，`ip add show ens160`命令输出示例：
```shell
# 1
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group defaul
t qlen 1000    
# 2
link/ether 00:0c:29:ee:ed:0e brd ff:ff:ff:ff:ff:ff
# 3
    inet 192.168.100.131/24 brd 192.168.100.255 scope global dynamic noprefixroute e
ns160       valid_lft 900sec preferred_lft 900sec
# 4
    inet6 fe80::69af:c449:76d3:770/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
单个网络接口可以具有多个IPv4或IPv6地址。示例中四个部分说明：
- 1：活动接口为UP
- 2：link/ether行指定设备的硬件(MAC)地址
- 3：inet行显示IPv4地址、其网络前缀长度和作用域
- 4：该inet6行显示接口具有链路作用域的IPv6地址，并且只能用于本地以太网链路上的通信：
    - 如果为`scope global`,此inet6行显示IPv6地址、其网络前缀长度和作用域。此地址属于全局作用域，通常使用此地址

#### 显示性能统计信息
&#8195;&#8195;`ip`命令也可用于显示关于网络性能的统计信息。每个网络接口的计数器可用于识别网络问题的存在。计时器记录的统计信息包括收到(RX)和传出(TX)的数据包数、数据包错误数，以及丢弃的数据包数。示例如下：
```
[root@redhat8 ~]# ip -s link show ens160
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT
 group default qlen 1000    link/ether 00:0c:29:ee:ed:0e brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    382475257  278554   0       0       0       3614    
    TX: bytes  packets  errors  dropped carrier collsns 
    2884634    36418    0       0       0       0  
```
### 检查主机之间的连接
&#8195;&#8195;`ping`命令可用于测试连接。该命令将持续运行，直到按下`Ctrl+c`（除非已指定了限制发送数据包数量的选项）。示例如下：
```
PING 192.168.100.131 (192.168.100.131) 56(84) bytes of data.
64 bytes from 192.168.100.131: icmp_seq=1 ttl=64 time=0.050 ms
64 bytes from 192.168.100.131: icmp_seq=2 ttl=64 time=0.144 ms
^C
--- 192.168.100.131 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 28ms
rtt min/avg/max/mdev = 0.050/0.097/0.144/0.047 ms
```
&#8195;&#8195;`ping6`命令是RHEL中ping的`IPv6`版本。它通过`IPv6`进行通信，并且采用`IPv6`地址，但是其他工作方式类似于`ping`。示例如下：
```
[root@redhat8 ~]# ping6 fe80::69af:c449:76d3:770
PING fe80::69af:c449:76d3:770(fe80::69af:c449:76d3:770) 56 data bytes
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=1 ttl=64 time=0.089 ms
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=2 ttl=64 time=0.092 ms
^C
--- fe80::69af:c449:76d3:770 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 19ms
rtt min/avg/max/mdev = 0.089/0.090/0.092/0.009 ms
```
&#8195;&#8195;当`ping`本地链路地址和本地链路全节点多播组(ff02::1) 时，必须使用作用域标识符（如 `ff02::1%ens3`）来显式指定要使用的网络接口。如果遗漏，则将显示错误`connect: Invalid argument`。示例如下：
```
[root@redhat8 ~]# ping6 ff02::1%ens160
PING ff02::1%ens160(ff02::1%ens160) 56 data bytes
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=1 ttl=64 time=0.084 ms
64 bytes from fe80::69af:c449:76d3:770%ens160: icmp_seq=2 ttl=64 time=0.198 ms
^C
--- ff02::1%ens160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 56ms
rtt min/avg/max/mdev = 0.084/0.141/0.198/0.057 ms
```
跟普通地址一样，`IPv6`本地链路地址可以由同一链路上的其他主机使用：
```
[root@redhat8 ~]# ssh fe80::69af:c449:76d3:770%ens160
The authenticity of host 'fe80::69af:c449:76d3:770%ens160 (fe80::69af:c449:76d3:770%
ens160)' can't be established.ECDSA key fingerprint is SHA256:pCN6N5A3ex61KwcBYkR8sUiNKBE/hPO5c4FJkvGIJhY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'fe80::69af:c449:76d3:770%ens160' (ECDSA) to the list of 
known hosts.root@fe80::69af:c449:76d3:770%ens160's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed May 25 01:29:25 2022 from 192.168.100.1
```
### 路由故障排除
#### 显示路由表
使用`ip`命令及`route`选项来显示路由信息：
```
Connection to fe80::69af:c449:76d3:770%ens160 closed.
[root@redhat8 ~]# ip route
default via 192.168.100.2 dev ens160 proto dhcp metric 100 
192.168.100.0/24 dev ens160 proto kernel scope link src 192.168.100.131 metric 100 
```
示例说明：
- 示例显示`IPv4`路由表
- 目标为`192.168.100.0/24`网络的所有数据包将通过设备`ens160`直接发送到该目标位置
- 所有其他数据包将发送到位于`192.168.100.2`的默认路由器，而且也通过设备`ens160`传输

添加`-6`选项可显示`IPv6`路由表，示例如下：
```
[root@redhat8 ~]# ip -6 route
::1 dev lo proto kernel metric 256 pref medium
fe80::/64 dev ens160 proto kernel metric 100 pref medium
```
#### 追踪流量采用的路由
&#8195;&#8195;要追踪网络流量通过多个路由器到达远程主机而采用的路径，可使用`traceroute`或`tracepath`命令。这可以识别用户的某个路由器或中间路由器是否存在问题：
- 默认情况下，两个命令都使用UDP数据包来追踪路径；
- 许多网络阻止`UDP`和`ICMP`流量。`traceroute`命令拥有可以跟踪`UDP`（默认）、`ICMP`(-I) 或`TCP` (-T)数据包路径的选项
- 默认情况下通常不安装`traceroute`命令

示例如下：
```
[root@redhat8 ~]# tracepath 115.152.254.112
 1?: [LOCALHOST]                      pmtu 1500
 1:  localhost                                             0.474ms 
 1:  localhost                                             0.307ms 
 2:  no reply
...output omitted...
30:  no reply
     Too many hops: pmtu 1500
     Resume: pmtu 1500 
```
命令及示例说明：
- `tracepath`输出中的每一行表示数据包在来源和最终目标位置之间所经过的路由器或跃点
- 命令也提供了其他信息，如往返用时(RTT)和最大传输单元(MTU)大小中的任何变化等
- `asymm`表示流量使用了不同的(非对称)路由到达该路由器的流量和从该路由器返回
- 显示的路由器是用于出站流量的路由器，而不是返回流量
- `tracepath6`和`traceroute -6`命令等效于`IPv6`版本的`tracepath`和`traceroute`

### 端口和服务故障排除
&#8195;&#8195;TCP服务使用套接字作为通信的端点，其由IP地址、协议和端口号组成。服务通常侦听标准端口，而客户端则使用随机的可用端口。`/etc/services`文件中列出了标准端口的常用名称。    
&#8195;&#8195;`ss`命令可用于显示套接字统计信息。`ss`命令旨在替换`net-tools`软件包中所含的较旧工具 `netstat`（这个工具可能更为某些系统管理员所熟知，但未必始终安装在系统中）。示例如下：
```
[root@redhat8 ~]# ss -ta
State  Recv-Q   Send-Q        Local Address:Port                Peer Address:Port   
LISTEN 0        128                 0.0.0.0:ssh                      0.0.0.0:*      
LISTEN 0        128               127.0.0.1:x11-ssh-offset           0.0.0.0:*      
LISTEN 0        128                 0.0.0.0:sunrpc                   0.0.0.0:*      
ESTAB  0        0           192.168.100.131:ssh                192.168.100.1:52684  
LISTEN 0        32                        *:ftp                            *:*      
LISTEN 0        128                    [::]:ssh                         [::]:*      
LISTEN 0        128                   [::1]:x11-ssh-offset              [::]:*      
LISTEN 0        128                    [::]:sunrpc                      [::]:*  
```

命令`ss`和`netstat`的选项：

选项|描述
:---:|:---
-n|显示接口和端口的编号，而不显示名称
-t|显示TCP套接字
-u|显示UDP套接字
-l|仅显示侦听中的套接字
-a|显示所有（侦听中和已建立的）套接字
-p|显示使用套接字的进程
-A inet|对于inet地址系列，显示活动的连接(但不显示侦听套接字)，忽略本地UNIX域套接字。对于ss，同时显示IPv4和IPv6连接。对于netstat ，仅显示IPv4连接(netstat -A inet6显示IPv6连接，netstat -46则同时显示IPv4和IPv6)

## 从命令行配置网络
### NetworkManager概念
&#8195;&#8195;`NetworkManager`是监控和管理网络设置的守护进程。除了该守护进程外，还有一个提供网络状态信息的`GNOME`通知区域小程序。命令行和图形工具与`NetworkManager`通信，并将配置文件保存在`/etc/sysconfig/network-scripts`目录中：
- 设备是网络接口
- 连接是可以为设备配置的设置的集合
- 对于任何一个设备，在同一时间只能有一个连接处于活动状态。可能存在多个连接，以供不同设备使用或者以便为同一设备更改配置。如果需要临时更改网络设置，而不是更改连接的配置，可以更改设备的哪个连接处于活动状态
- 每个连接具有一个用于标识自身的名称或ID
- `nmcli`实用程序可用于从命令行创建和编辑连接文件

### 查看联网信息
`nmcli dev status`命令可显示所有网络设备的状态：
```
[root@redhat8 ~]# nmcli dev status
DEVICE  TYPE      STATE      CONNECTION 
ens160  ethernet  connected  ens160     
lo      loopback  unmanaged  --   
```
`nmcli con show`命令可显示所有连接的列表。使用`--active`列出活动的连接：
```
[root@redhat8 ~]# nmcli con show
NAME    UUID                                  TYPE      DEVICE 
ens160  b561c316-1e2c-4f92-9f28-fb4cc66f0c40  ethernet  ens160 
[root@redhat8 ~]# nmcli con show --active
NAME    UUID                                  TYPE      DEVICE 
ens160  b561c316-1e2c-4f92-9f28-fb4cc66f0c40  ethernet  ens160 
```
### 添加网络连接
`nmcli con add`命令用于添加新的网络连接。示例：
```
[root@redhat8 ~]# nmcli con add con-name ens160 type ethernet ifname ens160
Connection 'ens160' (09521237-47ec-484f-8df8-db071b8ccd7a) successfully added.
```
示例说明：
- 示例将为接口`ens160`添加一个新连接`ens160`，此连接将使用`DHCP`获取`IPv4`联网信息并在系统启动后自动连接
- 示例还将通过侦听本地链路上的路由器播发来获取`IPv6`联网设置
- 配置文件的名称基于的`con-name`选项的值`ens160`，并保存到`/etc/sysconfig/network-scripts/ifcfg-ens160`文件

&#8195;&#8195;下面示例使用静态`IPv4`地址为`ens160`设备创建`ens160`连接，使用`IPv4`地址和网络前缀`192.168.100.131/24`及默认网关`192.168.100.254`，仍在启动时自动连接并将其配置保存到相同文件中：
```
[root@redhat8 ~]# nmcli con add con-name ens160 type ethernet \
ifname ens160 ipv4.address 192.168.100.131/24 ipv4.gateway 192.168.100.254
Connection 'ens160' (ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c) successfully added.
[root@redhat8 ~]# nmcli con show
NAME    UUID                                  TYPE      DEVICE 
ens160  ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c  ethernet  ens160 
```
&#8195;&#8195;示例使用静态`IPv6`和`IPv4`地址为`eno2`设备创建`eno2`连接，且使用`IPv6`地址和网络前缀`2001:db8:0:1::c000:207/64`及默认`IPv6`网关`2001:db8:0:1::1`，以及`IPv4`地址和网络前缀`192.0.2.7/24`及默认`IPv4`网关`192.0.2.1`，在启动时自动连接，并将其配置保存到`/etc/sysconfig/network-scripts/ifcfg-eno2`：
```
nmcli con add con-name eno2 type ethernet ifname eno2 \
ipv6.address 2001:db8:0:1::c000:207/64 ipv6.gateway 2001:db8:0:1::1 \
ipv4.address 192.0.2.7/24 ipv4.gateway 192.0.2.1
```
### 控制网络连接
&#8195;&#8195;`nmcli con up name`命令将在其绑定到的网络接口上激活`name`连接。注意，命令采用连接的名称，而非网络接口的名称。`nmcli con show`命令显示所有可用连接的名称。示例：
```
[root@redhat8 ~]# nmcli con up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManage
r/ActiveConnection/6)
```
&#8195;&#8195;`nmcli dev disconnect device`命令将断开与网络接口`device`的连接并将其关闭。此命令可以缩写为`nmcli dev dis device`。示例如下：
```
[root@redhat8 ~]# nmcli dev dis ens160
Device 'ens160' successfully disconnected.
```
&#8195;&#8195;使用`nmcli dev dis device`可停用网络接口。`nmcli con down name`命令可以关闭连接，但通常并非是停用网络接口的最佳方法。示例如下：
```
[root@redhat8 ~]# nmcli con down ens160
Connection 'ens160' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/8)
[root@redhat8 ~]# nmcli con show
NAME    UUID                                  TYPE      DEVICE 
ens160  ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c  ethernet  --   
```
说明：
- 默认情况下，大部分有线系统连接是在启用了`autoconnect`的情况下配置的。这将在其网络接口可用后立即激活连接
- 由于连接的网络接口仍可用，因此`nmcli con down name`将关闭接口，但是`NetworkManager`会立即将其重新开启，除非连接完全与接口断开

### 修改网络连接设置
`NetworkManager`连接具有两种类型的设置。有静态连接属性，它们是由管理员配置并存储在`/etc/sysconfig/network-scripts/ifcfg-*`中的配置文件中。还可能有活动连接数据，这些数据是连接从`DHCP`服务器获取的，不会持久存储。`nmcli con show name`命令列出某个连接的当前设置，小写的设置是静态属性，管理员可以更改全大写的设置是活动设置，临时用于此连接实例：
```
[root@redhat8 ~]# nmcli con show ens160
connection.id:                          ens160
connection.uuid:                        ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens160
connection.autoconnect:                 yes
connection.autoconnect-priority:        0
...output omitted...
GENERAL.NAME:                           ens160
GENERAL.UUID:                           ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c
GENERAL.DEVICES:                        ens160
GENERAL.STATE:                          activated
GENERAL.DEFAULT:                        yes
GENERAL.DEFAULT6:                       no
GENERAL.SPEC-OBJECT:                    --
GENERAL.VPN:                            no
...output omitted...
```
&#8195;&#8195;`nmcli con mod name`命令可用于更改连接的设置，更改保存在对应的`/etc/sysconfig/network-scripts/ifcfg-name`文件中。示例将`ens160`连接将`IPv4`地址设置为`192.160.100.130/24`：
```
[root@redhat8 ~]# nmcli con mod ens160 ipv4.address 192.160.100.130/24
[root@redhat8 ~]# ip add show ens160
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group defaul
t qlen 1000    link/ether 00:0c:29:ee:ed:0e brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.130/24 brd 192.168.100.255 scope global dynamic noprefixroute e
ns160       valid_lft 1491sec preferred_lft 1491sec
    inet 192.168.100.131/24 brd 192.168.100.255 scope global secondary noprefixroute
 ens160       valid_lft forever preferred_lft forever
    inet6 fe80::78cd:cdad:d838:916f/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
&#8195;&#8195;更改后，之前`192.168.100.131`的连接未断开，`down`关闭接口后再`up`，`192.168.100.131`的连接就断开，更改完成。示例更改`IPv6`地址设置：
```
nmcli con mod static-ens3 ipv6.address 2001:db8:0:1::a00:1/64 \
ipv6.gateway 2001:db8:0:1::1
```
重要说明：
-  如果某个连接之前通过`DHCPv4`服务器获取其`IPv4`信息，而现在更改为仅通过静态配置文件来获取，那么设置`ipv4.method`也应从`auto`更改为`manual`
- 如果某个连接之前通过`SLAAC`或`DHCPv6`服务器获取其`IPv6`信息，而现在更改为仅通过静态配置文件来获取，那么`ipv6.method`设置也应从`auto`或`dhcp`更改为`manual`
- 如果不更改，连接在激活后可能挂起或者无法成功完成，或者除了静态地址外还从`DHCP`获取`IPv4`地址或从 `SLAAC`或`DHCPv6`获取`IPv6`地址
- 很多设置可能具有多个值。通过向设置名称的开头添加`+`或`-`符号，可以在列表中添加或从列表中删除特定值

### 删除网络连接
&#8195;&#8195;`nmcli con del name`命令将从系统中删除名为 name 的连接，同时断开它与设备的连接并删除文件`/etc/sysconfig/network-scripts/ifcfg-name`。示例如下：
```
[root@redhat8 ~]# nmcli con del ens160
Connection 'ens160' (ff73ccde-8a3c-42aa-a2c1-8a60a01c1a9c) successfully deleted.
[root@redhat8 ~]# ls -l /etc/sysconfig/network-scripts/
total 0
```
### 修改网络设置权限
修改网络设置说明：
- `root`用户可以使用`nmcli`对网络配置进行任何必要的更改
- 在本地控制台上登录的普通用户也可以对系统进行多项网络配置更改。要获得此控制权，必须在系统键盘上登录基于文本的虚拟控制台或图形桌面环境。背后的逻辑是：
    - 如果某人实际出现在计算机控制台上，就说明该计算机可能被用作工作站或笔记本电脑，因此他们可能需要随意配置、激活和停用无线或有线网络接口
    - 反之，如果系统是数据中心中的服务器，则通常以本地方式登录到计算机本身的用户只能是管理员
- 使用`ssh`登录的普通用户在成为`root`之前无权更改网络权限
- 可以使用`nmcli gen permissions`命令来查看自己的当前权限

### 命令摘要
下表是此次学习中关键`nmcli`命令的列表：

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

## 编辑网络配置文件
### 描述连接配置文件
&#8195;&#8195;默认情况下，通过`nmcli con mod name`进行的更改会自动保存到`/etc/sysconfig/network-scripts/ifcfg-name`。还可以使用文本编辑器手动编辑此文件。执行此操作后，运行`nmcli con reload`以便`NetworkManager`读取配置更改。出于向后兼容性的原因，此文件中保存的指令具有不同于 `nm-settings`名称的名称和语法。下表将部分关键设置名称映射到`ifcfg-*`指令。`nm-settings`与 `ifcfg-*`指令的比较：

nmcli con mod|ifcfg-* file|影响
:---|:---|:---
ipv4.method manual|BOOTPROTO=none|IPv4以静态方式配置
ipv4.method auto|BOOTPROTO=dhcp|从DHCPv4服务器中查找配置设置。如果还设置了静态地址，则在从DHCPv4中获取信息之前，将不会激活这些静态地址
ipv4.addresses 192.0.2.1/24|IPADDR=192.0.2.1 PREFIX=24|设置静态IPv4地址和网络前缀。如果为连接设置了多个地址，则第二个地址由IPADDR1和PREFIX1指令定义，以此类推
ipv4.gateway 192.0.2.254|GATEWAY=192.0.2.254|设置默认网关
ipv4.dns 8.8.8.8|DNS1=8.8.8.8|修改/etc/resolv.conf以使用此nameserver
ipv4.dns-search example.com|DOMAIN=example.com|修改/etc/resolv.conf，以在search指令中使用这个域
ipv4.ignore-auto-dns true|PEERDNS=no|忽略来自DHCP服务器的DNS服务器信息
ipv6.method manual|IPV6_AUTOCONF=no|IPv6 地址以静态方式配置
ipv6.method auto|IPV6_AUTOCONF=yes|使用路由器播发中的SLAAC来配置网络设置
ipv6.method dhcp|IPV6_AUTOCONF=no DHCPV6C=yes|使用DHCPv6（而不使用 SLAAC）来配置网络设置
ipv6.addresses 2001:db8::a/64|IPV6ADDR=2001:db8::a/64|设置静态IPv6地址和网络前缀。如果为连接设置了多个地址，IPV6ADDR_SECONDARIES将采用空格分隔的地址/前缀定义的双引号列表
ipv6.gateway 2001:db8::1|IPV6_DEFAULTGW=2001:...|设置默认网关。
ipv6.dns fde2:6494:1e09:2::d|DNS1=fde2:6494:...|修改/etc/resolv.conf以使用此名称服务器。与IPv4完全相同
ipv6.dns-search example.com|IPV6_DOMAIN=example.com|修改/etc/resolv.conf，以在search指令中使用这个域
ipv6.ignore-auto-dns true|IPV6_PEERDNS=no|忽略来自DHCP服务器的DNS服务器信息
connection.autoconnect yes|ONBOOT=yes|在系统引导时自动激活此连接
connection.id ens3|NAME=ens3|此连接的名称
connection.interface-name ens3|DEVICE=ens3|连接与具有此名称的网络接口绑定
802-3-ethernet.mac-address ...|HWADDR=...|连接与具有此MAC地址的网络接口绑定

### 修改网络配置
&#8195;&#8195;可以通过直接编辑连接配置文件来配置网络。连接配置文件控制单个网络设备的软件接口，这些文件通常命名为`/etc/sysconfig/network-scripts/ifcfg-name`。以下是在用于静态或动态 IPv4 配置的文件中找到的标准变量：
- 静态：
    - BOOTPROTO=none
    - IPADDR0=172.25.250.10
    - PREFIX0=24
    - GATEWAY0=172.25.250.254
    - DEFROUTE=yes
    - DNS1=172.25.254.254
- 动态灵活：BOOTPROTO=dhcp
- 任意：
    - DEVICE=ens3
    - NAME="static-ens3"
    - ONBOOT=yes
    - UUID=f3e8(...)ad3e
    - USERCTL=yes

&#8195;&#8195;在静态设置中，IP地址、前缀和网关等变量的末尾都是数字。这允许将多组值指定到该接口。DNS 变量也有一个数字，用于在指定了多个服务器时指定查询的顺序。在修改了配置文件后，运行`nmcli con reload`使`NetworkManager`读取配置更改。接口依然需要重新启动，以便更改生效。示例如下：
```
[root@redhat8 ~]# nmcli con reload
[root@redhat8 ~]# nmcli con down ens160
Connection 'ens160' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/13)
[root@redhat8 ~]# nmcli con up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/14)
```
## 配置主机名和名称解析
### 更改系统主机名
`hostname`命令显示或临时修改系统的完全限定主机名：
```
[root@redhat8 ~]# hostname
redhat8
```
&#8195;&#8195;可以在`/etc/hostname`文件中指定静态主机名。`hostnamectl`命令用于修改此文件，也可用于查看系统的完全限定主机名的状态。如果此文件不存在，则主机名在接口被分配了`IP`地址时由反向`DNS`查询设定。示例如下：
```
[root@redhat8 ~]# hostnamectl status
   Static hostname: redhat8
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 0d4f35dc451749ecad3ae466d7cbf09a
           Boot ID: 6d74b4812217402dadc582db9d479dd0
    Virtualization: vmware
  Operating System: Red Hat Enterprise Linux 8.0 (Ootpa)
       CPE OS Name: cpe:/o:redhat:enterprise_linux:8.0:GA
            Kernel: Linux 4.18.0-80.el8.x86_64
      Architecture: x86-64
[root@redhat8 ~]# hostnamectl set-hostname redhat8
```
### 配置名称解析
&#8195;&#8195;根解析器用于将主机名称转换为`IP`地址，反之亦可。它将根据`/etc/nsswitch.conf`文件的配置来确定查找位置。默认情况下，先检查`/etc/hosts`文件的内容。 示例如下：
```
[root@redhat8 ~]# cat /etc/hosts
[root@redhat8 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
115.152.254.112	access.redhat.com
```
&#8195;&#8195;可以通过`getent hosts hostname`命令，利用`/etc/hosts`文件测试主机名解析。使用`ping`命令尝试解析。示例如下：
```
[root@redhat8 ~]# getent hosts access.redhat.com
115.152.254.112 access.redhat.com
[root@redhat8 ~]# ping access.redhat.com
PING access.redhat.com (115.152.254.112) 56(84) bytes of data.
64 bytes from access.redhat.com (115.152.254.112): icmp_seq=1 ttl=128 time=8.16 ms
64 bytes from access.redhat.com (115.152.254.112): icmp_seq=2 ttl=128 time=8.66 ms
^C
--- access.redhat.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 8.158/8.410/8.663/0.268 ms
```
&#8195;&#8195;如果在`/etc/hosts`文件中未找到条目，默认情况下，根解析器会尝试使用`DNS`名称服务器来查询主机名。`/etc/resolv.conf`文件控制如何执行这一查询：
- `search`：对于较短主机名尝试搜索的域名列表。不应在同一文件中设置此参数和`domain`，如果在同一文件中设置它们，将使用最后一个实例
- `nameserver`：要查询的名称服务器的`IP`地址。可以指定最多三个名称服务器指令，以在其中一个名称服务器停机时提供备用名称服务器

`/etc/resolv.conf`文件示例如下：
```
[root@redhat8 ~]# cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain
nameserver 192.168.100.2
```
&#8195;&#8195;`NetworkManager`将使用连接配置文件中的DNS设置更新`/etc/resolv.conf`文件。使用 `nmcli`命令修改连接。示例如下：
```
[root@redhat8 ~]# nmcli con mod ens160 ipv4.dns 8.8.8.8
[root@redhat8 ~]# nmcli con down ens160
Connection 'ens160' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/14)
[root@redhat8 ~]# nmcli con up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/15)
[root@redhat8 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens160 |grep -i dns
DNS1=8.8.8.8
```
&#8195;&#8195;`nmcli con mod ID ipv4.dns IP`的默认行为是将任何旧的`DNS`设置替换为提供的新`IP`列表。`ipv4.dns`参数前面的`+`或`-`符号可添加或删除个别条目。示例如下：
```
[root@redhat8 ~]# nmcli con mod ens160 -ipv4.dns 8.8.8.8
[root@redhat8 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens160 |grep -i dns
[root@redhat8 ~]# 
```
&#8195;&#8195;要将`IPv6`地址为`2001:4860:4860::8888`的`DNS`服务器添加到要与 `static-ens3`连接一起使用的名称服务器的列表：
```
nmcli con mod static-ens3 +ipv6.dns 2001:4860:4860::8888
```
#### 测试DNS名称解析
&#8195;&#8195;通过`Ping`可以测试DNS服务器连接，也可以使用`host HOSTNAME`命令测试`DNS`服务器连接。示例如下：
```
[root@redhat8 ~]# host access.redhat.com
access.redhat.com is an alias for access.redhat.com2.edgekey.net.
access.redhat.com2.edgekey.net is an alias for access.redhat.com2.edgekey.net.global
redir.akadns.net.access.redhat.com2.edgekey.net.globalredir.akadns.net is an alias for e40408.ca2.s.t
l88.net.e40408.ca2.s.tl88.net has address 115.152.254.112
e40408.ca2.s.tl88.net has address 115.152.254.98
[root@redhat8 ~]# host 115.152.254.112
Host 112.254.152.115.in-addr.arpa. not found: 3(NXDOMAIN)
```
&#8195;&#8195;`DHCP`会在接口启动时自动重写`/etc/resolv.conf`文件，除非在相关的接口配置文件中指定了`PEERDNS=no`。使用`nmcli`命令设置此项，示例：
```
nmcli con mod "ens160" ipv4.ignore-auto-dns yes
```
## 练习