# PowerHA Tools for IBM i - IASP Manager
&#8195;&#8195;PowerHA Tools IASP Manager是IBM Lab Services编写的高可用性产品。它专为使用IBM存储和/或将PowerHA与独立辅助存储池(IASP)解决方案结合使用的IBM i客户而设计。官方参考链接：[PowerHA Tools for IBM i - IASP Manager](https://www.ibm.com/support/pages/node/1126029)。
## 常用命令
常用命令如下：

命令|描述
:---|:---
CHKFLASH|Check FlashCopy
CHKPPRC|Check PPRC
CHGPPRC|Change PPRC
DSPCSEDTA|Display Copy Services Environment exit data
ENDFLASH|End a FlashCopy Backup
STRFLASH|Start a FlashCopy Backup
SWPPRC|Switch PPRC
WRKCSE|Work with Copy Services Environments

## Multi-Target Copy简介
&#8195;&#8195;`Multi-Target Copy`需要DS8000和IBM Copy Services Manager (CSM)。支持使用多达四个站点的多目标解决方案，包括：
- `Metro Mirror - Metro Mirror(MM/MM)`：来自同一生产（源）节点的两个`Metro Mirror`目标
- `Metro Mirror - Global Mirror(MM/GM)`：来自同一生产（源）节点的一个`Metro Mirror`目标和一个`Global Mirror`目标
- `Metro Mirror - Global Mirror + GCP(MM/GM)`：来自同一源的一个`Metro Mirror`目标，一个`Global Mirror`目标 ，加一个从`Global Mirror`节点级联的`Global Copy`目标

PowerHA Tools IASP Manager为`Multi-Target`环境提供完全自动化，包括：
- 检查环境是否准备好切换
- 在切换期间关闭生产中的`IASP`
- `Metro Mirror`或`Global Mirror`环境的计划内或计划外切换的自动化
- 切换到拥有生产副本的新节点后将`IASP`进行`Vary on`

## Flash Copy
### CHKFLASH命令
&#8195;&#8195;`CHKFLASH`(Check FlashCopy)命令检查`CSE CRG`、节点和硬件资源的状态，以确定可以启动 FlashCopy(STRFLASH)。在`CHKFLASH`期间发现的所有错误都记录在`qzrdhasm.log`文件中，该文件位于运行命令分区上的`/qibm/qzrdhasm`目录中。
### STRFLASH命令
&#8195;&#8195;执行必要的步骤，将当前生产分区`IASP`快速复制到FlashCopy(备份)分区`IASP`，并使FlashCopy(备份)分区 `IASP`可用：
- 此命令可以在`cluster/recovery`中的任何节点上运行，以执行使指定`IASP`的第二个副本可用所需的步骤
- 副本将从指定的源节点获取，该源节点也可能参与`Metro Mirror`或`Global Mirror`关系，无论复制方向如何
- 对于冷FlashCopy，生产分区`IASP`将自动关闭，FlashCopy数据将是生产数据的精确副本
- 对于热FlashCopy，生产分区`IASP`保持开启，因此FlashCopy可能会丢失一些尚未刷新到生产分区磁盘的数据

`STRFLASH`命令属性描述：
- `Environment name`：要使用的`FLASH CSE`环境的名称
- `Flash Target Node Name`：FlashCopy目标节点的名称，`*LOCAL`或`FC node`
- `Vary on after flash`：是否在`FlashCopy node`上`Vary on`，`*YES`或`*NO`
- `Quiesce Action`：`*ENV`或下面一种：
  - `*QUIESCE`：将内存刷新到磁盘并在闪存期间暂时挂起`PPRC`
  - `*FRCWRT`：将内存刷新到磁盘，但不在闪存期间暂停`PPRC`
  - `*NONE`：在启动闪存之前不要将内存刷新到磁盘
- `Cluster Resource Group`：
  - `*ENV`
  - `CRG name`：如果与环境名称不同
- `Preflashed`：
  - `*NO`
  - `*YES`：如果FlashCopy已经完成
- `Connect hosts`：`*ENV`或下面一种：
  - `*CURRENT`：假设当前连接正确
  - `*REQUIRED`：需要修改主机连接。如果未分配卷组，请继续并在适当时运行添加脚本。如果已分配卷组，请验证它们是否适用于此环境。如果没有，终止STRFLASH
  - `*ATTEMPT`：与`*REQUIRED`相同，但如果分配了不正确的卷组，则不要`Vary on` IASP
  - `*NO`：对主机连接不执行任何操作，但仍将环境标记为`*FLASHED`
- `Wait for completion`：`*ENV`, `*YES`或`*NO`。由于IASP的`Vary on`现在是异步完成的，因此必须等待`Vary on`完成才能开始保存
- `Completion timeout`：`1-600`,`*ENV`。在向`QSYSOPR`发送失败消息之前等待改变完成的分钟数
- `Vary on Source`：`*YES`或`*NO`。用于生产节点上的IASP
- `Exit program and library`：
  - `*ENV` 
  - `<name>`：IASP为`AVAILABLE`时要提交的程序名称

### FlashCopy过程
FlashCopy处理过程如下：
- 对集群、设备域、与DS的连接等进行基本检查
- 激活`FlashCopy`节点上的退出程序以执行其他检查：
  - `IASP`是否`varied off`
  - 是否正确安装了`DSCLI`
  - 发现主机连接
  - 确认`Metro Mirror`还是`Golbal Mirror`的源或目标。如果是，检查`FlashCopy`继续进行的情况下复制是否会处于正确状态
- 如果在命令或环境中请求，`STRFLASH`程序会向生产节点发出`quiesce/frcwrt/vary off`
- `STRFLASH`程序向`FlashCopy`节点提交作业以执行以下任务：
  - 将`Flash`状态设置为`20`以允许在不同节点上运行的`STRFLASH`命令知道`Flash`作业已成功提交。如果作业在60秒内未启动，则`STRFLASH`命令将出错
  - 根据IBM i的级别执行`mkflash`脚本或启动`ASP`会话
  - 将`Flash`状态设置为`90`以允许在不同节点上运行的`STRFLASH`命令在需要时继续处理生产系统的恢复并成功结束，如果等待完成是`*NO`
  - 如果需要，将`IASP`进行`Vary on`(包括相关资源的释放、重置和多路径重置)
  - 将`Flash`状态设置为`100`(`*FLASHED`)，如果等待完成为`*YES`，则允许`STRFLASH`成功结束
- 当`FlashCopy`状态变为`90`时，`STRFLASH`程序继续：
  - 如果生产节点被停顿或`varied off`，则提交退出程序以恢复或`vary on.`
  - 如果`Wait for completion`设置为`*YES`，则`STRFLASH`程序将保持活动状态，直到`FlashCopy`完成，并且状态更改为`100`
  - 如果`Wait for completion`设置为`*NO`，则`STRFLASH`程序成功结束

### FlashCopy过程中注意事项
FlashCopy过程中注意事项如下：
- 如果使用所有默认值，则有效进程将与ACS 2.1 FlashCopy进程(`wait for completion` = `*YES`并且`connect hosts`=`*CURRENT`)相同
- 该进程的日志现在分布在最多三个系统中，问题确定需要查看所有相关系统的日志
- `FlashCopy`程序与`FlashCopy`进程分开运行。应对其进行监控以确保其正常运行
- 成功的`STRFLASH`命令并不意味着`IASP`已成功连接并`varied on`。必须在`FlashCopy`过程结束时检查 `IASP`状态

### ENDFLASH命令
命令`ENDFLASH`说明：
- 执行从分区中删除所有FlashCopy `IASP`所需的步骤，并使它们为将来任何 `IASP`环境的`FlashCopy`操作做好准备
- `ENDFLASH`命令不会结束ACS 3.0或IASP Manager 4.0中的多个FlashCopie。只有在集群中只配置了一个 FlashCopy环境时，才能使用默认值`*ONLY`

`end flash`执行的步骤是：
- 属性`*YES`强制`Vary off`所有FlashCopy `IASPs`
- 如果不使用增量`FlashCopy`，移除DS8000上的`flash`
- 修改`CSEDTA`以指示没有Flash `Active`：如果使用增量，表示准备好进行下一次闪存

注意事项：
- `FlashCopy Status`字段必须为`*FLASHED`才能使用此命令

## Metro Mirror
### Metro Mirror概述
&#8195;&#8195;`Metro Mirror`是一个复杂的灾难恢复环境，包括两组独立的卷：`Preferred source volumes`和`preferred target volumes`：
- 生成节点上的数据副本在HA/DR节点上保持同步
- 生产节点在继续之前等待来自HA/DR节点的更新确认
- 性能考虑要求两组卷彼此非常接近，以便最小化等待

### Metro Mirror切换
#### CHKPPRC命令
格式如下：
```
CHKPPRC ENV(<name of IASP>) TYPE(*)
```
`CHKPPRC`命令在绿屏底部显示状态消息以显示进度，此命令完成的步骤：
- 状态信息：获取集群信息
  - 检查`CSE CRG`中指示的`PPRC`状态
  - 识别当前的`HA/DR`节点
  - 识别当前的`Production`(生产)节点
- 状态信息：检查集群节点
  - 检查集群节点是否处于活动状态
  - 检查所有节点是否都在设备域中
- 状态消息：检查HA/DR节点硬件分配
  - 执行DSCLI中的`lspprc`脚本以确保`PPRC`处于`Full Duplex`状态
- 如果配置正确：`CHKPPRC`报告`A PPRC check for IASP CRG <IASPname> completed successfully.`

更多信息及注意事项参考`CHKPPRC`命令详细说明。
#### SWPPRC命令使用*SCHEDULED选项
&#8195;&#8195;`Switch type`使用`SCHEDULED`用于生产和`HA/DR`节点以及存储设备都在运行但它们的角色需要切换时使用。执行以下步骤：
- 向生产节点上的`QSYSOPR`发送`*INQ`消息：`IAS0021 “Perform SWPPRC command for IASP device <IASP name>? (G C)`，回复`G`继续，`C`取消
- 在当前生产节点上使用`*YES`强制关闭IASP
- 使用DSCLI为IASP设备运行`PPRC Failover`任务
- `Release/Reset`HA/DR分区上的`IOP/IOA`资源，并使磁盘正确注册为IASP
- 如果需要，在当前`HA/DR`节点（将成为生产的节点）上将IASP进行`vary on`。默认值为`*YES`

#### SWPPRC命令使用*UNSCHEDULED选项
&#8195;&#8195;当生成节点发生故障，`HA/DR`节点需要承担生成角色时，`Switch type`使用`*UNSCHEDULED`：
- `Unscheduled`切换时，`Auto replicate`默认为`*NO`
- 当`TYPE`设置为`*MMIR`时，多个附加选项`Switch paused MMIR`：
  - 默认值为`*NO`，如果`CHKPRC`发现PPRC已挂起，则`*NO`的默认值会阻止执行切换
  - `*YES`将允许继续进行，即使PPRC被暂停

&#8195;&#8195;此命令尝试完成为计划切换列出的所有步骤，但即使检测到以下错误，也会允许切换发生：
- `Production node`(生产节点)故障
- 生产存储设备故障(即failbackpprc任务无法运行)

&#8195;&#8195;如果交互式运行，第一步会显示`Unscheduled PPRC Switch Warning`面板；如果批量运行，`*INQ`消息`IAS0727`将发送到`HA/DR`节点上的`QSYSOPR`。

&#8195;&#8195;由于故障，计划外的切换很可能是不完整的切换。纠正故障后，必须运行 `SWPPRC *COMPLETE`命令以完成`PPRC`故障转移过程。

#### SWPPRC命令使用*COMPLETE选项
`Switch type`使用`*COMPLETE`用于当错误阻止所有正常切换任务完成时，在计划外切换之后使用。此命令应在当前`HA/DR`节点上运行，以完成`PPRC`故障转移过程。

## Global Mirror
### Global Mirror概述
`Global Mirror`是一个复杂的灾难恢复环境，最多可以包含六组独立的卷：
- 生产节点上的数据副本异步复制到`HA/DR`节点；生产节点在继续之前不会等待来自`HA/DR`节点的更新确认
- 以配置的时间间隔，在源端存储服务器上创建一致性组(CG)。然后，源端存储服务器在确保该一致性组中的所有更改都已发送到目标存储系统后，在目标存储服务器上启动Flashcopy。然后可以在故障转移期间使用此Flashcopy将目标卷返回到上一个已知的良好一致性副本的状态
- 全局镜像可以设置为仅在单个方向或任一方向（对称）上运行

IASP Manager对六个卷集使用以下术语：
- `Source volumes`(A)：这些是`Production node`生产节点(`Preferred Source`首选源)上的应用程序通常使用的卷
- `Preferred Source CG Flash volumes`(E)：当`Global Mirror`以相反的方向，即`Preferred Target`到`Preferred Source`(仅对称symmetrical only)运行时，`Global Mirror`全局镜像使用这些卷在存储服务器上存储一致性信息(CGs)，跟踪自上次形成一致性组以来对`Preferred Source`卷所做的更改
- `Target volumes`(B)：这些卷通常在`HA/DR`节点(Preferred Target)上使用，以维护来自生产节点的数据副本。由于PPRC在全局镜像中异步运行，因此数据更新滞后于源卷的内容
- `Preferred Target CG Flash volumes`(C)：当`Global Mirror`以正常方向(`Preferred Source`到`Preferred Target`)运行时，`Global Mirror`使用这些卷来存储一致性信息(CGs)。目标存储服务器使用这些卷来跟踪自上次形成一致性组以来对`Preferred Target`卷所做的更改
- `Global Mirror target FlashCopy (DCcopy) volumes`(D和F)：这些卷用于在当前作为`Global Mirror`目标的存储服务器上制作可用数据的副本。此副本可以更改以用于测试或保存目的

### Global Mirror环境
#### Global Mirror基础环境(non-symmetrical)
卷集：`A`、`B`和`C`。说明：
- 此环境在正常方向运行时提供灾难/恢复保护，通过使用`C`卷来保持一致副本，但在切换时，它无法在相反方向创建一致副本
- 如果在手动切换回客户首选的生产节点之前被迫切换，客户通常会在`B`卷上运行尽可能短的时间

#### Global Mirror对称环境(symmetrical)
卷集：`A`,`B`,`C`和`E`。说明：
- 对称环境允许`Global Mirror`在正常或反向方向上一致地运行
- 此环境与大多数HA实现类似，只是在发生故障转移时，总会有一些数据不属于一致映像，此数据不可检索。但是，计划的切换具有零数据丢失。

#### Practice Failover–(Global Mirror目标FlashCopy(DCopy))
卷集：`D`和`F`(可以添加到任何GM环境)，说明：
- `Practice Failover`是暂停`Global Mirror`，目标卷保持一致(pprc故障转移和快速恢复)，并为一组卷创建新的 Flashcopy的过程，而不是用`Global Mirror`一致性的卷，然后重新启动`Global Mirror`
- 此过程创建可用于测试或保存的卷

### 使用Global Mirror进行切换和故障转移的限制
&#8195;&#8195;`Global Mirror`可以选择在正常操作期间将某些Flashcopy卷用于多种用途。但可能会造成在释放卷以供全局镜像使用之前可能无法完成`switchover/failover`(切换/故障转移)的情况。
#### 进行Practice Failover(Target-side flash)时阻止switchover或failover
说明如下：
- 在正常方向进行`Practice Failover`(Target-side flash)时，`B`卷建立了多个`FlashCopy`关系。为了最大限度地减少这些多重关系处于活动状态的时间，IASP Manager仅支持将`DCopy`(D)作为完整的磁盘副本
- 计划外切换过程的一部分是执行从`C`卷到`B`卷的快速恢复。如果存在现有的`B`卷与`D`卷的关系，这将失败，因为这将是不受支持的`cascading`(级联)`FlashCopy`
- 如果`D`卷存在于配置中，IASP Manager将始终对它们执行检查，如果`DCopy`(D)未完成，则在切换时检查失败。这时候可以等待`DCopy`完成，也可以使用`WRKCSE`运行`rmflash_GM_Dcopy_PT.script`，然后再次运行`SWPPRC`命令

#### Practice Failover时使用多增量FlashCopy
&#8195;&#8195;IASP Manager现在支持任何闪存环境的`Multiple incremental`(多增量)`FlashCopy`，包括全局镜像的`D-Copy`。通过应用以下规则增强了对此的支持：
- 如果`Multiple incremental`标志打开，则DS故障的`SWPPRC`将首先列出`D-Copy`环境
- 如果`D-Copy`的不同步扇区为零，即后台增量复制已完成，则`flash`将自动移除，并且计划外的切换将继续。 注意：
  - 删除`FlashCopy`关系不会影响磁盘上的数据，`FlashCopy`节点上的备份可以继续
  - 在返回正常完成后，仍应运行`end flash`
- 如果`D-Copy`仍有未完成的扇区要复制，则`SWPPRC`将失败。客户可以选择手动移除`Flash`并重试，或等待后台复制完成

#### 生产节点上使用的Flash Volumes阻止对称切换或故障切换
&#8195;&#8195;在`symmetrical`(对称)环境中运行时，通常使用生产节点`CG FlashCopy`卷(`C`或`E`卷)进行日常备份到磁带。但是，用于保存的`FlashCopy`的参数和功能与全局镜像的`FlashCopy`不同。因此，如果请求对称`switchover/failover`，IASP Manager将确保`preferred source`的`CG FlashCopy`卷没有`FlashCopy`处于活动状态。如果`FlashCopy`处于活动状态，则`CHKPPRC`切换将失败，并且必须等到他们使用完卷或在其`HA/DR`节点上运行`ENDFLASH`命令以删除`FlashCopy`关系。然后再次运行`SWPPRC`命令。

### 切换Global Mirror
#### CHKPPRC命令
&#8195;&#8195;`CHKPPRC`命令检查`CSE CRG`、节点和硬件资源的状态，以确定是否可以成功执行`SWPPRC`(Switch PPRC)命令进行切换：
- 此命令不对`Global Mirror`复制采取任何纠正措施，它只是检查事物的状态
- 在`CHKPPRC`期间发现的所有错误都记录在`qzrdhasm.log`文件中，该文件位于`/qibm/qzrdhasm`目录中

命令示例：
```
CHKPPRC ENV(<name of IASP>) TYPE(*)
```
`CHKPPRC`命令在绿屏底部显示状态消息以显示进度，此命令完成的步骤：
- 状态信息：获取集群信息
  - 检查`CSE CRG`中指示的`PPRC`状态
  - 识别当前的`HA/DR`节点
  - 识别当前的`Production`(生产)节点
- 状态信息：检查集群节点
  - 检查集群节点是否处于活动状态
  - 检查所有节点是否都在设备域中
- 状态消息：检查HA/DR节点硬件分配
  - 执行DSCLI中的`lspprc`脚本以确保`PPRC`处于`Copy Pending`状态
- 如果配置正确：`CHKPPRC`报告`A PPRC check for IASP CRG <IASPname> completed successfully.`

更多信息及注意事项参考`SWPPRC`命令详细说明。
#### SWPPRC命令使用*SCHEDULED选项
&#8195;&#8195;`Switch type`使用`SCHEDULED`用于生产和`HA/DR`节点以及存储设备都在运行但它们的角色需要切换时使用。执行以下步骤：
- 向生产节点上的`QSYSOPR`发送`*INQ`消息：`IAS0021 “Perform SWPPRC command for IASP device <IASP name>? (G C)`，回复`G`继续，`C`取消
- 在当前生产节点上使用`*YES`强制关闭IASP
- 使用DSCLI为IASP设备运行`PPRC Failover`任务
- `Release/Reset`HA/DR分区上的`IOP/IOA`资源，并使磁盘正确注册为IASP
- 如果需要，在当前`HA/DR`节点（将成为生产的节点）上将IASP进行`vary on`。默认值为`*YES`

#### SWPPRC命令使用*UNSCHEDULED选项
&#8195;&#8195;当只有`HA/DR`节点及其存储设备可操作时，`Switch type`使用`*UNSCHEDULED`。由于当前生产节点不可用，因此无法完成`*SCHEDULD`切换期间执行的某些步骤。执行以下步骤：
- 如果以交互方式运行，则在`HA/DR`节点上显示`Unscheduled PPRC Switch Warning`
- 如果批量方式运行，将`*INQ`消息`IAS0727`发送到`HA/DR`节点上的`QSYSOPR`：
  - `IAS0727："An Unscheduled SWPPRC command was issued for IASP device <IASP name>? (G C)"`，回复`G`继续，`C`取消
- 使用DSCLI为IASP设备运行`PPRC Failover`任务
- `Release/Reset`HA/DR分区上的`IOP/IOA`资源，并使磁盘正确注册为IASP
- 如果需要，在当前`HA/DR`节点（将成为生产节点）上将IASP进行`vary on`。默认值为`*YES`

### GMIR复制关系属性
GMIR重要属性选项`D-Copy Flash`：`D-Copy Flash normal`和`D-Copy Flash reversed`：
- 指定同名的单独FlashCopy环境可用于此首选目标或此首选源节点的目标节点
- 当`GMIR`方向为正常或反向时，两个此类FlashCopy环境允许`D-Copy Flash`在当前目标上运行

## Multi-target解决方案
&#8195;&#8195;IASP Manager不再支持`Metro-Global Mirror`，取而代之的是`multi-target`支持。需要使用单独的许可程序Copy Services Manager(CSM)。`multi-target`解决方案支持来自生产节点的两个目标： 
- 对于 MMIR：
  - `H1->H2`PPRC对被命名为`MMIR`
  - `H1->H3`PPRC对被命名为 `MMIR2` 
  - `H2->H3`PPRC对被命名为`MMIR3`
- 对于GMIR：
  - `H1->H2`PPRC对也称为`MMIR`
  - `H1->H3` PPRC对称为`GMIR`
  - `H2->H3` PPRC对称为`GMIR2`

### Metro Mirror-Metro Mirror概述
&#8195;&#8195;三个节点中的任何一个都可以作为两个高速镜像关系的源。两个`Metro Mirror`目标之间还创建了隐式关系，这称为`Multi Target Incremental Resync`(MTIR)关系。关系表如下：

源|活动目标|PPRC方向|*MTIR对
:---:|:---:|:---:|:---:
H1|H2, H3|Both Normal|H2->H3(MMIR3)
H2|H1, H3|H2->H1(MMIR)Reversed;H2->H3(MMIR3)Normal|H1->H3(MMIR2)
H3|H1, H2|H3->H1(MMIR2)Reversed;H3->H2(MMIR3)Reversed|H1->H2(MMIR)

`SWPPRC`(Switch PPRC)命令可以在任何活动目标上运行。
### Metro Mirror-Global Mirror概述
&#8195;&#8195;`Metro Mirror-Global Mirror`结合了`Metro Mirror`的同步可用性和`Global Mirror`的远距离可用性。需要三个系统或分区：`Metro Mirror Source`, `Metro Mirror Target`和`Global Mirror Target`。关系如下表：

源|活动目标/PPRC方向|未激活PPRC对/状态
:---:|:---:|:---:|:---:
H1|H2(MMIR)/Normal;H3(GMIR)/Normal|H2->H3(GMIR2 \*MTIR)
H2|H1(MMIR)/Reversed;H3(GMIR2)/Normal|H1->H2(GMIR \*MTIR)
H3(GMIR Reversed)|H1|H3->H2(GMIR2 \*INELIGIBLE);H1->H2(MMIR \*GCP \*NORMAL)
H3(GMIR2 Reversed)|H2|H3->H1(GMIR \*INELIGIBLE);H2->H1(MMIR \*GCP \*REVERSED)

说明及注意事项：
- 同样`SWPPRC`(Switch PPRC)命令可以在任何活动目标上运行
- 当`H3`是源（`GMIR`或`GMIR2 *REVERSED`）时，`MMIR PPRC`对正在执行全局复制 `*GCP`功能。扇区更改正在发送到`MMIR`目标，但没有一致性。这意味着此节点无法切换到作为`Global Copy`目标

#### Metro Mirror-Global Mirror(MG)其他注意事项
注意事项：
- `CHKPPRC`应针对以下两种环境执行：`MMIR`和`GMIR`
- Metro Mirror-Global Mirror中的`GMIR`部分允许命令`SWPPRC`中使用`SCHEDULED`和`*UNSCHEDULED` 
- 如果`GMIR`(或`GMIR2`)是对称的，则支持`SWPPRC`返回正常方向。否则，需要手动执行步骤 

### 从对称环境的故障转移中恢复
&#8195;&#8195;如果由于站点丢失或生产DS丢失而在对称环境中发生故障转移，则在DS恢复运行后，应使用 `SWPPRC *complete`重新启动复制。

### 在GMIR切换到反向后将非对称MG恢复到生产
&#8195;&#8195;如果`MM/GM`环境中的`GMIR`对被切换并且没有为反向配置的一致性组卷，则切换回`GMIR`源的唯一方法是执行计划切换。不允许计划外的切换，因为目标上的数据将不一致。

## 常用命令
#### CHGPPRC命令
命令示例：
```
                             Change PPRC (CHGPPRC)                         
Type choices, press Enter.                                                
Environment name . . . . . . . .                 Name                     
Type . . . . . . . . . . . . . .                 *GMIR, *GMIR2, *MMIR...  
Option . . . . . . . . . . . . .                 *DETACH, *REATTACH...    
Auto Vary On . . . . . . . . . .   *YES          *YES, *NO       
```
命令选项说明：
- `Environment name`：指定要更改的环境的名称。也指` Independent ASP CRG `名称
- `Type`：Copy Service环境的类型。 此可选参数可用于指定要更改的PPRC环境的类型：
  - `*GMIR` ：具有此环境名称的`PPRC Global Mirroring`环境将被更改
  - `*GMIR2`：作为多目标复制的一部分，具有此环境名称的`PPRC Global Mirroring`环境将被更改
  - `*MMIR`：具有此环境名称的`PPRC Metro Mirroring`环境将被更改
  - `*MMIR2`：作为多目标复制的一部分，具有此环境名称的`PPRC Metro Mirroring`环境将被更改
  - `*MMIR3`：作为多目标复制的一部分，具有此环境名称的`PPRC Metro Mirroring`环境将被更改
  - `*ALL`：该选项将在环境中活动复制对上执行。此值仅对`OPTION(*SUSPEND)`或 `OPTION(*RESUME)`有效
- `Option`：要执行的更改类型：
  - `*SUSPEND`：暂停复制。 仅当两个节点之间的复制由`CSM`(TPC)服务器管理时，此选项才有效
  - `*RESUME`：暂停的复制将恢复。 仅当两个节点之间的复制由`CSM`(TPC)服务器管理时，此选项才有效
  - `*DETACH`：复制将暂停，备份节点将获得对LUN的`read/write`访问权限。仅当复制由`CSM` (TPC) 服务器管理时，此选项才对`MMIR`复制有效
  - `*REATTACH`：对磁盘的`Read/write`访问将被重置，复制将再次开始。仅当复制由`CSM` (TPC) 服务器管理时，此选项才对`MMIR`复制有效
- `Vary`：此可选参数可用于控制在`DETACH`操作成功完成后是否要将`*ASP`设备描述改为`varied on`：
  - `*YES ` ：`*ASP`设备将`varied on`
  - `*NO`：`*ASP`设备将不会`varied on`

### SWPPRC命令
`SWPPRC`(Switch PPRC)命令用于切换PPRC。
#### 命令示例
例如计划性切换`*GMIR`，运行`SWPPRC`命令执行切换，按F4选择参数：
- `Environment name`：指定要切换的环境的名称。此名称也指Independent ASP CRG名称
- `Switch type`选项为`*SCHEDULED`
- `Type`选项为`*GMIR`
- `Auto Vary on`选项默认为`*YES`
- `Auto replicate`选项默认为`*DFT`
- `Switch paused MMIR`选项默认为`*NO`

#### 命令选项说明
PPRC生产系统是否可用？`Switch type`可能的值有：
- `*SCHEDULED`：Independent ASP切换是计划中的。生产系统可用
- `*UNSCHEDULED`：Independent ASP切换是非计划性的。生产系统不可用
- `*COMPLETE `：当先前的生产系统（当前使用备份系统）不可用时，此IASP切换用于从不完整的非计划切换中恢复。此选项仅在当前备份系统上运行有效，即不是运行`SWPPRC *UNSCHEDULED`的节点，而是另一个节点

&#8195;&#8195;`Type`表示IBM i Copy Services环境的类型。此可选参数可用于指定要切换的`PRC`环境的类型。具体选项说明如下：
- `*`：默认选项。具体类型(GMIR、GMIR2、LUN、MMIR、MMIR2或MMIR3)将由命令处理程序解析。如果环境只有一种类型，则允许使用`*`，当有两个或更多时，必须指定
- `*GMIR`：将切换具有此环境名称的`PPRC`全局镜像环境。此环境可以是单个镜像环境，也可以是具有`GMIR2`和`MMIR`环境的多目标复制配置的一部分
- `*GMIR2`：将切换具有此环境名称的多目标`PPRC`全局镜像环境。此环境必须是具有`GMIR`和`MMIR`环境的多目标复制配置的一部分
- `*LUN`：将切换具有此环境名称的`LUN`级别连接切换环境
- `*MMIR`：将切换具有此环境名称的`PPRC Metro Mirroring`环境：
  - 此环境可以是单个镜像环境，也可以是多目标复制配置的一部分
  - 如果它用于多目标复制，则必须同时存在`GMIR`和`GMIR2`环境，或者同时存在`MMIR2`和`MMIR3`环境
- `*MMIR2`：将切换具有此环境名称的多目标`PPRC Metro Mirroring`环境。此环境必须是具有`MMIR`和 `MMIR3`环境的多目标复制环境的一部分
- `*MMIR3`：将切换具有此环境名称的多目标`PPRC Metro Mirroring`环境。此环境必须是具有`MMIR`和 `MMIR2`环境的多目标复制环境的一部分

&#8195;&#8195;`Auto Vary On`此可选参数提供了在PPRC切换完成后使IASP在备份节点上脱机的方法。可选的值：
- `*YES`：将自动为备份节点上的此IASP发出`VRYCFG *ON`命令
- `*NO`：IASP配置不会`varied on`

&#8195;&#8195;`Auto replicate`此可选参数提供了一种方法来覆盖在使用`CHGCSEDTA`命令时为某些类型和切换配置的自动复制设置。可选的值有：
- `*DFT`：使用特定切换类型的默认设置：
    - 对于计划的切换，使用`CHGCSEDTA`命令配置的设置
    - 对于计划外切换，此默认值为`*NO`
- `*YES`：使用`TPC-R`的`MMIR`切换和`GMIR`切换将自动复制
- `*NO`：使用`TPC-R`的`MMIR`切换和`GMIR`切换将不会自动复制

#### Switch paused MMIR参数说明
&#8195;&#8195;该可选参数提供了一种在PPRC暂停后切换`MMIR`的方法。仅适用于计划外切换。可以设置参数说明如下：
- `*NO`：PPRC按预期运行以完成切换
- `*YES`：PPRC可能会暂停以完成切换

#### 补充说明
补充说明：
- 如果使用双CSM服务器，如果`CHKPPRC`和`SWPPRC`无法与主服务器通信，它们将自动执行对备份`CSM`服务器的接管
- `TPC`和`TPC-R`被Spectrum Control或独立的IBM Copy Services Manager(CSM)取代

### CHKPPRC命令
命令格式：
```
CHKPPRC ENV(<name of IASP>) TYPE(*)
```
#### 命令示例及说明
例如检查`*GMIR`，输入`CHKPPRC`命令，按F4在参选选项中选择需求参数：
- `Environment name`：指定要切换的环境的名称。此名称也指Independent ASP CRG名称
- `Type`选择`*GMIR`
- 运行完成后确认返回结果`chkpprc complete successfully`

&#8195;&#8195;还有两个附加选项是跟Message相关的。`Environment name`和`Type`选项的内容说明同`SWPPRC`命令，参考`SWPPRC`命令说明即可。

#### 命令注意事项
命令注意事项：
- 如果环境只有一种类型，则允许使用`TYPE(*)`，当有两个或更多时，必须指定
- 如果使用双CSM服务器，如果`CHKPPRC`和`SWPPRC`无法与主服务器通信，它们将自动执行对备份`CSM`服务器的接管
- 如果执行了CSM服务器接管，并且发现旧的主CSM服务器处于活动状态，`CHKPPRC`和`SWPPRC`将自动以HA模式重新启动CSM，但旧的主服务器现在将成为备份服务器
- 如果`CHKPPRC`要在生产节点和`HA/DR`节点上运行，它们不能同时运行，因为当两者都尝试同时访问集群资源时会发生冲突
- 对于CSM环境，如果CSM服务器丢失，`CHKPPRC`会发出转义消息`IAS00AE`：
  - 这是一条警告消息，表明配置可运行，但需要采取措施才能实现完全冗余
  - 监视`CHKPPRC`功能时，CL程序应区分`IAS0070`(failed)和`IAS00AE`

## 故障排查
### 故障排查过程
未成功完成的第一个指示通常是一条状态消息，例如以下原因示例之一：
-  `A PPRC check for IASP CRG <CRGname> failed.`
  - 生产系统名称=PPRC系统名称
  - 生产系统名称=不存在的系统名称
- `This command must run on the backup node.`
  - PPRC系统名称=生产系统名称
  - SWPPRC尝试在生产系统上运行
- `Clustering not started on node <system name>.`
  - PPRC系统名称=不存在的系统名称
  - 所有群集节点处于非活动状态；自动启动设置为`*NO`
  - TCP/IP未在生产系统上运行
- `VRYCFG failed for device <IASPname>`
  - 在IASP进行`vary on`期间TCP/IP出现failed
  - “Signature”失败

### 状态代码
#### CRG PPRC状态码
CRG PPRC状态码及描述：

状态|描述
:---:|:---
0或`*Ready`|PPRC为`SWPPRC`做好准备
10|PPRC批准收到生产节点操作员或用户的回复
20|PPRC failover任务已完成，收到HA/DR操作员的回复
100或`*INCOMPLETE`|PPRC计划外切换不完整，如果在`Metro Mirror`环境中，则需要`SWPPRC *COMPLETE`。Global Mirror要求手动完成

#### CRG FlashCopy状态码
CRG FlashCopy状态码及描述：

状态|描述
:---:|:---
0或`*NONE`|Flash Copy就绪，可以进行`STRFLASH`
20|Flash Copy完成;操作员在消息`IAS0001`中回复`G`
90|Flash Copy过程已完成;开始`vary on` IASP
100或`*FLASHED`|Flash Copy命令`STRFLASH`完成, 可以进行`ENDFLASH`

#### CRG请求码
CRG请求码如下：

代码|描述
:---:|:---
0|没有请求，没有对任何节点采取任何操作。仅用于设置退出数据
10|Flash Copy硬件检查
20|向生产系统操作员或用户发送Flash Copy查询消息
100|不是请求，用于定义FlashCopy/PPRC请求边界
105|PPRC检查CHKPPRC
120|向生产系统操作员或用户发送PPRC查询消息
122|向备份系统操作员或用户发送PPRC查询消息
205|在FlashCopy节点上执行pre-Flash前检查
211|在生产节点上执行`vary off`（冷闪存）
214|在生产节点上执行`vary on`
215|在生产节点上执行`quiesce`
216|在生产节点上执行`resume`
217|在生产节点上执行`*FRCWRT`
230|在FlashCopy节点上提交FlashCopy程序

## 待补充