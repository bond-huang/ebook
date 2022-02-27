# HP_iLO使用
&#8195;&#8195;iLO是Integrated Ligths-out的简称,是HP服务器上集成的远程管理端口,它是一组芯片内部集成vxworks嵌入式操作系统,通过一个标准RJ45接口连接到工作环境的交换机。
## iLO基础使用
### iLO IP配置
&#8195;&#8195;iLO管理口默认是开启了DHCP，需要路由器DHCP分发IP或者用Laptop通过软件(例如dhcpsrv)分发IP。iLO的IP配置或修改说明如下：
- 如果是新机器，默认开启DHCP，开机按指定的键进入iLO进行修改
- 如果已经开机（不能关）并且没有改过iLO的IP，使用DHCP服务分发IP进入iLO，可以修改iLO的IP
- 如果已经开机，并且改过iLO的IP忘记了，只能重启设备，在启动时会提示iLO的IP，或者进入iLO查看

相关参考链接：
- [HP iLO-Discover the IP address](https://techexpert.tips/hp-ilo/discover-ilo-ip-address/)
- [HP iLO-Password recovery](https://techexpert.tips/hp-ilo/hp-ilo-password-recovery/)

#### dhcpsrv使用
确认iLO是DHCP开启状态下：
- 用Laptop网线直连到机器后面iLO端口上
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
- `IPPOOL`配置成`192.168.120.3-4`，给服务器分配会是这两个IP，ping这两个ip即可

#### 配置固定IP
以DL580 G7为例，版本为iLO3：
- 开机根据提示按`F8`进入iLO
- 在选项`Network`中选中`DNS/DHCP`,回车确认，将DHCP选项设置成`OFF`，按F10保存
- 在选项`Network`中选中`NIC and TCP/IP`，在里面配置IP信息，按F10保存

官方参考链接：[HP ProLiant DL580 G7服务器-开启ilo功能的方法](https://support.hpe.com/hpesc/public/docDisplay?docId=kc0110695zh_cn&docLocale=zh_CN)

### iLO登录
&#8195;&#8195;默认用户Administrator，默认密码在机器前面的铭牌标签上有写。如果登录密码忘记，需要进入iLO进行重置（重置iLO）或者修改。
#### 修改Administrator密码
以DL580 G7为例，版本为iLO3：
- 开机时候根据提示`F8`进入iLO
- 在选项`User`中选中`Administrator`
- 选择`Edit User`对用户进行编辑
- 在密码处输入新的密码，然后验证再输一次，按F10保存

#### 提示使用过期或者不安全的TLS安全设置
&#8195;&#8195;使用网页登录管理界面时，打不开网页，提示无法安全地连接到此页面，可能是使用过期或者不安全的TLS安全设置，，需要进行设置：
- 打开Internet选项，选择高级选项
- 将TLS 1.3取消掉，保留1.0、1.1及1.2
- 再次尝试，如果不行再去掉几个

### iLO重置
以DL580 G7为例，版本为iLO3：
- 开机时候根据提示`F8`进入iLO
- 在选项`File`中选中`Set Defaults`
- 回车确认即reset了iLO

重置说明：
- 重置后DHCP开启，IP地址等信息为空
- 用户Administrator密码恢复为默认，其他之前建立的用户会删除
- iLO里面关于机器的日志等信息都会重置
- 其他用户自定义配置都会重置

## iLO设备管理
