# RedHat-网络管理
RHEL新版本（8.0或以上）网络管理工具参考后面的学习笔记：- [RHEL-系统网络管理](https://gitbook.big1000.com/05-IBM_Operating_System/06-RHEL%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/06-RHEL-%E7%B3%BB%E7%BB%9F%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86.html)。
## 网络端口管理
### netstat命令
查看端口是否可以访问：
```sh
telnet 192.168.100.138 9999
```
查看系统端口信息：
```sh
netstat -ntulp
```
查看多个端口状态：
```sh
netstat -anltp |grep -E "8888|9999|6666|7777"
```
注意：
- 较新版本RHEL中`ss`命令替换了`net-tools`软件包中所含的较旧工具 `netstat`。

### ncat命令
ncat是一个网络工具,主要用于端口侦听。使用示例：
```
[root@centos82 ~]# nc -vz 121.43.191.157 80
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to 121.43.191.157:80.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
```
示例说明：
- `-v`：显示命令执行过程
- `-z`：表示zero，只扫描侦听守护进程，而不向它们发送任何数据

参考链接：[Linux 命令（138）—— ncat 命令](https://cloud.tencent.com/developer/article/2179887)
### TCP端口状态
端口连接状态：
- LISTEN：表示等待来自任何远程TCP和端口的连接请求
- SYN-SENT：表示在发送了连接请求之后等待匹配的连接请求
- SYN-RECEIVED ： 表示在接收和发送了连接请求之后等待确认连接请求确认
- ESTABLISHED：表示打开的连接，接收到的数据可以传递给用户。连接的数据传输阶段的正常状态
- FIN-WAIT-1：表示等待来自远程TCP的连接终止请求或对先前发送的连接终止要求的确认
- FIN-WAIT-2：表示等待来自远程TCP的连接终止请求
- CLOSE-WAIT：表示等待来自本地用户的连接终止请求
- CLOSING：表示等待来自远程TCP的连接终止请求确认
- LAST-ACK ：表示等待先前发送到远程TCP的连接终止请求的确认（其包括对其连接结束请求的确认）
- TIME-WAIT：表示等待足够的时间以确保远程TCP收到其连接终止请求的确认
- CLOSED：CLOSED是虚构的，它代表了没有TCB的状态，因此没有连接。

参考文档或链接：
- [TCP/IP State Transition Diagram (RFC793)](file:///C:/Users/admin/Downloads/TCPIP_State_Transition_Diagram.pdf)
- [TCP端口状态说明ESTABLISHED、TIME_WAIT、 CLOSE_WAIT](https://developer.aliyun.com/article/572240)

## 防火墙
### 防火墙管理
列出所有链的所有规则：
```sh
iptables -L
```
## 网络抓包
### tcpdump命令
常用参数说明：
- `-c count`：收到计数数据包后退出
- `-C file_size`：指定存储文件大小，如果文件该文件大于file_size，关闭当前存储文件并打开新的存储文件。file_size的单位是1000000字节，而不是1048576字节
- `-i interface`或`--interface=interface`：监听的网络接口，例如`eth0`
- `-w file`：将原始数据包写入文件
- `-n`：不要将主机地址转换为名称。可避免DNS查找
- `-q`  ：快速输出，打印更少的协议信息，因此输出行更简短
- `-r file`：从文件中读取数据包（该文件是用-w选项创建的，或者由其他编写pcap或pcap ng文件的工具创建的）。如果`file`为`-`，则使用标准输入
- `-s snaplen`或`--snapshot-length=snaplen`：从每个数据包中抓取snaplen字节的数据，默认是262144字节
- `-t`：不要在每个dump行上打印时间戳
- `-tt`：在每个dump行上打印时间戳，以自1970年1月1日00:00:00 UTC以来的秒为单位
- `-v`：在解析和打印时，生产多一点的详细输出
- `-vv`：在解析和打印时，生产更多一点的详细输出
- `-vvv`：在解析和打印时，生产更详细的输出

#### tcpdump命令示例
打印到达或离开redhat8主机的所有数据包：
```sh
tcpdump host redhat8
```
打印主机redhat8和redhat8a或redhat8b之间的流量：
```sh
tcpdump host redhat8 and \( redhata or redhatb \)
```
打印redhat8和除redhat9外的任何主机之间的所有IP数据包：
```sh
tcpdump ip host redhat8 and not redhat9
```
要通过互联网网关snup打印所有ftp流量：
```sh
tcpdump 'gateway snup and (port ftp or ftp-data)'
```
打印既不来自本地主机也不指向本地主机的流量（如果通过网关连接到另一个网络，这些东西永远不应该进入到本地网络）：
```sh
tcpdump ip and not net localnet
```
打印涉及非本地主机的每个TCP会话的开始和结束数据包（SYN和FIN数据包）：
```sh
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
```
打印进出端口80的所有IPv4 HTTP数据包，即仅打印包含数据的数据包，而不是例如SYN和FIN数据包以及仅ACK数据包：
```sh
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
打印通过网关snup发送的长度超过576字节的IP数据包：
```sh
tcpdump 'gateway snup and ip[2:2] > 576'
```
打印未通过以太网广播或多播发送的IP广播或多广播数据包：
```sh
tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'
```
打印所有不是回显请求/回复的ICMP数据包（即不是ping数据包）：
```sh
tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'
```
### 数据包字段含义
数据包中字段含义：
- `SYN`：同步序列编号（Synchronize Sequence Numbers）。是TCP/IP建立连接时使用的握手信号。在客户机和服务器之间建立正常的TCP网络连接时，客户机首先发出一个SYN消息，服务器使用SYN+ACK应答表示接收到了这个消息，最后客户机再以ACK消息响应
- `ACK`：确认字符（Acknowledge character），在数据通信中，接收站发给发送站的一种传输类控制字符，表示发来的数据已确认接收无误。在TCP/IP协议中，如果接收方成功的接收到数据，那么会回复一个ACK数据
- `RST`：重置连接（Reset the connection），表示正在中止连接。对于活动连接，节点会发送一个带有RST标志的TCP段，以响应在连接上接收到的不正确TCP段，从而导致连接失败
- `FIN`：完成发送数据（Finish sending data），指示TCP段发送器已完成在连接上发送数据。当TCP连接正常终止时，每个TCP对等端都会发送一个设置了FIN标志的TCP段

参考链接：[Understanding TCP Flags SYN ACK RST FIN URG PSH](https://www.howtouselinux.com/post/tcp-flags)
### Wrieshark工具
收集eth0网口上到指定主机22端口上的数据包信息，并写入指定文件：
```sh
tcpdump -i eth0 host 192.168.100.2 and dst port 22 -w wrieshark.cap
```
&#8195;&#8195;然后从服务器传出文件，通过Wrieshark打开，进行分析。Windows和Linux系统可以直接安装对应版本进行使用，如果没安装传出到有Wrieshark的机器查看分析。     
官方网站：[https://www.wireshark.org/](https://www.wireshark.org/)。     
参考链接：[Wireshark下载安装和使用教程](https://c.biancheng.net/view/6379.html)
## 老版本网络管理
### REHL 6.6网络配置
查看系统版本：
```sh
[root@localhost ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 6.6 (Santiago)
```
关掉NetworkManager：
```sh
[root@localhost network-scripts]# service NetworkManager status
NetworkManager (pid  2084) is running...
[root@localhost network-scripts]# service NetworkManager stop
Stopping NetworkManager daemon: [  OK  ]
[root@localhost ~]# service NetworkManager status
NetworkManager is stopped
```
配置ifcfg-eth1：
```ini
# for the documentation of these parameters.
DEVICE=eth1
BOOTPROTO=none
NETMASK=255.255.255.0
TYPE=Ethernet
HWADDR=56:f4:f2:9e:7a:08
IPADDR=10.110.147.166
IPV6INIT=no
ONBOOT=yes
USERCTL=no 
~
```
重启服务：
```
[root@localhost network-scripts]# service network restart
```
## 待补充