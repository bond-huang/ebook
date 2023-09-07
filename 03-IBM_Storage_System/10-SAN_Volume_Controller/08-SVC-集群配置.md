# SVC-集群配置
## 集群初始化
### T口初始化系统
步骤如下：
- 个人电脑网口配置DHCP，或者配置静态地址，示例：
    - IP：192.168.0.100（不能配置192.168.0.1）
    - 掩码：255.255.255.0
    - 网关和dns：192.168.0.1
- 将个人电脑网口连接到SVC的`Technician port`，通常在服务器端口旁边有标记`T`
- 打开浏览器访问`https://service`，或者`https://192.168.0.1`
- 新系统的话，系统会自动跳转到初始化工具，根据提示一步步进行配置即可

官方参考链接：
- [Initializing the system by using the technician port (IBM® SAN Volume Controller 2145-SV3 and 2147-SV3)](https://www.ibm.com/docs/en/sanvolumecontroller/8.6.x?topic=is-initializing-system-by-using-technician-port-san-volume-controller-2145-sv3-2147-sv3)
- [Adding nodes to an existing system](https://www.ibm.com/docs/en/sanvolumecontroller/8.6.x?topic=system-adding-nodes-existing)

## SAN配置
### SAN配置和zone规划
官方参考链接：
- [SAN configuration and zoning rules summary](https://www.ibm.com/docs/en/sanvolumecontroller/8.6.x?topic=details-san-configuration-zoning-rules-summary)
- [Example SAN configurations](https://www.ibm.com/docs/en/sanvolumecontroller/8.6.x?topic=details-example-san-configurations)

## 外部存储器配置
### 外部Storwize系列存储
参考链接：[Direct Attachment of Storwize and SAN Volume Controller Systems](https://www.ibm.com/support/pages/direct-attachment-storwize-and-san-volume-controller-systems)