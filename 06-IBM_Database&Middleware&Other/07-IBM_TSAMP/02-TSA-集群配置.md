# TSA-集群配置
## Service IP配置
官方参考文档：[IBM.ServiceIP resource class](https://www.ibm.com/docs/en/tsafm/4.1.1?topic=class-attributes-used-by-ibmserviceip)
### 定义Service IP
&#8195;&#8195;示例定义一个服务IP地址，IP地址为`192.168.100.138`，网络掩码为`255.255.255.0`，可在节点node1和node2上运行：
```sh
mkrsrc IBM.ServiceIP
  Name="myServerIP"
  NodeNameList="{'node1','node2'}"
  IPAddress=192.168.100.138
  NetMask=255.255.255.0  
```
&#8195;&#8195;如果节点node1和node2各自具有一个以上的网络接口。需要使用节点node1和节点node1的eth0设备，创建一个`Equivalency`：
```sh
mkequ MyInterfaces IBM.NetworkInterface:eth0:node1,eth0:node2
```
然后将服务IP与之关联：
```sh
mkrel -p dependson -S IBM.ServiceIP:myServerIP -G IBM.Equivalency:MyInterfaces 
         mySerIp_depon_MyInterfaces
```
查看关系：
```sh
lsrel -M mySerIp_depon_MyInterfaces
```
## 待补充