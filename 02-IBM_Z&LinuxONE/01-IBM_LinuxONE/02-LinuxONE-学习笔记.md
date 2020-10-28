# LinuxONE-学习笔记
个人在听课学习中记录的要点，不是系统性的操作文档或者功能介绍文档，方便自己随手翻阅巩固知识点。
## LinuxONE-基础
机器和HMC基础知识：
- HMC是用户管理多台LinuxONE的控制台，和Power的HMC差不多
- SE 是一台LinuxONE的控制节点，主备模式，保存微码、license、VPD和profile
- 如果关闭两台SE，不影响LinuxONE运行，但是无法进行管理
- 一定要做好SE备份，两台SE硬盘全坏了就会丢失LinuxONE配置信息，没有备份无法恢复
- 系统操作员的操作都可以通过HMC完成，涉及到硬件维护的需要在SE本地完成（微码升级新版本可以通过HMC）
- Execption：红色"x"标记，表示资源"not accetptable",不代表有严重问题，和Power的HMC里面的严重程度不一样
- LinuxONE目前虚拟化主要使用DPM(Dynamic Partition Manager),支持用户动态修改逻辑分区
- 一个逻辑分区的Processor只能选择Shared或者Dedicated
- FCP占用一个设备号，网络设备占用三个设备号
- HMC中Tasks index菜单列出当前用户能够做的操作的清单
- 分区启动方式中“none”通常是用作调试，逻辑分区可以启动，资源会获取到，不会去启动操作系统
- 操作系统是以“device number”来驱动使用HBA卡的
- DPM Storage Group管理功能可以模拟逻辑分区操作去检查FCP的path和lun的可用性，点击“Connection Report”会触发，并且生成一个报告

## LinuxOne逻辑分区安装
### 安装分区注意
安装分区知识点：
- z15 T01最多可同时激活85个LPAR,z15 T02最多可同时激活40个LPAR
- Linux操作系统启动中需要手动安装就配置kickstart.cfg脚本,自动安装就不需要
- 在RHEL7版本中系统生成的网卡设备名称是什么在rd.znet中输入设备号0.0.0016,0.0.0017,0.0.0018,生成设备名enccw0.0.0016，在redhat8中是enc开头，enc13
- 逻辑分区的vNIC及SG attached操作完成后建议将Boot设置成NONE，然后进行一次分区的Start/Stop操作，以便将资源加载到配置中
- LinuxONE使用脚本安装时，kickstart可以通过以下两种方式指定并传给内核：
    - 在HMC Console中手工输入应答指定
    - 在RPM文件中通过inst.ks选项指定
- 如果采用手工安装并且选项GUI会TUI界面方式，操作工作站或笔记本需连接到逻辑分区OSA vNIC的连接网络
- 只使用HMC上的U盘进行逻辑分区的Linux操作系统安装时，必须满足的条件：
    - HMC的ETH1与SE的EM4连接在同一交换机
    - 用作系统盘的SG状态为Complete并已Attach到逻辑分区上
- 逻辑分区安装完成后，可以在线添加磁盘卷（LUN），不需要重启

开始逻辑分区操作系统安装前需要完成的工作：
- 完成逻辑分区网卡定义
- 获取逻辑分区volume信息
- 获取伙计分区ip及vlan信息

逻辑分区分配物理I/O端口时需要考虑的维度：
- 按I/O板卡打散分配
- 按I/O Drawer打散分配

### Partition Details-Network说明
在HMC中DMP进行虚拟网卡管理菜单说明：
- `name`是给HMC查看分区使用，不会传递到操作系统中，可以根据自己喜好自定义
- `Device Number`就是设备号，也是操作系统里面的设备号；创建时候可以自定义，不定义的话微码会自动分配，同一个逻辑分区不能重复，不同的逻辑分区可以重复
- `Adapter name`分四段，第一段网卡类型，例如：OSA或RoCE,第二段是AdapterID,第三段是卡所在I/O抽屉的位置编号，第四段是槽位号；在创建虚拟网卡选择对应物理设备时候，要注意看此网卡信息
- `Adapter port`，OSA一般为0或RoCE一般为1
- `card typy`是完整的网卡信息
- `Vlan ID`在HMC操作中很少用到
- `MAC address`在网络连通启动时候才显示

### FCP链路信息查看
安装准备时候，需要查看FCP链路信息：
- 进入需要安装系统对应的SG里面，检查下状态是否是complete
- 在VOLUMES选项下面会看到vol，根据`Type`类型选择对应卷点击`GET DETAILS`会出现盘卷对应的信息

如果做了一个逻辑分区定义，如果从来没有start，就会发现`GET DETAILS`是点不了的

### INS文件说明
文件示例:
```
* minimal lpar ins file
images/kernel.img 0x00000000
images/initrd.img 0x02000000
images/genericdvd.prm 0x00010480
images/initrd.addrsize 0x00010408
```
说明：
- 第一行是个注释行
- 第二行开始用来标识说明内核启用需要的文件路径

### PRM文件
rd.znet=qeth,0.0.0013,0.0.0014,0.0.0015,layer2=1:
- 用来定义网卡，qeth是OSA驱动程序名称
- 0.0.0013是三个device number，一个网卡会占用三个设备号，前面的0.0.最好建议带上
- layer2=1用来设定OSA虚拟网卡运行在哪一层模式，一般建议2层，可以打开监听模式，可以做网桥等等

ip=182.158.10.129:::255.255.255.0:bp01b1:enccw0.0.0013:off：
- 用来定义逻辑分区启动后网卡上使用的IP地址，用冒号分隔，如果中间参数不需要输入，也需要加入冒号进行隔离
- 格式说明：`ip:dns信息:网关信息:掩码:hostname:网卡的设备名:状态`

inst.ks=ftp://182.158.10.100/pub/ks/rhel77_bp01b1_ks.cfg：
- 这是用脚本安装方式配置，传给安装程序的参数，定义KICKSTART文件位置
- 如果手动安装不需要KICKSTART脚本，把inst.ks改成inst.repo，后面内容指向安装文件所在的目录即可

rd.multipath=1：
- 如果是FCP磁盘一般需要配置此行
- 启动介质默认没有带multipath，多路径磁盘会在操作系统上认到多个磁盘，路径不会整合，可能会导致系统装好了，但是启动不了

rd.zfcp=0.00003,500507680c31cd11,0000000000000000：
- 系统盘卷的FCP链路信息，一般有多个
- 第一段FCP是设备号，第二段是对端磁盘盘控HBA卡端口的wwpn，第三段是磁盘卷lun id

### kickstart脚本
可以实现自动化安装部署内容：
- 定义用户组和用户的创建
- 盘卷分区或逻辑卷定义
- 除一定义的vNIC之外的其它网卡网络设置定义
- 定义软件仓库定义repo
- 额外按的软件包，定义需要安装的安装包必须要在之前定义repo里面能够找到
- 系统安装完成后的配置，%post内容是在安装好的操作系统中以root身份对系统进行的操作，每一行就是一条命令
