# AS400-PowerHA SystemMirror
官方参考链接：
- [IBM PowerHA SystemMirror for i 概述](https://www.ibm.com/docs/zh/i/7.3?topic=overview-powerha-systemmirror-i)
- [IBM PowerHA SystemMirror for i interfaces](https://www.ibm.com/docs/zh/i/7.3?topic=ham-powerha-systemmirror-i-interfaces)
- [PowerHA data replication technologies](https://www.ibm.com/docs/zh/i/7.3?topic=technologies-powerha-data-replication)
- [IBM i 7.3 管理PowerHA](https://www.ibm.com/docs/zh/i/7.3?topic=availability-managing-powerha)
- [IBM i 7.3 实现高可用性](https://www.ibm.com/docs/zh/i/7.3?topic=availability-implementing-high)
- [IBM i 7.3 Comparison of PowerHA data resiliency technologies](https://www.ibm.com/docs/en/i/7.3?topic=overview-comparison-powerha-data-resiliency-technologies)

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

#### CRG类型
CRG类型如下：
- `*APP`：应用程序集群资源组
- `*DATA`：数据集群资源组
- `*DEV`：设备集群资源组
- `*PEER`：对等集群资源组

#### CRG状态
CRG状态及描述如下：
- `Active`：集群资源组管理的资源当前是`resilient`状态
- `Inactive`：集群资源组管理的资源目前是非`resilient`状态
- `Indoubt`：集群资源组对象中包含的信息可能不准确：
  - 当使用`Undo`操作代码调用退出程序但未能成功完成时，会出现此状态
- `Restored`：集群资源组对象已在此节点上恢复，并且尚未复制到恢复域中的其他节点：
  - 在此节点上启动集群资源服务时，集群资源组将与恢复域中的其他节点同步，并将其状态设置为`Inactive` 
- `Add Pending`：Add Node Pending，新节点正在添加到集群资源组的恢复域中：
  - 如果退出程序成功，则状态将重置为其调用命令时的值
  - 如果退出程序失败且无法恢复原始状态，则状态设置为`Indoubt`
- `Delete Pending`：集群资源组对象正在被删除：
  - 当退出程序完成时，集群资源组将从恢复域中的所有节点中删除
- `Change Pending`：正在更改群集资源组：
  - 如果退出程序成功，则状态将重置为调用命令时的值
  - 如果退出程序失败并且无法恢复原始状态，则将状态设置为`Indoubt`
- `End Pending`：集群资源组的`Resilience`正在结束：
  - 如果退出程序成功，则状态设置为`Inactive`
  - 如果退出程序失败且无法恢复原始状态，则状态设置为`Indoubt`
- `Initialize Pending`：正在创建集群资源组，并且正在初始化：
  - 如果退出程序成功，则状态设置为`Inactive`
  - 如果退出程序失败，集群资源组将从所有节点中删除
- `Remove Pending`：Remove Node Pending。一个节点正在从集群资源组的恢复域中删除：
  - 如果退出程序成功，则状态将重置为调用命令时的值
  - 如果退出程序失败且无法恢复原始状态，则状态设置为`Indoubt`
- `Start Pending`：`Resilience`正在为集群资源组启动
  - 如果退出程序成功，则状态设置为`Active`
  - 如果退出程序失败且无法恢复原始状态，则状态设置为`Indoubt`
- `Switchover Pending`：`CHGCRGPRI`(Change Cluster Resource Group Primary)命令调用，出现集群资源组故障或节点故障，导致切换或故障开始，第一备用节点正在成为主节点：
  - 如果退出程序成功，则状态设置为`Active`
  - 如果退出程序失败且无法恢复原始状态，则状态设置为`Indoubt`
- `Delete Pending`：Delete Cmd Pending。`DLTCRG`(Delete Cluster Resource Group)命令正在删除集群资源组对象。集群资源组对象仅从运行该命令的节点中删除：
  - 这不是分布式请求。命令完成后，集群资源组将从节点中删除
- `Add Device Pending`：Add Device Entry Pending。正在将设备条目添加到群集资源组：
  - 如果退出程序成功，则状态将重置为其调用命令时的值
  - 如果退出程序失败并且无法恢复原始状态，则将状态设置为`Indoubt`
- `Remove Device Pending`：Remove Device Entry Pending。正在将设备条目从群集资源组中移除：
  - 如果退出程序成功，则状态将重置为其调用命令时的值
  - 如果退出程序失败并且无法恢复原始状态，则将状态设置为`Indoubt`
- `Change Device Pending`：Change Device Entry Pending。正在更改群集资源组中的设备条目：
  - 如果退出程序成功，则状态将重置为其调用命令时的值
  - 如果退出程序失败并且无法恢复原始状态，则将状态设置为`Indoubt`
- `Change Status Pending`：Change Node Status Pending。正在更改群集资源组的当前恢复域中节点的状态：
  - 如果更改成功，则状态将重置为其调用`CHGCLUNODE`(Change Cluster Node Entry)命令时的值
  - 退出程序失败会导致集群资源组的状态设置为`Indoubt`
  - 如果备份节点被重新分配为`Resilience`设备集群资源组的主节点，并且设备的所有权无法转移到新的主节点，则状态设置为`Indoubt`

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

### 管理集群管理域
参考链接：[IBM i 7.3 管理集群管理域](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-managing-cluster-administrative-domains)
#### 显示集群管理域
使用`WRKCADMRE`命令：[IBM i 7.3 Work with Monitored Resources](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/wrkcadmre.htm)
#### Monitored Resources Global Status
&#8195;&#8195;在`Work with Monitored Resources`显示受监视资源中的`Global Status`项显示的状态详细说明如下：
- `Added`：受监控资源条目及其属性已添加到集群管理域中的受监控资源目录中，但尚未同步，因为该域未处于活动状态
- `Consistent`：系统监视的所有资源属性的值在活动集群管理域中是相同的
- `Failed`：该资源不再受集群管理域的监视，应删除受监视的资源条目
- `Inconsistent`：受监视资源条目的一个或多个受监视属性未设置为域中一个或多个节点上的集群管理域已知的值
- `Pending`：受监视属性的值正在跨集群管理域同步
- `Unknown`：无法确定受监视资源条目的状态

#### Monitored Resources Local Status
&#8195;&#8195;在`Display Monitored Resource Details`显示Node中的`Local Status`项显示本地节点上资源的状态详细说明如下：
- `Current`：此节点上的受监视资源没有挂起的更新
- `Delete fail`：集群中的某个节点上的资源已被删除，管理员需要完成该过程
- `Delete pending`：已在集群中的某个节点上删除了受监视的资源，但该过程尚未在所有节点完全完成
- `Move fail`：资源已在集群中的某个节点上移动，管理员需要完成该过程
- `Move pending`：受监控的资源已在集群中的某个节点上移动，但所有节点的进程尚未完全完成
- `Rename fail`：资源已在集群中的某个节点上重命名，管理员需要完成该过程
- `Rename pending`：受监控的资源已在集群中的某个节点上重命名，但所有节点的进程尚未完全完成
- `Restore fail`：资源已在集群中的某个节点上恢复，管理员需要完成该过程
- `Restore pending`：受监控的资源已在集群中的某个节点上恢复，所有节点的进程尚未完全完成
- `Update fail`：此节点上的资源更新失败
- `Update pending`：此节点上的受监视资源有挂起的更新
- `Unknown`：无法确定受监视资源条目的状态

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
- `Geographic mirroring`是一种IBM复制技术，可用于任何存储。数据在独立ASP的两个副本之间复制，同时支持同步和异步复制。`Geographic mirroring`可以防止服务器和存储中断
- 对于具有外部存储的客户，有多种技术可用：
  - `Switched logical units`允许将数据的一个副本从一个系统切换到另一个系统，并防止服务器中断
  - `Metro Mirror`和 `Global Mirror`是用于外部存储的同步和异步复制技术，通过将数据从 IASP 的主副本复制到备份副本来防止服务器和存储中断。
  - `HyperSwap`是一种`DS8000`技术，可在存储中断的情况下提供近乎零的中断
  - `FlashCopy`是一种时间点复制机制，它可以与任何技术结合使用，用于备份和其他用途

官方参考链接：[PowerHA data replication technologies](https://www.ibm.com/docs/zh/i/7.3?topic=technologies-powerha-data-replication)
### Geographic mirroring
&#8195;&#8195;`Geographic mirroring`使用IBM i集群技术提供高可用性解决方案，其中存储在生产系统独立磁盘池中的一致数据副本在镜像副本上维护。`Geographic mirroring`通过使用内部或外部存储来维护独立磁盘池的一致备份副本：
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
&#8195;&#8195;`Global Mirror`在两个IBM System Storage外部存储单元之间维护一致的数据副本。`Global Mirror`在两个外部存储单元之间提供磁盘 I/O 子系统级别的镜像：
- 这种异步解决方案通过允许目标站点落后于源站点几秒钟，在无限距离上提供更好的性能，将数据中心分开更远的距离有助于防止区域中断
- `Global Mirror`使用异步技术提供跨两个站点的远程远程复制。通过高速光纤通道通信链路运行，旨在以几乎无限的距离异步维护完整且一致的远程数据镜像，对应用程序响应时间几乎没有影响
- 使用`Global Mirror`，复制到备份站点的数据在几秒钟内就可以与生产站点保持同步

官方参考链接：
- [IBM i 7.3 Global Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-global-mirror)
- [Planning Global Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-global-mirror)
- [Configuring Global Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-global-mirror)
- [Managing Global Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-global-mirror)
- [Scenario: Global Mirror](https://www.ibm.com/docs/zh/i/7.3?topic=availability-scenario-global-mirror)

### Switched logical units
&#8195;&#8195;`Switched logical units`是一个独立的磁盘池。当交换逻辑单元与IBM i集群技术相结合时，可以为计划内和一些计划外中断创建简单且经济高效的高可用性解决方案：
- 设备集群资源组(CRG)控制独立磁盘池，可以在计划外中断的情况下自动切换，也可以通过切换手动切换
- 集群中的一组系统可以利用切换功能将对切换逻辑单元池的访问从一个系统转移到另一个系统
- 可切换逻辑单元必须位于通过存储区域网络连接的IBM System Storage中
- 当切换独立磁盘池时，IBM System Storage单元内的逻辑单元会从一个系统重新分配到另一个系统

官方参考链接：
- [IBM i 7.3 Switched logical units](https://www.ibm.com/docs/zh/i/7.3?topic=technologies-switched-logical-units)
- [Planning switched logical units (LUNs)](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-switched-logical-units-luns)
- [Configuring switched logical units (LUNs)](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-switched-logical-units-luns)
- [Managing switched logical units (LUNs)](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-switched-logical-units-luns)
- [PowerHA supported storage servers](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-powerha-supported-storage-servers)

### FlashCopy
&#8195;&#8195;在使用IBM System Storage外部存储单元的IBM i 高可用性环境中，可以使用`FlashCopy`。`FlashCopy`为外部存储上的独立磁盘池提供几乎即时的时间点副本，这可以减少完成日常备份所需的时间：
- 时间点复制功能可让用户即时复制或查看原始数据在特定时间点的样子
- 目标副本完全独立于与源无关的磁盘池，并且在`FlashCopy`命令被处理后即可进行读写访问

#### 配置示例
系统中查看`FlashCopy`描述示例：
```
                          Display ASP Copy Description                 E980PRD 
                                                             05/21/21  14:56:21
 ASP copy description . . . . . . . . . :   E980PRD                            
 Device description . . . . . . . . . . :   CBSIASP                            
 Cluster resource group . . . . . . . . :   *NONE                              
 Cluster resource group site  . . . . . :   *NONE                              
 Location . . . . . . . . . . . . . . . :   E980PRD                            
 Sessions . . . . . . . . . . . . . . . :   IASPFC1                         
 IBM System Storage device  . . . . . . :   IBM.2107-75HBT61                   
   User . . . . . . . . . . . . . . . . :     qlpar                            
   Internet address . . . . . . . . . . :     10.22.168.66                      
                                                                               
   Alternate internet address . . . . . :     10.22.168.67 
```
对应的I/O资源查看示例：
```
                           Display ASP I/O Resources                   E980PRD 
                                                             05/21/21  14:56:32
                                   LUN ranges                                  
                                                                               
 Storage                            Consistency                                
 Identifier           Range         group range                                
 IBM.2107-75HAT61     1100-12A4      3100-32A4                                 
                      1100-13A4      3100-33A4  
```
官方参考链接：
- [IBM i 7.3 FlashCopy](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-flashcopy)
- [Planning FlashCopy](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-flashcopy-feature)
- [Configuring a FlashCopy session](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-flashcopy-session)
- [Managing the FlashCopy technology](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-flashcopy-technology)
- [Scenario: Performing a FlashCopy function](https://www.ibm.com/docs/zh/i/7.3?topic=environment-scenario-performing-flashcopy-function)

### DS8000 Full System HyperSwap
&#8195;&#8195;在 IBM i 高可用性环境中，使用HyperSwap作为一种方法来帮助减少或消除由于存储和SAN相关的中断而导致的中断：
- `Full System HyperSwap`是一个单系统IBM i 存储硬件高可用性解决方案，它使用IBM Systems Storage DS8000 设备上的`Metro Mirror`来维护两个IBM Systems Storage外部存储单元之间的一致数据副本
- `Full System HyperSwap`允许进行计划内或计划外的IBM Systems Storage外部存储设备切换，而无需使应用程序脱机以进行切换
- `Full System HyperSwap`仅支持全系统 (SYSBAS) 复制，不支持独立磁盘池复制
- `Full System HyperSwap` 具有与传统 Metro Mirror 跨站点镜像解决方案相同的距离限制
- 源卷和目标卷可以位于同一个外部存储单元上，也可以位于不同的外部存储单元上
- 在独立单元的情况下，目标存储单元可以位于最远300公里（186 英里）以外的另一个站点。但是，在此距离上使用同步通信时可能会对性能产生影响，考虑使用更短的同步通信以最大限度地减少性能影响可能更实际
- 使用`Full System HyperSwap`必须在系统上安装`IBM PowerHA for i Express Edition`
- 使用`Full System HyperSwap`不需要集群，也不使用集群技术

官方参考链接：
- [DS8000 Full System HyperSwap](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-ds8000-full-system-hyperswap)
- [Planning for DS8000 Full System HyperSwap](https://www.ibm.com/docs/zh/i/7.3?topic=resiliency-planning-ds8000-full-system-hyperswap)
- [Configuring DS8000 Full System HyperSwap](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-configuring-ds8000-full-system-hyperswap)
- [Managing DS8000 Full System HyperSwap](https://www.ibm.com/docs/zh/i/7.3?topic=powerha-managing-ds8000-full-system-hyperswap)

### DS8000 HyperSwap with IASPs
&#8195;&#8195;在 IBM i高可用性环境中，使用HyperSwap作为一种方法来帮助减少或消除由于存储和SAN相关的中断而导致的中断。`HyperSwap`是一种存储高可用性解决方案，允许在两个IBM System Storage DS8000单元之间镜像的逻辑单元以接近零的中断时间进行切换：
- 当`HyperSwap`在`IASP`级别实施时，可以与其他PowerHA技术相结合，为计划内和计划外存储中断提供最短停机时间解决方案，并为服务器计划内和计划外中断提供最短停机时间解决方案
- 要使用`HyperSwap`，必须在系统上安装`IBM PowerHA for i Enterprise Edition`
- 要将`DS8000 HyperSwap`与`IASP`一起使用，需要一个集群并且确实使用了PowerHA技术

官方参考链接：
- [DS8000 HyperSwap with independent auxiliary storage pools (IASPs)](https://www.ibm.com/docs/zh/i/7.3?topic=pdrt-ds8000-hyperswap-independent-auxiliary-storage-pools-iasps)
- [Planning DS8000 HyperSwap with independent auxiliary storage pools (IASPs)](https://www.ibm.com/docs/zh/i/7.3?topic=pdr-planning-ds8000-hyperswap-independent-auxiliary-storage-pools-iasps)
- [Configuring DS8000 HyperSwap with independent auxiliary storage pools (IASPs)](https://www.ibm.com/docs/zh/i/7.3?topic=ip-configuring-ds8000-hyperswap-independent-auxiliary-storage-pools-iasps)
- [Managing DS8000 HyperSwap with independent auxiliary storage pools (IASPs)](https://www.ibm.com/docs/zh/i/7.3?topic=mp-managing-ds8000-hyperswap-independent-auxiliary-storage-pools-iasps)

## 待补充