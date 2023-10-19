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
## 防火墙
### 防火墙管理
列出所有链的所有规则：
```sh
iptables -L
```
## 网络抓包
### Wrieshark工具
收集eth0网口上到指定主机22端口上的数据包信息，并写入指定文件：
```sh
tcpdump -i eth0 host 192.168.100.2 and dst port 22 -w wrieshark.cap
```
&#8195;&#8195;然后从服务器传出文件，通过Wrieshark打开，进行分析。Windows和Linux系统可以直接安装对应版本进行使用，如果没安装传出到有Wrieshark的机器查看分析。官方网站：[https://www.wireshark.org/](https://www.wireshark.org/)。

## 待补充