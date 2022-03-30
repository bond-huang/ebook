# SMC-基本操作
使用交换机型号： SMC8126L2 SMC TigerSwitch 26-Ports 10/100/1000
## 用户登录
### 管理网口登录
没有默认IP，默认是DHCP获取IP，示例使用dhcpsrv通过Laptop进行DHCP：
- 用Laptop网线直连到机器上管理网口(示例型号是25号口)
- 在Laptop上配置IP，例如：192.168.120.2
- 使用`dhcpwiz.exe`进行配置，或者直接编辑`dhcpsrc.ini`文件进行配置，示例见后续
- 使用管理员权限打开`dhcpsrv.exe`
    - 首先点击`Install`
    - 然后点击`Start`开启服务
- 长ping IP池里面的IP：
    - 能ping通然后通过网页打开管理终端
    - 如果不能ping通重新配置下Laptop上的IP

配置文件`dhcpsrc.ini`(后面DNS和HTTP服务不重要)示例：
```ini
[SETTINGS]
IPPOOL_1=192.168.120.3-4
IPBIND_1=192.168.120.2
AssociateBindsToPools=1
Trace=1
DeleteOnRelease=0
ExpiredLeaseTimeout=3600

[GENERAL]
LEASETIME=86400
NODETYPE=8
SUBNETMASK=255.255.255.0
NEXTSERVER=192.168.120.2
ROUTER_0=192.168.120.1
```
说明：
- Laptop配置成`192.168.120.2`，掩码`255.255.255.0`，网关和DNS不需要配置
- `IPPOOL`配置成`192.168.120.3-4`，给服务器分配会是这两个IP中之一
- Ping成功后，通过telnet或者WEB访问即可（直接输入IP，不需要端口号）
- 默认用户`admin`,默认密码`admin`

### 串口登录
RS-232串行口配置如下：
- 速率设置为9600
- 数据格式为8个数据位，1个停止位，无奇偶
- 流量控制设为无
- 仿真模式设为VT100
- 使用超级终端时，选择Terminal，不是Windowns键

## 待补充

