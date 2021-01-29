# HMC-RMC
## 简介
&#8195;&#8195;要执行动态分区操作，需要逻辑分区与硬件管理控制台(HMC)之间的资源监视和控制(RMC)连接。RMC用作AIX和Linux逻辑分区与HMC之间的主通信信道，使用RMC可以配置用于管理普通系统情况的响应操作或脚本；如果不能针对逻辑分区执行添加或除去处理器、内存或I/O设备的操作，需要检查RMC连接是否处于活动状态。

## 验证RMC连接
需要超级管理员权限。
### 查看状态
连接到HMC命令行，输入`lspartition -dlpar`命令即可查看状态：
```
hscroot@DGNSHHMC1:~> lspartition -dlpar
<#0> Partition:<2*8233-E8B*069991R, , 10.8.251.102>
       Active:<1>, OS:<AIX, 6.1, 6100-07-06-1241>, DCaps:<0x2c5f>, CmdCaps:<0x4000001b, 0x1b>, PinnedMem:<1981>
<#1> Partition:<7*8233-E8B*06B496R, , 10.8.251.37>
       Active:<1>, OS:<AIX, 6.1, 6100-04-06-1034>, DCaps:<0xc4f>, CmdCaps:<0x4000001b, 0x1b>, PinnedMem:<1921>
<#2> Partition:<4*8408-E8E*840FB7W, , 10.8.253.204>
       Active:<1>, OS:<AIX, 7.1, 7100-03-09-1717>, DCaps:<0x2c5f>, CmdCaps:<0x4000001b, 0x1b>, PinnedMem:<4516>
<#3> Partition:<8*8233-E8B*069991R, , 10.8.251.18>
       Active:<1>, OS:<AIX, 6.1, 6100-07-06-1241>, DCaps:<0x2c5f>, CmdCaps:<0x4000001b, 0x1b>, PinnedMem:<6067>
```
说明：
- 如果是`<Active 1>`，那么RMC连接已建立
- 如果是`<Active 0>`或命令结果中未显示某些逻辑分区，说明RMC连接异常

在HMC命令行运行以下命令，检查高速缓存在HMC的数据存储库中的RMC连接状态的值：
```
hscroot@TEST:~> lssyscfg -r lpar -m 9117-570*65B4D6E -F name,rmc_state,rmc_ipaddr,rmc_ossh
utdown_capable,dlpar_mem_capable,dlpar_proc_capable,dlpar_io_capable
dump_9.200.104.134_not_shutdown,active,9.200.104.134,1,1,1,1
teacher02 9.200.104.218,inactive,9.200.104.218,0,0,0,0
teacher01 9.200.104.217,inactive,9.200.104.217,0,0,0,0
.....
VIOserver2,active,9.200.104.133,1,1,1,1
VIOserver1,active,9.200.104.132,1,1,1,1
```
说明：
- `rmc_state`属性的值必须是`active`或`inactive`,并且必须启用所有功能
- 如果`rmc_state`属性的值不是`active`或未将所有功能设置为1，通过运行`chsysstate -m <system name> -o rebuild -r sys`命令执行系统重建以刷新数据

### 故障排查
故障排查步骤：
- 验证是否禁用了HMC上的RMC防火墙端口，如果已禁用，需开启：
    - 在导航窗格中，打开 HMC 管理，选择：更改网络设置，单击LAN适配器选项卡
    - 选择除HMC与服务处理器相连的网卡，即配置HMC与分区网络互通的网卡，然后单击详细信息
    - 在LAN适配器选项卡的局域网信息下，验证打开是否处于选中状态，以及分区通信状态是否显示为已启用
    - 单击防火墙设置选项卡，确保RMC是允许的主机中显示的其中一个选项
    - 如果RMC没有被允许，更改设置对RMC开放，最后单击确定修改配置
- 使用telnet来访问逻辑分区。如果无法使用telnet，那么在HMC上打开虚拟终端，以便在逻辑分区上设置网络
- 如果逻辑分区网络已正确设置，并且仍然没有RMC连接，那么验证是否安装了RSCT文件集
- 如果RSCT文件集已安装，那么从逻辑分区使用telnet访问 HMC，以验证该网络是否正常工作以及防火墙是否已被禁用
- 如果RSCT文件集尚未安装，那么使用AIX 安装盘来安装
- 通过运行df命令（需要超级用户权限）验证HMC中的/tmp文件系统是否已满，如果满了需要清理

## 实例说明
### 实例一
&#8195;&#8195;在近期遇到过一种情况，就是同一个HMC的A机的a系统迁移到B机的b分区上了，b系统上原有RMC连接的网络弃用了，采用a分区的网络配置，这样就导致b上的RMC连接出现问题，`lspartition -dlpar`,命令还是可以看到a分区系统的信息，b分区的RMC连接异常，尝试过下面几种方法（hscroot用户）：
- 直接运行`chsysstate -m <system name> -o rebuild -r sys`，不行
- 重新启动HMC，不行，`lspartition -dlpar`看到结果还是一样
- 在HMC上删掉a分区的配置信息（如有需要建议进行备份），然后`chsysstate`就可以了

注意：
- 如果在没有RMC连接情况下进行动态分区操作，会有提示需要在重启分区后生效，虽然看到HMC上的资源是修改后的资源数量，但是实际系统使用的还是原来的
- 官方文档中有提到：更改网络设置或激活逻辑分区后，RMC连接大约需要五分钟来建立连接。建议在修改配置后，尽量等几分钟再查看状态

### 实例二
&#8195;&#8195;A分区安装后，RMC使用的是IP1，后来改成了IP2，但是HMC上记录的还是IP1信息，RMC连接状态就异常，使用命令`chsysstate -m <system name> -o rebuild -r sys`依然是不行，在HMC上重置了此物理机器的FSP连接后，等几分钟就正常了。

## 官方文档
官方参考文档：[验证移动分区的RMC连接](https://www.ibm.com/support/knowledgecenter/zh/POWER7/p7hc3/iphc3hmcpreprmc.htm)

官方参考文档：[逻辑分区与HMC之间的RMC连接故障诊断](https://www.ibm.com/support/knowledgecenter/zh/POWER7/p7hat/iphatrmctroubleshoot.htm)

