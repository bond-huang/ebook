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
官方参考链接：[IBM i 7.3 显示集群配置](https://www.ibm.com/docs/zh/i/7.3?topic=clusters-displaying-cluster-configuration)
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

### 待补充