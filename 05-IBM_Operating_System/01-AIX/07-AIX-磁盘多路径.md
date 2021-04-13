# AIX-磁盘多路径
## AIXPCM
### 简介
&#8195;&#8195;IBM存储产品对SDDPCM的支持将于2020年6月30日正式终止（EOS）。 AIXPCM全称AIX path control module，是AIX操作系统中替代SDDPCM的默认多路径软件，目前据我了解不需要单独安装，在某个版本后默认在系统里面，如果没有升级到拥有的版本即可。
### 相关链接
官方相关链接如下：
- 各版本对应补丁下载：[Fix pack information for: AIX PCM RAS Enhancements](https://www.ibm.com/support/pages/node/5874489)
- SDDPCM迁移到AIXPCM说明：[How To Migrate SDDPCM to AIXPCM](https://www.ibm.com/support/pages/node/698075?mhsrc=ibmsearch_a&mhq=sddpcm)
- 迁移步骤说明：[Migrate to AIXPCM using "Manage_Disk_Drivers" Command](https://www.ibm.com/support/pages/migrate-aixpcm-using-managediskdrivers-command)
- Spectrum Virtualize存储推荐多路径说明：[Spectrum Virtualize Multipathing Support for AIX and Windows Hosts](https://www.ibm.com/support/pages/node/1106937?myns=s035&mync=E&cm_sp=s035-_-NULL-_-E)
- AIX和VIOS系统推荐多路径说明：[The Recommended Multi-path Driver to use on IBM AIX and VIOS When Attached to SVC and Storwize storage](https://www.ibm.com/support/pages/node/697363)
- IBM MPIO多路径说明：[IBM AIX multipath I/O (MPIO) resiliency and problem determination](https://developer.ibm.com/articles/au-aix-multipath-io-mpio/)
- lsmpio命令:[lsmpio命令](https://www.ibm.com/docs/en/aix/7.2?topic=l-lsmpio-command)

### 从SDDPCM迁移到AIXPCM
参考官方文档，使用PowerHA环境进行了测试：
- 操作系统版本：7100-04-05-1720
- PowerHA版本：7.2.1.4
- 外置存储类型：IBM SVC

#### 检查准备
检查AIX PCM 安装包（AIX 7.1）：
```
IV33411 Abstract: AIX PCM RAS Enhancements
    Fileset devices.common.IBM.mpio.rte:7.1.3.0 is applied on the system.
```
查看当前使用多路径：
```
# manage_disk_drivers -l |grep -i svc
IBMSVC              NO_OVERRIDE           NO_OVERRIDE,AIX_AAPCM,AIX_non_MPIO
# manage_disk_drivers -l |grep -i 2107ds8k
2107DS8K            NO_OVERRIDE           NO_OVERRIDE,AIX_AAPCM,AIX_non_MPIO
```
说明：
- 如果安装了SDDPCM，则值`NO_OVERRIDE`表示SDDPCM用于配置该系列的设备。如果未安装SDDPCM，则使用AIX默认PCM
- 对于`AIX_AAPCM`选项，即使安装了SDDPCM，管理员也可以指示AIX使用AIX默认PCM，修改后重新引导系统即可

修改默认属性：
```
# chdef -a queue_depth=32 -c disk -s fcp -t mpioosdisk
queue_depth changed
# chdef -a reserve_policy=no_reserve -c disk -s fcp -t mpioosdisk
reserve_policy changed
# chdef -a algorithm=shortest_queue -c PCM -s friend -t fcpother
algorithm changed
```
对于DS8K:
```
# chdef -a queue_depth=32 -c disk -s fcp -t aixmpiods8k
queue_depth changed
# chdef -a reserve_policy=no_reserve -c disk -s fcp -t aixmpiods8k
reserve_policy changed
```
说明：
- 当然也不一定要在修改路径模式前修改，如果本来都是使用的默认值的话
- 此修改根据需求来定，并不会修改当前磁盘的属性，修改后重新引导系统以修改后的值作为默认值
- AIXPCM的默认路径选择算法是`fail_over`
- AIXPCM支持`shortest_queue`算法，该算法类似于SDDPCM的`load_balance`算法

#### 确认是否使用SDDPCM
如果系统中没有SDDPCM设备，直接删除SDDPCM的安装包即可，查看目前系统有：
```
# pcmpath query device
Total Dual Active and Active/Asymmetric Devices : 4
DEV#:   4  DEVICE NAME: hdisk4  TYPE: 2145  ALGORITHM:  Load Balance
SERIAL: 600507180120262E3000000000000313
==========================================================================
Path#      Adapter/Path Name          State     Mode     Select     Errors
    0*          fscsi0/path0           OPEN   NORMAL         35          0
    1           fscsi0/path1           OPEN   NORMAL    2790295          0
    2*          fscsi1/path2           OPEN   NORMAL         35          0
    3           fscsi1/path3           OPEN   NORMAL    2765410          0
```
上面命令有输出代表使用了SDDPCM，继续查看：
```
# lsdev -Cc disk
hdisk0 Available          Virtual SCSI Disk Drive
hdisk4 Available 24-T1-01 MPIO FC 2145
```
#### 修改步骤
在PowerHA备机上进行，首先停掉PowerHA，执行下面命令修改：
```
# manage_disk_drivers -d IBMSVC -o AIX_AAPCM
 ********************** ATTENTION *************************
  For the change to take effect the system must be rebooted
```
提示需要重启，重新启动系统。
#### 检查确认
系统重新启动后，检查是否还使用SDDPCM：
```
# pcmpath query device
No device file found
```
然后使用`lsmpio`命令查看：
```
# lsmpio
name    path_id  status   path_status  parent  connection
hdisk0  0        Enabled  Sel          vscsi0  810000000000
hdisk1  0        Enabled  Sel          vscsi1  810000000000
hdisk4  0        Enabled  Clo          fscsi0  500507180120c1c4,2000000000000
hdisk4  1        Enabled  Clo          fscsi0  500507180120c5cb,2000000000000
```
可以看到NPIV方式过来的存储磁盘没有使用SDDPCM了，继续查看：
```
# manage_disk_drivers -l |grep -i svc
IBMSVC              AIX_AAPCM             NO_OVERRIDE,AIX_AAPCM,AIX_non_MPIO
# lsmpio -ar
Adapter Driver: fscsi0 -> AIX PCM
    Adapter WWPN:  c0507109681e0006
    Link State:    Up
                          Paths      Paths      Paths      Paths
    Remote Ports        Enabled   Disabled     Failed    Missing         ID
    500507180120c1c4          4          0          0          0   0xc95e00
    500507180120c5cb          4          0          0          0   0xc95600

Adapter Driver: fscsi1 -> AIX PCM
    Adapter WWPN:  c0507609688e0004
    Link State:    Up
                          Paths      Paths      Paths      Paths
    Remote Ports        Enabled   Disabled     Failed    Missing         ID
    500507180110c1c4          4          0          0          0   0xc95400
    500507180110c5cb          4          0          0          0   0xc95600
```
#### 信息对比
以hdisk4为例进行对比前后属性差别，SDDPCM时候VPD信息：
```
# lscfg -vpl hdisk4
  hdisk4           U8408.E8E.8490ECW-V2-C24-T1-W500507130110C1A4-L2000000000000  MPIO FC 2145

        Manufacturer................IBM
        Machine Type and Model......2145
        ROS Level and ID............0000
        Device Specific.(Z0)........0000063268181002
        Device Specific.(Z1)........0200602
        Serial Number...............600507180110821E3000000000000311
  PLATFORM SPECIFIC
  Name:  disk
    Node:  disk
    Device Type:  block
```
修改后VPD信息：
```
# lscfg -vpl hdisk4
  hdisk4           U8408.E8E.8490ECW-V2-C24-T1-W500507110110C1CA4-L2000000000000  MPIO IBM 2145 FC Disk
        Manufacturer................IBM
        Machine Type and Model......2145
        ROS Level and ID............30303030
        Serial Number...............2145
        Device Specific.(Z0)........0000063268181002
        Device Specific.(Z1)........
        Device Specific.(Z2)........
        Device Specific.(Z3)........
  PLATFORM SPECIFIC
  Name:  disk
    Node:  disk
    Device Type:  block
```
修改前属性：
```
# lsattr -El hdisk4
PCM             PCM/friend/sddpcm                                   PCM                                     True
PR_key_value    none                                                Reserve Key                             True
algorithm       load_balance                                        Algorithm                               True
clr_q           no                                                  Device CLEARS its Queue on error        True
dist_err_pcnt   0                                                   Distributed Error Percentage            True
dist_tw_width   50                                                  Distributed Error Sample Time           True
flashcpy_tgtvol no                                                  Flashcopy Target Lun                    False
hcheck_interval 60                                                  Health Check Interval                   True
hcheck_mode     nonactive                                           Health Check Mode                       True
location                                                            Location Label                          True
lun_id          0x2000000000000                                     Logical Unit Number ID                  False
lun_reset_spt   yes                                                 Support SCSI LUN reset                  True
max_coalesce    0x40000                                             Maximum COALESCE size                   True
max_transfer    0x40000                                             Maximum TRANSFER Size                   True
node_name       0x500507110110c1a4                                  FC Node Name                            False
pvid            00fa90eee3be28e90000000000000000                    Physical volume identifier              False
q_err           yes                                                 Use QERR bit                            True
q_type          simple                                              Queuing TYPE                            True
qfull_dly       2                                                   delay in seconds for SCSI TASK SET FULL True
queue_depth     20                                                  Queue DEPTH                             True
recoverDEDpath  no                                                  Recover DED Failed Path                 True
reserve_policy  no_reserve                                          Reserve Policy                          True
retry_timeout   120                                                 Retry Timeout                           True
rw_timeout      60                                                  READ/WRITE time out value               True
scbsy_dly       20                                                  delay in seconds for SCSI BUSY          True
scsi_id         0xc95400                                            SCSI ID                                 False
start_timeout   180                                                 START unit time out value               True
svc_sb_ttl      0                                                   IO Time to Live                         True
timeout_policy  fail_path                                           Timeout Policy                          True
unique_id       33213600507110180813E50000000000003F304214503IBMfcp Device Unique Identification            False
ww_name         0x500507110110c1a4                                  FC World Wide Name                      False
```
修改后属性：
```
# lsattr -El hdisk4
PCM             PCM/friend/fcpother                                 Path Control Module              False
PR_key_value    none                                                Persistant Reserve Key Value     True+
algorithm       shortest_queue                                      Algorithm                        True+
clr_q           no                                                  Device CLEARS its Queue on error True
dist_err_pcnt   0                                                   Distributed Error Percentage     True
dist_tw_width   50                                                  Distributed Error Sample Time    True
hcheck_cmd      test_unit_rdy                                       Health Check Command             True+
hcheck_interval 60                                                  Health Check Interval            True+
hcheck_mode     nonactive                                           Health Check Mode                True+
location                                                            Location Label                   True+
lun_id          0x2000000000000                                     Logical Unit Number ID           False
lun_reset_spt   yes                                                 LUN Reset Supported              True
max_coalesce    0x40000                                             Maximum Coalesce Size            True
max_retry_delay 60                                                  Maximum Quiesce Time             True
max_transfer    0x80000                                             Maximum TRANSFER Size            True
node_name       0x500507110100c1a4                                  FC Node Name                     False
pvid            00fa90eee3be28e90000000000000000                    Physical volume identifier       False
q_err           yes                                                 Use QERR bit                     True
q_type          simple                                              Queuing TYPE                     True
queue_depth     20                                                  Queue DEPTH                      True+
reassign_to     120                                                 REASSIGN time out value          True
reserve_policy  no_reserve                                          Reserve Policy                   True+
rw_timeout      30                                                  READ/WRITE time out value        True
scsi_id         0xc95400                                            SCSI ID                          False
start_timeout   60                                                  START unit time out value        True
timeout_policy  fail_path                                           Timeout Policy                   True+
unique_id       33213600507110180813E50000000000003F304214503IBMfcp Unique device identifier         False
ww_name         0x500507110110c1a4                                  FC World Wide Name               False
```
#### 卸载SDDPCM
使用`smit deinstall`命令即可：
```
                  Remove Installed Software
Type or select values in entry fields.
Press Enter AFTER making all desired changes.

                                                        [Entry Fields]
* SOFTWARE name			[devices.fcp.disk.ibm.mpio.rte]                     +
  PREVIEW only? (remove operation will NOT occur)     yes                   +
  REMOVE dependent software?                          yes                   +
  EXTEND file systems if space needed?                no                    +
  DETAILED output?                                    no                    +

  WPAR Management
      Perform Operation in Global Environment         yes                   +
      Perform Operation on Detached WPARs             no                    +
          Detached WPAR Names                        [_all_wpars]  
```
说明：
- 卸载`devices.fcp.disk.ibm.mpio.rte`包即可，`REMOVE dependent software?`选`yes`会卸载依赖包
- 先预览可以看到卸载的三个包，确认后`PREVIEW only?`选`no`开始卸载
- 卸载完成后建议重启下操作系统

卸载结果示例:
```
Installation Summary
--------------------
Name                        Level           Part        Event       Result
-------------------------------------------------------------------------------
devices.sddpcm.71.rte       2.6.6.0         ROOT        DEINSTALL   SUCCESS
devices.sddpcm.71.rte       2.6.6.0         USR         DEINSTALL   SUCCESS
devices.fcp.disk.ibm.mpio.r 1.0.0.24        USR         DEINSTALL   SUCCESS

File /etc/inittab has been modified.
```
验证是否卸载：
```
# lslpp -l |grep sddpcm
# pcmpath query device
ksh: pcmpath:  not found.
```
#### 后续操作
在PowerHA备节点修改成功及卸载SDDPCM后：
- 启动备节点PowerHA
- 从主节点切换资源组到备节点
- 按照之前步骤修改主节点的多路径方式

以上步骤在测试机进行验证，没有报错，没有发现异常，说明对PowerHA没有影响。

## MPIO与SDDPCM
SDDPCM IBM已经EOS。
### 简介
&#8195;&#8195;AIX系统默认安装了MPIO Disk Path Control Module,SDDPCM(Subsystem Device Driver Path Control Module)是一个可单独安装的软件包，通过AIX MPIO框架为磁盘提供AIX MPIO支持，当支持的设备配置为MPIO-capable设备时，SDDPCM将被加载并成为AIX MPIO FCP（光纤通道协议）/FCoE（以太网光纤通道）设备驱动程序或SAS设备驱动程序的一部分。带有SDDPCM模块的AIX MPIO设备驱动程序或SAS设备驱动程序可增强数据可用性和I/O负载平衡。

### MPIO常用命令及示例
查看所有磁盘路径类型（使用了SDDPCM会有所不同）：
```
# lsdev -Cc disk
hdisk0   Available 05-08-00-3,0 16 Bit LVD SCSI Disk Drive
hdisk1   Available 05-08-00-4,0 16 Bit LVD SCSI Disk Drive
hdisk2   Available 05-08-00-5,0 16 Bit LVD SCSI Disk Drive
hdisk3   Available 06-08-02     MPIO 2810 XIV Disk
hdisk4   Available 06-08-02     MPIO 2810 XIV Disk
```
查看所有磁盘路径及单个磁盘路径：
```
# lspath
# lspath -l hdisk3
Enabled hdisk3 fscsi0
Enabled hdisk3 fscsi1
```
查看所有磁盘路径（带路径id和及路径连接信息）及单个磁盘路径：
```
# lspath -F "status name path_id parent connection" 
# lspath -F "status name path_id parent connection" | grep -w hdisk3
Enabled hdisk3   0 fscsi0 500173800ccd0150,1000000000000
Enabled hdisk3   1 fscsi1 500173800ccd0140,1000000000000
```
列出所有存储系列及其支持的驱动程序（部分示例）：
```
# manage_disk_drivers -l
Device              Present Driver        Driver Options     
2810XIV             AIX_AAPCM             AIX_AAPCM,AIX_non_MPIO
DS4100              AIX_APPCM             AIX_APPCM,AIX_fcparray
```
修改驱动程序，使用AIX_non_MPIO管理2810XIV设备（会有提示需要重启）：
```
manage_disk_drivers -d 2810XIV -o AIX_non_MPIO
```
查看磁盘路径优先级：
```
# lspath -AHE -l hdisk2 -p vscsi1 
attribute value description user_settable
priority  1     Priority    True                        
```
修改磁盘路径优先级,步骤如下：
- smit mpio
- MPIO Path Management
- Change/Show Path Characteristice
- Change/Show Characteristics for a Device's Path
- Choose one Devide Name,and then choose one path
- Change the Priority 

命令示例：
```
chpath -l 'hdisk100' -p 'fscsi0' -w '500173800ccd0150,62000000000000' -a priority='1' 
```
禁用及启用vscsi下hdisk2的路径：
```
# chpath -l hdisk2 -p vscsi1 -s disable
paths Changed
# chpath -l hdisk2 -p vscsi1 -s enable 
paths Changed
```
强制删除hdisk2的vscsi0下830000000000路径，然后重新扫描（部分显示）：
```
# rmpath -l hdisk2 -p vscsi0 -w 830000000000 -d
path Deleted
# cfgmgr -vl vscsi0
----------------
attempting to configure device 'vscsi0'
Time: 0 LEDS: 0x25b3
invoking /usr/lib/methods/cfg_vclient -l vscsi0 
Number of running methods: 1
----------------
Completed method for: vscsi0, Elapsed time = 0
return code = 0
```
修改磁盘属性示例：
```
# chdev -l hdisk2 -a reserve_policy=no_reserve
# chdev -l hdisk2 -a algorithm=load_balance_port
# chdev -l hdisk2 -a algorithm=round_robin
```
### 路径状态
一共六种状态：
- Enabled：正常状态
- Disabled：手动disable
- Defined：已定义，ODM库中有信息
- Available：包含Enabled和Failed状态
- Missing：路径丢失
- Failed：路径失败

### SDDPCM命令
SDDPCM重要命令及其功能：
- pcmpath: 显示和管理 SDDPCM 设备。
- pcmpath query adapter: 显示适配器配置
- pcmpath query version: 显示 SDDPCM 的版本
- pcmpath query device: 显示 SDDPCM 设备（pcmpath query device 44 仅显示此设备）
- pcmpath query essmap: 显示完整概述
- pcmpath set device algorithm: 动态更改路径选择算法
- pcmpath set device hc_mode: 动态更改路径运行状况检查模式
- pcmpath set device hc_interval: 动态更改路径运行状况检查时间间隔
- pcmpath set device Mpath N online/offline: 动态启用（联机）或禁用（脱机）路径
- pcmpath set adapter N online/offline: 动态启用（联机）或禁用（脱机）适配器（SDDPCM 保留设备的最后一个路径，并且如果该设备正在使用最后一个路径，则会失败）
- pcmquerypr: 读取并清除暂存的保留和注册密钥
- pcmquerypr -vh /dev/hdisk30: 查询并显示暂存的保留（-V 详细模式以及详细信息）
- pcmquerypr -rh /dev/hdisk30: 释放暂存保留（如果设备被当前主机保留）
- pcmquerypr -ch /dev/hdisk30: 删除暂存保留并清除所有保留密钥注册
- pcmquerypr -ph /dev/hdisk30: 删除暂存保留（如果设备被其他主机保留）
- pcmgenprkey: 设置或清除所有 SDDPCM 多路径 I/O (MPIO) 设备的 PR_key_value Object Data Manager (ODM) 属性

### 磁盘重要属性介绍
#### algorithm（只使用MPIO情况下）       
algorithm = fail_over：    
&#8195;&#8195;默认的，某些第三方ODM使用不同的默认值。使用此算法，一次只能将I/O沿一条路径路由；如果用于发送I/O的路径发生故障或被禁用，则会选择列表中的下一个启用的路径，并将I/O路由到该路径。可通过修改每个路径上的path_priority来自定义列表中路径选择的顺序。          
algorithm = round_robin：     
&#8195;&#8195;使用此算法，将在磁盘的所有启用路径上分配和激活I/O。可以通过修改path_priority属性来加权沿每个路径路由的I /O百分比；如果路径发生故障或被禁用，则该路径不再用于发送I/O。      
algorithm = shortest_queue：     
&#8195;&#8195;该算法的行为与round_robin轻负载时非常相似。当负载增加时，此算法将优先选择活动I/O操作最少的路径。因此，如果一个路径由于存储区域网络（SAN）的拥塞而变慢，则其他较少拥塞的路径将用于更多的I/O操作。该算法将忽略路径优先级值。    

#### algorithm（使用SDDPCM）  
Failover only (fo 故障转移)：    
&#8195;&#8195;设备的所有I/O操作都将发送到同一（首选）路径，直到该路径由于I/O错误而failed。发生故障后选择一条备用路径用于后续的I/O操作。                
Load balancing (lb 负载均衡)：    
&#8195;&#8195;通过评估每个路径所连接的适配器上的负载来选择用于I/O操作的路径。如果多个路径具有相同的负载，则从这些路径中随机选择一个路径。负载均衡模式还包含故障转移保护。           
Load balancing sequential (lbs 负载均衡顺序)：          
&#8195;&#8195;此策略与Load balancing相同，针对I/O顺序进行了优化。负载均衡顺序策略也称为优化顺序策略，这是SDDPCM的默认设置。          
Round robin (rr 轮循)：          
&#8195;&#8195;从上一次I/O操作未使用的路径中随机选择用于每个I/O操作的路径。如果设备只有两个路径，那么 SDD 会交替使用这两个路径。             
Round robin sequential (rrs 轮循顺序)：       
&#8195;&#8195;该策略与round-robin策略相同，针对顺序 I/O 进行优化。

#### hcheck_mode（路径运行状况检查模式）      
hcheck_mode = nonactive：   
&#8195;&#8195;在此模式下，PCM在没有活动I/O的路径上发送运行状况检查命令，其中包括状态为failed的路径。如果选择的算法是故障转移，那么还将在状态为启用但没有活动I/O的每个路径上发送运行状况检查命令；如果选择的算法为round_robin或shortest_queue，则仅在状态为failed的路径上发送运行状况检查命令；如果磁盘处于空闲状态，则在运行状况检查间隔到期时，会在没有挂起I/O的任何路径上发送运行状况检查命令。

hcheck_mode = enabled：      
&#8195;&#8195;在此模式下，PCM沿所有启用的路径发送健康检查命令，甚至在健康检查时具有其它活动I/O的路径也是如此。
hcheck_mode = failed：         
&#8195;&#8195;在此模式下，PCM仅向标记为failed的路径发送路径运行状况检查。

#### hcheck_interval（路径运行状况检查间隔）
&#8195;&#8195;路径运行状况检查间隔是指基于hcheck_mode设置的MPIO路径运行状况检查将探测并检查打开的磁盘的路径可用性的时间间隔（以秒为单位）。当设置`hcheck_interval = 0`表示禁用MPIO的路径健康检查机制，这意味着任何failed的路径需要人工干预，以恢复或重新启用。     
&#8195;&#8195;大多数情况下，hcheck_interval为默认最好，设置过小会快速检查路径，以便恢复及尽快启用，但是可能会占用SAN上的大量带宽。     
&#8195;&#8195;AIX实施紧急最后一次健康检查，以在需要时恢复路径。如果设备只有一条非failed路径，并且在最后一条路径上检测到错误，那么AIX会在重试I/O之前在所有其它failed路径上发送运行状况检查命令，而不管hcheck_interval设置如何。      

#### timeout_policy（超时策略）
&#8195;&#8195;此属性指示发生命令超时时PCM应该采取的措施。当I/O操作未能rw_timeout在磁盘上的值内完成时，将发生命令超时。timeout_policy共有三个值（如果在设备上可以设置fail_path，推荐设置为此值）：       
timeout_policy = retry_path：      
&#8195;&#8195;在刚刚经历命令超时的同一路径上重试，这可能会导致I/O恢复的延迟，因为该命令可能会在此路径上继续失败，在连续几次失败之后，AIX才会使路径failed并在备用路径上尝试I/O。         
timeout_policy = fail_path：              
&#8195;&#8195;假设设备还有至少一条其它路径未处于failed状态，那么单个命令在路径上超时后，将此路径标记为failed，然后强制在其它正常路径上重试I/O，这可以从命令超时中更快地恢复，并且还可以更快地检测到设备的所有路径都failed的情况。系统会通过AIX运行状况检查命令恢复由于超时策略而failed的路径，但是，AIX在恢复后的一段时间内避免将路径用于用户I/O，以确保路径不会出现重复的故障。           
timeout_policy = disable_path：         
&#8195;&#8195;此设置会导致路径被禁用。禁用的路径只能通过手动用户干预，使用chpath命令重新启用路径来恢复。

#### 参考链接
IBM官方相关参考链接：
- MPIO介绍：[IBM AIX MPIO](https://developer.ibm.com/articles/au-aix-mpio/)
- SDDPCM介绍与下载：[Subsystem Device Driver Path Control Module](https://www.ibm.com/support/pages/node/651285)
- SDDPCM介绍:[Subsystem Device Driver Path Control Module for IBM AIX](https://developer.ibm.com/technologies/systems/articles/au-aix-install-sddpcm/#)

## 其它多路径软件
近期有安装过AIX Host Utilities，但是由于兼容性最后没成功，后期用到了再记录。
