# AIX 网络与通信
## 网络管理
参考链接：
- [Network overview - Monitoring the hardware](https://developer.ibm.com/articles/au-aix7networkoptimize1/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)
- [NFS monitoring and tuning](https://developer.ibm.com/articles/au-aix7networkoptimize2/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)
- [Monitoring your network packets and tuning the network](https://developer.ibm.com/articles/au-aix7networkoptimize3/?mhsrc=ibmsearch_a&mhq=Ken%20Milberg)


nslookup

ssh -v <host>

cat /etc/ssh/sshd_config

/etc/netsvc.conf  hosts

mtrace  :打印从源到接收方的多点广播路径

traceroute 显示IP信息包至某个网络主机的路由



on remote system:
startsrc -s iptrace -a"-b -d <scpClientHostName> -l 300000000 -a /tmp/iptraceSCPserver.out"
<start scp command on client systema>
<then after scp command is terminated>
stopsrc -s iptrace

ON client system:
startsrc -s iptrace -a"-b -d <remoteHostName> -l 300000000 -a /tmp/iptraceSCPclient.out"
<issue scp command>
<wait 5 minutes then CTRL-C the command>
stopsrc -s iptrace

upload iptrace files:
FROM CLIENT:
/tmp/iptraceSCPclient.out
/tmp/iptraceSCPclient.out.old . ...... if exists
FROM SERVER:
/tmp/iptraceSCPserver.out


# no -po sb_max=2097152 
# chdev -l enX -a tcp_recvspace=1048576 -a tcp_sendspace=1048576

# no -po sb_max=8388608
# chdev -l enX -a tcp_recvspace=4193204 -a tcp_sendspace=4193204