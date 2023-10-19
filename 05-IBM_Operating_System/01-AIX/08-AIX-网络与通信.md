# AIX 网络与通信
## 网络管理
参考链接：
- [Network overview - Monitoring the hardware](https://developer.ibm.com/articles/au-aix7networkoptimize1/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)
- [NFS monitoring and tuning](https://developer.ibm.com/articles/au-aix7networkoptimize2/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)
- [Monitoring your network packets and tuning the network](https://developer.ibm.com/articles/au-aix7networkoptimize3/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)

## 网络配置
### 命令行配置
AIX使用smitty工具很方便，有时也会用到命令行。IP配置示例：
```sh
/usr/sbin/mktcpip -h'hostname' -a'192.168.100.160' -m'255.255.255.0' -i'en0'
```
使用smitty mktcpip配置IP和网关，命令同下：
```sh
/usr/sbin/mktcpip -h'hostname' -a'192.168.100.160' -m'255.255.255.0' -i'en0' \
-g'192.168.100.254' -A'no' -t'N/A' '-s'
```
静态路由配置：
```sh
chdev -l inet0 -a route=net,-hopcount,0,,,,,,-static,11.18.18.118,192.168.100.254
```
删除默认路由：
```sh
chdev -l inet0 -a delroute=net,-hopcount,0,,0,192.168.100.254
```
## 网络管理
### 网络trace
收集en0上端口22的trace，写入telnet.cap文件，命令示例：
```sh
tcpdump -i en0 -w telnet.cap -s 0 host 192.168.100.2 and port 22
```
## 待补充

nslookup

ssh -v <host>

cat /etc/ssh/sshd_config

/etc/netsvc.conf  hosts

mtrace  :打印从源到接收方的多点广播路径

traceroute 显示IP信息包至某个网络主机的路由



on remote system:
```
startsrc -s iptrace -a"-b -d <scpClientHostName> -l 300000000 -a /tmp/iptraceSCPserver.out"
<start scp command on client systema>
<then after scp command is terminated>
stopsrc -s iptrace
```

ON client system:
```
startsrc -s iptrace -a"-b -d <remoteHostName> -l 300000000 -a /tmp/iptraceSCPclient.out"
<issue scp command>
<wait 5 minutes then CTRL-C the command>
stopsrc -s iptrace
```

upload iptrace files:
FROM CLIENT:
/tmp/iptraceSCPclient.out
/tmp/iptraceSCPclient.out.old . ...... if exists
FROM SERVER:
/tmp/iptraceSCPserver.out

```
# no -po sb_max=2097152 
# chdev -l enX -a tcp_recvspace=1048576 -a tcp_sendspace=1048576

# no -po sb_max=8388608
# chdev -l enX -a tcp_recvspace=4193204 -a tcp_sendspace=4193204
```