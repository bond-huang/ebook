# TSA-常见操作及变更
## 配置变更
### 更改集群ServerIP
&#8195;&#8195;例如将集群的服务IP更改为192.168.100.139，TSA版本为4.1，RHEL版本为7.2。首先更改环境变量，确保所有群集命令设置为应用于对等域中的所有节点：
```sh
export CT_MANAGEMENT_SCOPE=2    
```
停止集群资源组：
```sh
chrg -o offline rg_test
```
查看是否停止：
```sh
lssam
```
将`CritRsrcProtMethod`更改为`5`以阻止可能发生意外的重新启动：
```sh
chrsrc -c IBM.PeerNode CritRsrcProtMethod=5
```
修改`/etc/hosts`表，集群中两个节点都需要修改；    
修改TSA集群服务IP(定义名称test_serviceip)：
```sh
chrsrc -s ‘Name == “test_serviceip”’ IBM.ServiceIP  IPAddress=192.168.100.139
```
修改后检查确认整个`IBM.ServiceIP`配置：
```sh
lsrsrc -l IBM.ServiceIP
```
查看集群资源组详细信息：
```sh
lsrg -Ab -g rg_test -TV
```
查看集群服务：
```sh
lsrpdomain
```
更改CritRsrc保护方法回3：
```sh
chrsrc -c IBM.PeerNode CritRsrcProtMethod=3
```
启动资源组：
```sh
chrg -o online rg_test
```
查看服务状态：
```sh
lssam
```
参考链接：
- [chrsrc Command - IBM Documentation](https://www.ibm.com/docs/en/rsct/3.2?topic=chrsrc-command)
- [Attributes used by IBM.ServiceIP](https://www.ibm.com/docs/en/tsafm/4.1.1?topic=class-attributes-used-by-ibmserviceip)
- [Manually adding/removing interface alias/secondary addresses](https://www.ibm.com/support/pages/manually-addingremoving-interface-aliassecondary-addresses)
- [How to change the IP addresses for IBM Intelligent Operations Center 1.6 high availability topology](https://www.ibm.com/support/pages/how-change-ip-addresses-ibm-intelligent-operations-center-16-high-availability-topology)

### 更改node IP
&#8195;&#8195;例如将node1 eth0上的`192.168.100.131`改成`192.168.100.134`，TSA版本为4.1，RHEL版本为7.2。首先更改环境变量，确保所有群集命令设置为应用于对等域中的所有节点：
```sh
export CT_MANAGEMENT_SCOPE=2    
```
停止集群资源组：
```sh
chrg -o offline rg_test
```
查看是否停止：
```sh
lssam
```
将`CritRsrcProtMethod`更改为`5`以阻止可能发生意外的重新启动：
```sh
chrsrc -c IBM.PeerNode CritRsrcProtMethod=5
```
停止集群服务：
```sh
stoprpdomain test_Domain
```
关闭掉`192.168.100.131` IP所在的网卡：
```sh
ifconfig eth0 down
```
修改网卡配置文件：
```sh
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```
删除原有IP：
```sh
ip addr del 192.168.100.131/24 dev eth0
```
重启网络服务（高版本RHEL用systemctl）：
```sh
service network restart
```
确认IP正常后，更新RSCT的`ctrmc`配置，以允许使用新IP地址跨群集节点进行通信：
```sh
preprpnode node1 node2
```
启动集群服务：
```sh
startrpdomain test_Domain
```
确认已经是online状态：
```sh
lsrpdomain
```
检查确认是否更新（两个节点网络信息都显示）：
```sh
lsrsrc -Ab IBM.NetworkInterface
```
如果以上命令输出未显示新的IP 地址，则可能需要强制RSCT在所有节点之间同步配置。其中一个节点root用户执行：
```sh
runact -c IBM.PeerDomain SyncConfig TargetNodes='{“node1”，“node2”}'
```
再次检查确认是否更新（两个节点网络信息都显示）：
```sh
lsrsrc -Ab IBM.NetworkInterface
```
确认已经是online状态：
```sh
lsrpdomain
```
更改CritRsrc保护方法回3：
```sh
chrsrc -c IBM.PeerNode CritRsrcProtMethod=3
```
启动资源组：
```sh
chrg -o online rg_test
```
查看服务状态：
```sh
lssam
```
参考链接：[TSAMP: Changing a node IP address in a running cluster](https://www.ibm.com/support/pages/tsamp-changing-node-ip-address-running-cluster)

### 更改资源监控
&#8195;&#8195;有时候需要手动停掉某个应用，但是不想通过TSA命令，手动操作后但是马上自动启动了，需要修改TSA某个应用不监控，就不会自动启动了，命令如下：
```
chrsrc -s 'Name=="<appname>"' IBM.Application MonitorCommandTimeout=0
```
注意事项：
- 搞完后修改回来，要不然TSA高可用无意义了
- 直接在线改，应用状态会变成`unknown`
- 如果停掉资源组去修改，启动资源组时候可能无法启动

## 待补充