# AS400-PowerHA SystemMirror
官方参考链接：
- [IBM PowerHA SystemMirror for i 概述](https://www.ibm.com/docs/zh/i/7.3?topic=overview-powerha-systemmirror-i)
- [IBM PowerHA SystemMirror for i interfaces](https://www.ibm.com/docs/zh/i/7.3?topic=ham-powerha-systemmirror-i-interfaces)
- [PowerHA data replication technologies](https://www.ibm.com/docs/zh/i/7.3?topic=technologies-powerha-data-replication)
- [IBM i 7.3 管理PowerHA](https://www.ibm.com/docs/zh/i/7.3?topic=availability-managing-powerha)
- [IBM i 7.3 实现高可用性](https://www.ibm.com/docs/zh/i/7.3?topic=availability-implementing-high)

## 常用命令
### 集群节点命令
命令|描述
:---|:---
ENDCLUNOD|End Cluster Node
RMVCLUNODE|Remove Cluster Node Entry
RMVDEVDMNE|Remove Device Domain Entry
WRKCLU|Work with Cluster

### 集群资源组CRG命令
命令|描述
:---|:---
ADDCRGNODE|Add CRG Node Entry
CHGCRG|Change Cluster Resource Group
DSPCRGINF|Display CRG Information
DLTCRG|Delete Cluster Resource Group
DLTCRGCLU|Delete CRG Cluster
ENDCRG|End Cluster Resource Group
RMVCRGNODE|Remove CRG Node Entry

命令`WRKCLU`示例：
```
                               Work with Cluster                              
                                                             System:   LPAR110
 Cluster  . . . . . . . . . . . . . . . . :   TESTCLU                       
 Select one of the following:                                                 
                                                                              
      1. Display cluster information                                          
      2. Display cluster configuration information                            
           
      6. Work with cluster nodes                                              
      7. Work with device domains                                             
      8. Work with administrative domains                                     
      9. Work with cluster resource groups                                    
     10. Work with ASP copy descriptions                                      
                                                                              
     20. Dump cluster trace     
```
## 配置PowerHA
### 配置节点
官方参考链接：[IBM i 7.3 配置节点](https://www.ibm.com/docs/zh/i/7.3?topic=infrastructure-configuring-nodes)
## 管理集群
官方参考链接：[IBM i 7.3 管理集群](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-clusters)
### 监视集群状态
&#8195;&#8195;可以使用IBM Navigator for i或CL命令，示例使用命令`DSPCLUINF`(Display Cluster Information)打印集群`MYCLUSTER`的详细信息：
```
DSPCLUINF CLUSTER(MYCLUSTER) OUTPUT(*PRINT)
```
打印集群`MYCLUSTER`中定义的所有集群资源组的基本配置信息： 
```
DSPCRGINF CLUSTER(MYCLUSTER) CRG(*LIST) OUTPUT(*PRINT)
```
或者使用`WRKCLU`命令。官方参考链接：[IBM i 7.3 监视集群状态](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-monitoring-cluster-status)
### 显示集群配置
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`DSPCLUINF`(Display Cluster Information)或命令`WRKCLU`(Work with Cluster)。官方参考链接：[IBM i 7.3 显示集群配置](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-displaying-cluster-configuration)
### 管理节点
官方参考链接：[IBM i 7.3 管理节点](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-managing-nodes)
#### 显示节点属性
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`WRKCLU`(Work with Cluster)，示例显示集群中所有节点的列表以及有关每个节点的详细信息：
```
WRKCLU OPTION(*NODE)
```
显示集群资源组的列表，包含用于获取集群资源组更多信息的选项：
```
WRKCLU OPTION(*CRG)
```
官方参考链接：
- [IBM i 7.3 显示节点属性](https://www.ibm.com/docs/zh/i/7.3?topic=nodes-displaying-node-properties)
- [Work with Cluster (WRKCLU)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/wrkclu.htm)

#### 停止节点
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`ENDCLUNOD`(End Cluster Node)，示例立即结束节点`NODE01`上集群`MYCLUSTER`的集群资源服务：
```
ENDCLUNOD CLUSTER(MYCLUSTER) NODE(NODE01) OPTION(*IMMED)
```
官方参考链接：
- [IBM i 7.3 停止节点](https://www.ibm.com/docs/zh/i/7.3?topic=nodes-stopping)
- [End Cluster Node(ENDCLUNOD)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/endclunod.htm)

#### 移除节点
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`RMVCLUNODE`(Remove Cluster Node Entry)，示例从集群`MYCLUSTER`集群中删除节点`NODE01`,群集资源服务在节点`NODE01`上结束：
```
RMVCLUNODE CLUSTER(MYCLUSTER) NODE(NODE01)
```
官方参考链接：
- [IBM i 7.3 移除节点](https://www.ibm.com/docs/zh/i/7.3?topic=nodes-removing)
- [Remove Cluster Node Entry(RMVCLUNODE)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/rmvclunode.htm)

#### 从设备域中除去节点
&#8195;&#8195;设备域是集群中的一小部分节点，它们共享设备资源。从设备域中除去节点时，务必十分谨慎。如果从设备域中除去节点，并且该节点是任何独立磁盘池的当前主访问点，该设备域中的其他节点不再能够访问那些独立磁盘池。    
&#8195;&#8195;从设备域中除去节点后，如果一个或多个现有集群节点仍属于该设备域，那么无法将该节点添加回到该设备域。如需添加回来必须执行下列操作：
- 删除正在添加至设备域的节点当前拥有的独立磁盘池
- 通过对该节点执行IPL，重新启动系统
- [将节点添加至设备域](https://www.ibm.com/docs/zh/i/7.3?topic=nodes-adding-node-device-domain)
- 重新创建已在步骤1中删除的独立磁盘池

&#8195;&#8195;可以使用IBM Navigator for i或CL命令`RMVDEVDMNE`(Remove Device Domain Entry)，示例从集群`MYCLUSTER`集群中的设备域`MYDOMAIN`中删除节点`NODE01`：
```
RMVDEVDMNE CLUSTER(MYCLUSTER) DEVDMN(MYDOMAIN) NODE(NODE01)
```
官方参考链接：
- [IBM i 7.3 从设备域中除去节点](https://www.ibm.com/docs/zh/i/7.3?topic=nodes-removing-node-from-device-domain)
- [Remove Device Domain Entry(RMVDEVDMNE)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/rmvdevdmne.htm)

### 管理集群资源组(CRG)
官方参考链接：[管理集群资源组（CRG）](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-managing-cluster-resource-groups-crgs)
#### 显示CRG状态
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`DSPCRGINF`(Display CRG Information)，示例打印集群`MYCLUSTER`中定义的所有CGR的基本配置信息：
```
DSPCRGINF CLUSTER(MYCLUSTER) CRG(*LIST) OUTPUT(*PRINT)
```
或者使用命令`WRKCLU`，选择选项`9`，或下面示例显示集群当前已知的CRG列表：
```
WRKCLU OPTION(*CRG)
```
官方参考链接：
- [IBM i 7.3 显示CRG状态](https://www.ibm.com/docs/zh/i/7.3?topic=crgs-displaying-crg-status)
- [Display CRG Information(DSPCRGINF)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/dspcrginf.htm)

#### 停止CRG
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`ENDCRG`(End Cluster Resource Group)。示例结束`MYCLUSTER`集群中`MYCRG`集群资源组(当集群资源组退出程序被调用时，会向其传递恢复域中所有活动节点的退出程序数据“important information”)：
```
ENDCRG CLUSTER(MYCLUSTER) CRG(MYCRG)
    EXITPGMDTA('important information')
```
官方参考链接：
- [IBM i 7.3 停止CRG](https://www.ibm.com/docs/zh/i/7.3?topic=crgs-stopping-crg)
- [End Cluster Resource Group(ENDCRG)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/endcrg.htm)

#### 删除CRG
&#8195;&#8195;可以使用IBM Navigator for i或CL命令`DLTCRGCLU`(Delete CRG Cluster)。示例删除`MYCLUSTER`集群中`MYCRG`集群资源组：
```
DLTCRGCLU CLUSTER(MYCLUSTER) CRG(MYCRG)
```
&#8195;&#8195;或使用命令`DLTCRG`(Delete Cluster Resource Group)进行删除，示例从本地系统中删除`CRGTEST`集群资源组：
```
DLTCRG CRG(CRGTEST)
```
官方参考链接：
- [IBM i 7.3 删除CRG](https://www.ibm.com/docs/zh/i/7.3?topic=crgs-deleting-crg)
- [Delete Cluster Resource Group(DLTCRG)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/dltcrg.htm)
- [Delete CRG Cluster(DLTCRGCLU)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/dltcrgclu.htm)

#### 更改CRG的恢复域
&#8195;&#8195;在导航器的`Recovery Domain`页面上可添加和除去节点、更改节点角色、更改站点名称以及更改数据端口IP地址。或者使用CL命令操作。示例使用命令`ADDCRGNODE`(Add CRG Node Entry)将备份节点添加到CRG恢复域：
```
ADDCRGNODE CLUSTER(MYCLUSTER) CRG(MYCRG) RCYDMN(NODE1 *BACKUP 3)
```
示例说明：
- 示例将节点`NODE1`添加到`MYCLUSTER`集群中的集群资源组`MYCRG`的恢复域
- 该节点被添加为第三个备份节点，任何现有的备份节点将按顺序重新编号

&#8195;&#8195;示例使用命令`CHGCRG`(Change Cluster Resource Group)对CRT的恢复域进行更改，更改了文本描述和数据端口IP：
```
CHGCRG CLUSTER(MYCLUSTER) CRG(MYCRG) CRGTYPE(*DEV)
    EXITPGMFMT(*SAME) TEXT('CRG FOR CROSS SITE MIRRORING')
    RCYDMNACN(*CHGCUR)
    RCYDMN((NODE1 *SAME *SAME *SAME *ADD ('1.1.1.1')))
```
示例说明：
- 示例更改`MYCLUSTER`集群中`MYCRG`集群资源组，集群资源组对象的文本描述更改为指定的值
- 为节点`NODE1`添加数据端口IP地址

&#8195;&#8195;示例使用命令`RMVCRGNODE`(Remove CRG Node Entry)从集群`MYCLUSTER`中的集群资源组`MYCRG`的恢复域中删除节点`NODE03`：
```
RMVCRGNODE CLUSTER(MYCLUSTER) CRG(MYCRG) NODE(NODE03)
```
官方参考链接：
- [IBM i 7.3 更改CRG的恢复域](https://www.ibm.com/docs/zh/i/7.3?topic=crgs-changing-recovery-domain-crg)
- [Add CRG Node Entry(ADDCRGNODE)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/addcrgnode.htm)
- [Change Cluster Resource Group(CHGCRG)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/chgcrg.htm)
- [Remove CRG Node Entry(RMVCRGNODE)](https://www.ibm.com/docs/en/ssw_ibm_i_73/cl/rmvcrgnode.htm)

## iASP数据复制
### 从集群系统到独立分区
&#8195;&#8195;Attach IASP操作(通常称为ASP assigner process)是从连接到独立分区的IASP中定位磁盘单元，并更新分区以将IASP接受到配置中. 然后可以改变IASP，允许程序访问IASP中的数据。附加IASP操作强制执行以下限制：
- 分区必须在单一系统环境中。这意味着分区不能是设备域中的节点
- 该分区不能有任何IASP，这些独立的辅助存储池是在执行附加IASP操作时配置的
- 除非是磁盘池组，否则不能将多个IASP附加到分区。磁盘池组是单个主磁盘池以及与主磁盘池关联的所有辅助磁盘池

将独立辅助存储池(IASP)从集群节点复制到非集群节点的步骤：
- 接收IASP副本的独立分区正常运行，且当前没有将IASP副本的磁盘映射到此独立分区
- 通过外部存储操作，对IASP中的所有磁盘单元执行复制操作。例如FlashCopy、全局复制、克隆、快照等
- 在接收IASP副本的独立分区上，如果存在IASP，使用命令`CFGDEVASP`将其删除，示例：`CFGDEVASP ASPDEV(IASPNAME) ACTION(*DELETE)`
- 然后运行`CFGDEVASP ASPDEV(*ALL) ACTION(*PREPARE)`命令为连接包​​含IASP副本的磁盘单元准备系统配置。说明及注意事项：
    - 此命令删除并清理当前配置的所有IASP
    - 此时不应将存储LUN映射到接收IASP的目标独立分区
- `IPL`独立复制目标系统完成准备工作
- 将复制的磁盘单元分配给独立复制目标系统（将卷从存储映射到主机）
- 从专用服务工具(DST) 或系统服务工具 (SST) 运行Attach IASP操作：
    - 运行命令`STRSST`
    - 选择选项`3. Work with disk units`
    - 选择选项`3. Work with disk unit recovery`
    - 选择选项`8. Detect attached IASP disk units`
    - 进入`Confirm attach IASP disk units`屏幕，显示独立磁盘池和磁盘池中的单元，如果显示的磁盘池和单元正确，回车确认运行配置
- 运行高级分析命令`MULTIPATHRESETTER`(尝试过没成功，官网提到需要做)：
    - `STRSST`进入SST
    - 选择选项`1. Start a service tool`
    - 选择选项`4. Display/Alter/Dump`
    - 选择选项`·. Display/Alter storage`
    - 选择选项`2. Licensed Internal Code (LIC) data`
    - 选择选项`14. Advanced analysis`
- 进入`Select Advanced Analysis Command`屏幕，在`Option`第一行中输入`1`(Select)，然后在后面输入命令`MULTIPATHRESETTER`
- 回车确认，进入`Specify Advanced Analysis Options`屏幕，`Option`中`-RESETMP -ALL`，回车确认
- 阅读并按照屏幕上显示的说明确认应重置路径（这一步和官方描述有点不一样）
- Vary-on IASP。如果IASP设备描述不存在，先使用`CRTDEVASP`命令创建它
- 验证IASP中的数据，是否能够运行使用独立磁盘池中的数据的作业

官方参考链接：[PowerHA: How to Copy an Independent Auxiliary Storage Pool From a Cluster Node to a Non-Cluster Node](https://www.ibm.com/support/pages/powerha-how-copy-independent-auxiliary-storage-pool-cluster-node-non-cluster-node)
## PowerHA数据复制技术
&#8195;&#8195;PowerHA提供了几种不同的数据复制技术。这些技术可以单独使用，有时也可以组合使用，以提供更高级别的中断保护：
- `Geographic mirroring`是一种 IBM复制技术，可用于任何存储。数据在独立 ASP 的两个副本之间复制，同时支持同步和异步复制。`Geographic mirroring`可以防止服务器和存储中断
- 对于具有外部存储的客户，有多种技术可用：
  - `Switched logical units`允许将数据的一个副本从一个系统切换到另一个系统，并防止服务器中断
  - `Metro Mirror`和 `Global Mirror`是用于外部存储的同步和异步复制技术，通过将数据从 IASP 的主副本复制到备份副本来防止服务器和存储中断。
  - `HyperSwap`是一种`DS8000`技术，可在存储中断的情况下提供近乎零的中断
  - `FlashCopy`是一种时间点复制机制，它可以与任何技术结合使用，用于备份和其他用途

官方参考链接：[PowerHA data replication technologies](https://www.ibm.com/docs/zh/i/7.3?topic=technologies-powerha-data-replication)
### Geographic mirroring
&#8195;&#8195;`Geographic mirroring`使用IBM i 集群技术提供高可用性解决方案，其中存储在生产系统独立磁盘池中的一致数据副本在镜像副本上维护。`Geographic mirroring`通过使用内部或外部存储来维护独立磁盘池的一致备份副本：
- 如果生产站点发生中断，生产将切换到备份站点，其中包含数据的镜像副本，通常位于另一个位置：
  - 在同步交付模式下，数据在生产系统上完成写入操作之前被镜像，通常用于在发生故障时不会遭受任何数据丢失的应用程序
  - 在异步交付模式下，数据仍会在写入操作完成之前发送到镜像副本，但是，在镜像写入实际到达镜像副本之前，控制权会返回给应用程序
  - 使用现有同步交付模式的一个很好的原因是，如果应用程序想要确保生产端所有已完成的写入都已到达镜像副本端
  - 使用异步交付模式，应用程序响应时间不会像同步交付模式那样受到影响。大量延迟可能会导致异步交付模式所需的额外主存储和 CPU 资源
- `Geographic mirroring`通过使用数据端口服务在独立磁盘池之间提供逻辑页面级镜像。数据端口服务管理多个 IP 地址的连接，从而在`Geographic mirroring`环境中提供冗余和更大的带宽
- `Geographic mirroring`允许生产副本和镜像副本在Geographic上分开，这样可以在发生站点范围的中断时提供保护：
  - 在规划`Geographic mirroring`解决方案时，生产和镜像独立磁盘池之间的距离可能会影响应用程序响应时间
  - 生产副本和镜像副本之间的距离越远，可能会导致响应时间越长
  - 在实施使用`Geographic mirroring`的高可用性解决方案之前，必须了解用户的距离要求以及对应用程序的相关性能影响
- 具有异步交付模式的`Geographic mirroring`仅适用于 PowerHA 2.0 及更高版本

官方参考链接：
- [IBM i 7.3 Geographic mirroring](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-geographic-mirroring)
- [Planning geographic mirroring](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-geographic-mirroring)
- [Configuring geographic mirroring](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-geographic-mirroring)
- [Managing geographic mirroring](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-geographic-mirroring)
- [Scenario: Geographic mirroring](https://www.ibm.com/docs/zh/i/7.3?topic=availability-scenario-geographic-mirroring)

### Metro Mirror
&#8195;&#8195;`Metro Mirror`在两个IBM System Storage外部存储单元之间维护一致的数据副本，卷的目标副本不断更新以匹配对源卷所做的更改；`Metro Mirror`与集群技术一起使用时，可提供高可用性和灾难恢复解决方案。
- 与`Geographic mirroring`一样，也可以镜像存储在独立磁盘池中的数据，但是对于`Metro Mirror`，磁盘位于IBM System Storage的外部存储单元上
- 镜像从通常位于生产站点的源外部存储单元到通常位于备份站点的一组目标存储单元发生
- 数据在外部存储单元之间复制，为计划内和计划外中断提供可用性
- 它通常用于在发生故障时不会遭受任何数据丢失的应用程序
- 源卷和目标卷可以位于同一外部存储单元上，也可以位于单独的外部存储单元上
- 在独立单元的情况下，目标存储单元可以位于最远300公里（186 英里）以外的另一个站点但是，在此距离上使用同步通信时可能会对性能产生影响，考虑使用更短的同步通信以最大限度地减少性能影响可能更实际

官方参考链接：
- [IBM i 7.3 Metro Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-metro-mirror)
- [Planning Metro Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-metro-mirror)
- [Configuring Metro Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-metro-mirror)
- [Managing Metro Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-metro-mirror)
- [Scenario: Metro Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=availability-scenario-metro-mirror)
- [PowerHA supported storage servers](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-powerha-supported-storage-servers)

### Global Mirror

## 待补充