# AIX-磁盘多路径

### MPIO与SDDPCM

#### 简介
&#8195;&#8195;AIX系统默认安装了MPIO Disk Path Control Module,SDDPCM(Subsystem Device Driver Path Control Module)是一个可单独安装的软件包，通过AIX MPIO框架为磁盘提供AIX MPIO支持，当支持的设备配置为MPIO-capable设备时，SDDPCM将被加载并成为AIX MPIO FCP（光纤通道协议）/FCoE（以太网光纤通道）设备驱动程序或SAS设备驱动程序的一部分。带有SDDPCM模块的AIX MPIO设备驱动程序或SAS设备驱动程序可增强数据可用性和I/O负载平衡。

#### MPIO常用命令及示例
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
#### SDDPCM命令
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

#### 磁盘重要属性介绍
##### algorithm（只使用MPIO情况下）       
algorithm = fail_over：    
&#8195;&#8195;默认的，某些第三方ODM使用不同的默认值。使用此算法，一次只能将I/O沿一条路径路由；如果用于发送I/O的路径发生故障或被禁用，则会选择列表中的下一个启用的路径，并将I/O路由到该路径。可通过修改每个路径上的path_priority来自定义列表中路径选择的顺序。          
algorithm = round_robin：     
&#8195;&#8195;使用此算法，将在磁盘的所有启用路径上分配和激活I/O。可以通过修改path_priority属性来加权沿每个路径路由的I /O百分比；如果路径发生故障或被禁用，则该路径不再用于发送I/O。      
algorithm = shortest_queue：     
&#8195;&#8195;该算法的行为与round_robin轻负载时非常相似。当负载增加时，此算法将优先选择活动I/O操作最少的路径。因此，如果一个路径由于存储区域网络（SAN）的拥塞而变慢，则其他较少拥塞的路径将用于更多的I/O操作。该算法将忽略路径优先级值。    

##### algorithm（使用SDDPCM）  
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

##### hcheck_mode（路径运行状况检查模式）      
hcheck_mode = nonactive：   
&#8195;&#8195;在此模式下，PCM在没有活动I/O的路径上发送运行状况检查命令，其中包括状态为failed的路径。如果选择的算法是故障转移，那么还将在状态为启用但没有活动I/O的每个路径上发送运行状况检查命令；如果选择的算法为round_robin或shortest_queue，则仅在状态为failed的路径上发送运行状况检查命令；如果磁盘处于空闲状态，则在运行状况检查间隔到期时，会在没有挂起I/O的任何路径上发送运行状况检查命令。

hcheck_mode = enabled：     
&#8195;&#8195;在此模式下，PCM沿所有启用的路径发送健康检查命令，甚至在健康检查时具有其它活动I/O的路径也是如此。
hcheck_mode = failed：    
&#8195;&#8195;在此模式下，PCM仅向标记为failed的路径发送路径运行状况检查。

##### hcheck_interval（路径运行状况检查间隔）
&#8195;&#8195;路径运行状况检查间隔是指基于hcheck_mode设置的MPIO路径运行状况检查将探测并检查打开的磁盘的路径可用性的时间间隔（以秒为单位）。当设置`hcheck_interval = 0`表示禁用MPIO的路径健康检查机制，这意味着任何failed的路径需要人工干预，以恢复或重新启用。     
&#8195;&#8195;大多数情况下，hcheck_interval为默认最好，设置过小会快速检查路径，以便恢复及尽快启用，但是可能会占用SAN上的大量带宽。     
&#8195;&#8195;AIX实施紧急最后一次健康检查，以在需要时恢复路径。如果设备只有一条非failed路径，并且在最后一条路径上检测到错误，那么AIX会在重试I/O之前在所有其它failed路径上发送运行状况检查命令，而不管hcheck_interval设置如何。      

##### timeout_policy（超时策略）
&#8195;&#8195;此属性指示发生命令超时时PCM应该采取的措施。当I/O操作未能rw_timeout在磁盘上的值内完成时，将发生命令超时。timeout_policy共有三个值（如果在设备上可以设置fail_path，推荐设置为此值）：       
timeout_policy = retry_path：      
&#8195;&#8195;在刚刚经历命令超时的同一路径上重试，这可能会导致I/O恢复的延迟，因为该命令可能会在此路径上继续失败，在连续几次失败之后，AIX才会使路径failed并在备用路径上尝试I/O。         
timeout_policy = fail_path：              
&#8195;&#8195;假设设备还有至少一条其它路径未处于failed状态，那么单个命令在路径上超时后，将此路径标记为failed，然后强制在其它正常路径上重试I/O，这可以从命令超时中更快地恢复，并且还可以更快地检测到设备的所有路径都failed的情况。系统会通过AIX运行状况检查命令恢复由于超时策略而failed的路径，但是，AIX在恢复后的一段时间内避免将路径用于用户I/O，以确保路径不会出现重复的故障。           
timeout_policy = disable_path：         
&#8195;&#8195;此设置会导致路径被禁用。禁用的路径只能通过手动用户干预，使用chpath命令重新启用路径来恢复。

##### 参考链接
IBM官方相关参考链接：
- MPIO介绍：[IBM AIX MPIO](https://developer.ibm.com/articles/au-aix-mpio/)
- SDDPCM介绍与下载：[Subsystem Device Driver Path Control Module](https://www.ibm.com/support/pages/node/651285)
- SDDPCM介绍:[Subsystem Device Driver Path Control Module for IBM AIX](https://developer.ibm.com/technologies/systems/articles/au-aix-install-sddpcm/#)

### 其它多路径软件
近期有安装过AIX Host Utilities，但是由于兼容性最后没成功，后期用到了再记录。
