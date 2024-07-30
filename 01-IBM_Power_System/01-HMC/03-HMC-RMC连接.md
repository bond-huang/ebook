# HMC-RMC
## 简介
&#8195;&#8195;要执行动态分区操作，需要逻辑分区与硬件管理控制台(HMC)之间的资源监视和控制(RMC)连接。RMC用作AIX和Linux逻辑分区与HMC之间的主通信信道，使用RMC可以配置用于管理普通系统情况的响应操作或脚本；如果不能针对逻辑分区执行添加或除去处理器、内存或I/O设备的操作，需要检查RMC连接是否处于活动状态。

### 参考文档
官方参考链接：
- [Which HMC manages this server? ](https://www.ibm.com/support/pages/which-hmc-manages-server-0)
- [lssvc 命令](https://www.ibm.com/docs/zh/power9?topic=commands-lssvc-command#p9hcg_lssvc)
- [stopsvc 命令](https://www.ibm.com/docs/zh/power9?topic=commands-stopsvc-command)
- [Management domain configuration](https://www.ibm.com/docs/en/rsct/3.2?topic=security-management-domain-configuration)
- [Creating an RSCT management domain](https://www.ibm.com/docs/en/rsct/3.2?topic=managers-creating-rsct-management-domain)

其他参考文档：
- [Hardware Management Console (HMC ) Explained - UnixMantra](https://www.unixmantra.com/2013/10/hardware-management-console-hmc-explained.html)

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

### 实例三
&#8195;&#8195;HMC上列出分区信息后面可以看到AIX操作系统版本，但是当某个系统升级后，上面的信息没有更新，使用命令`chsysstate -m <system name> -o rebuild -r sys`等几分钟后刷新即可。

### 实例四
&#8195;&#8195;最近遇到在一个vios系统上，发现RMC连接异常，vios到HMC的657、22和443等端口都通，在vios上查看RMC连接信息发现还是以前的HMC信息，示例：
```
# lsrsrc IBM.MCP
Resource Persistent Attributes for IBM.MCP
resource 1:
        MNName            = "100.126.217.218"
        NodeID            = 14480238873508198302
        KeyToken          = "HMC113"
        IPAddresses       = {"100.126.217.113"}
        ConnectivityNames = {"100.126.217.218"}
        HMCName           = "7042CR8*84270AD"
        HMCIPAddr         = "10.126.217.113"
        HMCAddIPs         = ""
        HMCAddIPv6s       = ""
        ActivePeerDomain  = ""
        NodeNameList      = {"986W_VIOS1"}
```
实例中`10.126.237.113`地址不是现有HMC的地址，依次执行下面命令刷新配置：
```sh
/usr/sbin/rsct/bin/rmcctrl -z
/usr/sbin/rsct/bin/rmcctrl -A
/usr/sbin/rsct/bin/rmcctrl -p
```
启动后自动添加了一个配置：
```
#  lsrsrc IBM.MCP
Resource Persistent Attributes for IBM.MCP
resource 1:
        MNName            = "100.126.217.218"
        NodeID            = 14480238873508198302
        KeyToken          = "HMC113"
        IPAddresses       = {"100.126.217.113"}
        ConnectivityNames = {"100.126.217.218"}
        HMCName           = "7042CR8*84270AD"
        HMCIPAddr         = "10.126.217.113"
        HMCAddIPs         = ""
        HMCAddIPv6s       = ""
        ActivePeerDomain  = ""
        NodeNameList      = {"986W_VIOS1"}
resource 2:
        MNName            = "100.126.217.218"
        NodeID            = 1342464117648976727
        KeyToken          = "HMC259"
        IPAddresses       = {"100.126.217.259"}
        ConnectivityNames = {"100.126.217.218"}
        HMCName           = "7042CR9*686891D"
        HMCIPAddr         = "100.126.217.249"
        HMCAddIPs         = ""
        HMCAddIPv6s       = ""
        ActivePeerDomain  = ""
        NodeNameList      = {"986W_VIOS1"}
```
HMC上查看RMC连接状态：
```
<#280> Partition:<1*8286-42A*84C877W, , 100.026.217.218>
       Active:<1>, OS:<AIX, 6.1, 6100-09-07-1614>, DCaps:<0x14f9f>, CmdCaps:<0x4000003a, 0x3a>, PinnedMem:<3070>
```
### 实例六
&#8195;&#8195;有一台PowerLinux服务器，也是PowerVM虚拟化，近期将其从物理HMC替换成了虚拟HMC，但是HMC的IP没有变，迁移过去后，PowerLinux系统上的RMC连接异常，检查RMC信息还是以前的HMC的型号序列号：
```
# lsrsrc IBM.MCP
Resource Persistent Attributes for IBM.MCP
resource 1:
        MNName            = "10.10.114.113"
        NodeID            = 9845953214215121694
        KeyToken          = "hmc186"
        IPAddresses       = {"100.124.28.186"}
        ConnectivityNames = {"10.10.114.113"}
        HMCName           = "7042CR8*06A315C"
        HMCIPAddr         = "100.124.28.186"
        HMCAddIPs         = ""
        HMCAddIPv6s       = ""
        ActivePeerDomain  = ""
        NodeNameList      = {"testnode"}
```
&#8195;&#8195;使用`chsysstate -m Server-9008-22L-SN7894B4A -o rebuild -r sys`不行，使用`refrsrc IBM.MCP`不行。首先创建一个配置文件infile.mcp，写入如下目标HMC信息内容：
```
PersistentResourceAttributes::
resource 1:
        MNName            = "10.10.114.113"
        KeyToken          = "vhmc186"
        IPAddresses       = {"100.124.28.186"}
        ConnectivityNames = {"10.10.114.113"}
        HMCName           = "V807f77*6d11757"
        HMCIPAddr         = "101.124.24.194"
        HMCAddIPs         = "100.124.28.186"
```
上面`101.124.24.194`和`100.124.28.186`都是新虚拟HMC的IP，主要是用原来的IP。输入如下命令更新配置：
```sh
chrsrc -f infile.mcp -s 'MNName == "10.10.114.113"' IBM.MCP
```
依次执行下面命令重启服务并刷新配置：
```sh
/usr/sbin/rsct/bin/rmcctrl -z
/usr/sbin/rsct/bin/rmcctrl -A
/usr/sbin/rsct/bin/rmcctrl -p
```
完成后RMC连接恢复正常。
## 官方文档
官方参考文档：[验证移动分区的RMC连接](https://www.ibm.com/support/knowledgecenter/zh/POWER7/p7hc3/iphc3hmcpreprmc.htm)

官方参考文档：[逻辑分区与HMC之间的RMC连接故障诊断](https://www.ibm.com/support/knowledgecenter/zh/POWER7/p7hat/iphatrmctroubleshoot.htm)

## 系统rsct相关
### rsct相关命令查看
操作系统上使用lsrsrc命令查看，正常示例：
```
# lsrsrc IBM.MCP
Resource Persistent Attributes for IBM.MCP
resource 1:
        MNName            = "10.10.114.202"
        NodeID            = 1348464137648977727
        KeyToken          = "HMC666"
        IPAddresses       = {"10.126.237.123"}
        ConnectivityNames = {"10.10.114.202"}
        HMCName           = "7042CR9*686666D"
        HMCIPAddr         = "10.126.237.123"
        HMCAddIPs         = "10.126.219.220"
        HMCAddIPv6s       = ""
        ActivePeerDomain  = "TESTDB_Clu"
        NodeNameList      = {"TESTDB2"}
```
相关进程查看示例：
```
qstcdb1:/#lssrc -a |grep rsct
 ctrmc            rsct             3801192      active
 IBM.DRM          rsct_rm          3604726      active
 IBM.ConfigRM     rsct_rm          3146338      active
 IBM.HostRM       rsct_rm          3604834      active
 IBM.MgmtDomainRM rsct_rm          3735902      active
 IBM.ServiceRM    rsct_rm          4391000      active
 IBM.StorageRM    rsct_rm          4260500      active
 IBM.GblResRM     rsct_rm          4850134      active
 IBM.RecoveryRM   rsct_rm          4784300      active
 IBM.TestRM       rsct_rm          4260282      active
 ctcas            rsct                          inoperative
 IBM.ERRM         rsct_rm                       inoperative
 IBM.AuditRM      rsct_rm                       inoperative
 IBM.LPRM         rsct_rm                       inoperative
```
rmc domain状态查看：
```
# /usr/sbin/rsct/bin/rmcdomainstatus -s ctrmc

Peer Domain Status
  I A  0xd34ca9a5ea1e37c6  0001  TESTDB1
  I A  0x849959f09d9ceec6  0003  TESTDB3
  S S  0xe8a76231700ca2eb  0002  TESTDB2

Management Domain Status: Management Control Points
  I A  0x12b6b4fd1cebb73f  0001  10.126.37.29
```
### 重启RMC服务
重启步骤如下：
- 停止RMC守护程序：`# /usr/sbin/rsct/bin/rmcctrl -z`
- 重新配置必要的信息并启动RMC守护程序：`/usr/sbin/rsct/bin/rmcctrl -A`
- 启用远程客户端连接：`# /usr/sbin/rsct/bin/rmcctrl -p`

### 刷新配置
&#8195;&#8195;命令`/usr/sbin/rsct/install/bin/recfgct`可以重新配置rsct，进而刷新rmc连接等，但是Domain在运行状态下无法执行。使用示例：
```
testdb1:/#/usr/sbin/rsct/install/bin/recfgct
Cannot run if an online Peer Domain exists (see output of lsrpdomain).
For assistance with removing domains, refer to the rmrpnode and rmrpdomain commands.
testdb1:/#lsrpdomain
Name       OpState RSCTActiveVersion MixedVersions TSPort GSPort 
testdb_clu Online  3.1.4.0           Yes           12347  12348  
```
重置配置成功示例：
```
# /usr/sbin/rsct/install/bin/recfgct
/usr/lib/dr/scripts/all/ctrmc_MDdr               DR script to refresh Management Domain configuration
0513-071 The ctcas Subsystem has been added.
0513-071 The ctrmc Subsystem has been added.
0513-059 The ctrmc Subsystem has been started. Subsystem PID is 11993300.
```
## 其他
同一个HMC,A机器的a分区rmc使用ip 1，B机器的b分区rmc使用ip 2，现在客户把a分区的业务迁移到b分区了，b分区rmc使用的ip 也改成ip 1，a分区关闭（但是不删除），这样在这台HMC上，用`lspartition -dlpar`查看a分区rmc ip还是ip 1，那么b分区的rmc连接是不正常的，使用`chsysstate -m <system name> -o rebuild -r sys`命令不行，除非删掉a分区然后chsysstate后，b分区的RMC才正常。请问下有没有办法不删除分区，在两个分区配置同样RMC ip时候（其中一个分区关闭了），把关闭分区的rmc 的ip信息删除

http://www-01.ibm.com/support/knowledgecenter/SGVKBA_3.1.5/com.ibm.rsct315.trouble/bl507_diagrmc.htm