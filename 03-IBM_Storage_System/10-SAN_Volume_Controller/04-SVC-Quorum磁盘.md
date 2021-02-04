# SVC-Quorum磁盘
定额磁盘是MDisk或受管驱动器，包含专门用于系统管理的保留区域，系统自动分配候选定额磁盘。
## 定额磁盘介绍
### 概念
&#8195;&#8195;定额磁盘是MDisk或受管驱动器，包含专门用于系统管理的保留区域（如记录镜像卷的同步状态），系统自动分配候选定额磁盘。
### 用途
&#8195;&#8195;定额磁盘用于在一组表决节点不同意系统的当前状态时进行仲裁。系统使用定额磁盘来管理将系统均匀一分为二的 SAN 故障。 一半系统继续运行，另一半系统停止，直至SAN连接恢复。
### 仲裁机制
&#8195;&#8195;可将系统分为两组，每组均包含系统中原来一半数量的节点。定额设备确定哪组节点停止运行并停止处理I/O请求。在这种仲裁情况下，访问定额设备的第一组节点会标记为定额设备的所有者，因而可以继续作为系统运行，并处理所有I/O请求。如果另一组节点无法访问定额设备，或者发现定额设备由另一组节点所拥有，那么该组节点将停止作为系统运行，并且不会处理I/O请求。
### 选定条件
&#8195;&#8195;在定额磁盘发现期间，系统评估每个逻辑单元 (LU) 以确定其是否可能用作定额磁盘。 系统从一组合格的LU中指定三个候选定额磁盘。     
LU 必须满足以下条件，才能被视为候选定额磁盘：
- 必须处于受管方式
- 必须对系统中的所有节点都可视
- 必须由作为定额磁盘的认可主机的存储系统提供
- 在iSCSI连接的存储系统上无法找到定额磁盘
- 必须具有足够的空闲数据块来保存系统状态和配置元数据

&#8195;&#8195;每个系统只能拥有一个在仲裁情况下使用的活动定额设备。但是，系统最多使用三个定额设备来记录在发生灾难时要使用的系统配置数据的备份。
### 注意事项
&#8195;&#8195;如果没有可用的定额磁盘，那么镜像卷可能会变为脱机状态。 在定额磁盘上记录镜像卷的同步状态；     
&#8195;&#8195;为了避免在单次故障中失去所有定额设备的可能性，请在多个存储系统上分配候选定额磁盘或者在多个服务器上运行IP定额应用程序。
## 配置建议
### 单节点配置
&#8195;&#8195;在未将系统配置为延伸或HyperSwap系统时，正常配置使用受管驱动器或MDisk作为定额设备。系统自动分配候选定额磁盘。但是，向系统添加新存储器或除去现有存储器时，最好是查看定额磁盘分配情况。（可选）可将 IP 定额设备配置为使用定额磁盘的替代方法或者提供额外的冗余。
### 其他情况配置
参考后面的官方链接。
## 查看及修改定额磁盘
### 查看方法
使用命令lsquorum可以查看当前系统定额磁盘配置，active状态为yes的是活动的定额磁盘。
### 修改条件
更改已分配为候选定额磁盘的受管磁盘时，请遵循以下一般准则：
- 如果可能，尽量使每个MDisk均由一个不同的存储系统来提供
- 在更改候选定额磁盘之前，请确保要分配为候选定额磁盘的受管磁盘的状态报告为online
- 请确保要分配为候选定额磁盘的受管磁盘具有512MB或更多容量
- 使用更小容量的MDisk，或将驱动器用作定额设备，以显著减少在必要时运行系统恢复过程（也称为第3层或T3恢复）需要的时间量

### 修改方法
使用命令chquorum修改当前系统定额磁盘配置，步骤如下：
- 首先使用lsquorum查看当前定额磁盘配置
- 系统默认三个，记录下需要修改旧定额磁盘的quorum_index列的编号，例如为1
- 查看确认新定额磁盘的id，在SVC上，在按池划分的mdisk中，找到需要作为新定额磁盘的mdisk，查看属性，记录（标识）id，例如为6
- 使用如下命令修改仲裁盘：`chquorum -mdisk 6 1`
- 其中“6”是新定额磁盘的id，“1”是旧定额磁盘的quorum_index
- 运行完成后无输出，lsquorum查看状态，可以看到已经修改成功了

## 参考链接
关于定额磁盘官方相关描述和命令及HyperSwap配置或基于IP的光纤通道用法参考链接：
- [配置定额](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_quorumoverview.html)
- [定额磁盘配置]https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_scsiquorumdiskovr_1bchni.html
- [定额磁盘创建和数据块分配](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_quorumdisksupportreq_293lei.html)
- [使用 CLI 设置定额磁盘](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_setquorumdisksmdisks_cli_05151600.html)
- [chquorum](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_chquorum.html)
- [lsquorum ](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_lsquorum_03301100.html)
- [镜像卷](https://www.ibm.com/support/knowledgecenter/zh/STPVGU_8.3.1/com.ibm.storage.svc.console.831.doc/svc_vdiskmirroring_3r7ceb.html)
