# PowerHA Tools for IBM i - IASP Manager
&#8195;&#8195;PowerHA Tools IASP Manager是IBM Lab Services编写的高可用性产品。它专为使用IBM存储和/或将PowerHA与独立辅助存储池(IASP)解决方案结合使用的IBM i客户而设计。官方参考链接：[PowerHA Tools for IBM i - IASP Manager](https://www.ibm.com/support/pages/node/1126029)。
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

## Metro Mirror
### Metro Mirror概述
&#8195;&#8195;`Metro Mirror`是一个复杂的灾难恢复环境，包括两组独立的卷：`Preferred source volumes`和`preferred target volumes`：
- 生成节点上的数据副本在HA/DR节点上保持同步
- 生产节点在继续之前等待来自HA/DR节点的更新确认
- 性能考虑要求两组卷彼此非常接近，以便最小化等待

## Global Mirror
### Global Mirror概述
`Global Mirror`是一个复杂的灾难恢复环境，最多可以包含六组独立的卷：
- 生产节点上的数据副本异步复制到 `HA/DR`节点；生产节点在继续之前不会等待来自`HA/DR`节点的更新确认
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

## Multi-target解决方案
&#8195;&#8195;IASP Manager不再支持`Metro-Global Mirror`，取而代之的是`multi-target`支持。需要使用单独的许可程序Copy Services Manager(CSM)。`multi-target`解决方案支持来自生产节点的两个目标： 
- 对于 MMIR：
  - `H1->H2`PPRC对被命名为`MMIR`
  - `H1->H3`PPRC对被命名为 `MMIR2` 
  - `H2->H3`PPRC对被命名为`MMIR3`
- 对于GMIR：
  - `H1->H2`PPRC对也称为`MMIR`
  - ` H1->H3` PPRC对称为`GMIR`
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
H1|H2(MMIR)/Normal;H3(GMIR)/Normal|H2->H3(GMIR2 *MTIR)
H2|H1(MMIR)/Reversed;H3(GMIR2)/Normal|H1->H2(GMIR *MTIR)
H3(GMIR Reversed)|H1|H3->H2(GMIR2 *INELIGIBLE);H1->H2(MMIR *GCP *NORMAL)
H3(GMIR2 Reversed)|H2|H3->H1(GMIR *INELIGIBLE)|H2->H1(MMIR *GCP *REVERSED)

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

#### Switch type选项说明
PPRC生产系统是否可用？可能的值有：
- `*SCHEDULED`：Independent ASP切换是计划中的。生产系统可用
- `*UNSCHEDULED`：Independent ASP切换是非计划性的。生产系统不可用
- `*COMPLETE `：当先前的生产系统（当前使用备份系统）不可用时，此IASP切换用于从不完整的非计划切换中恢复。此选项仅在当前备份系统上运行有效，即不是运行`SWPPRC *UNSCHEDULED`的节点，而是另一个节点

#### Type选项说明
&#8195;&#8195;`Type`表示IBM i Copy Services环境的类型。此可选参数可用于指定要切换的`PRC`环境的类型。具体选项说明如下：
- `*`：默认选项。具体类型(GMIR、GMIR2、LUN、MMIR、MMIR2或MMIR3)将由命令处理程序解析。只能在此环境名称中配置这些类型中的一种
- `*GMIR`：将切换具有此环境名称的`PPRC`全局镜像环境。此环境可以是单个镜像环境，也可以是具有`GMIR2`和`MMIR`环境的多目标复制配置的一部分
- `*GMIR2`：将切换具有此环境名称的多目标`PPRC`全局镜像环境。此环境必须是具有`GMIR`和`MMIR`环境的多目标复制配置的一部分
- `*LUN`：将切换具有此环境名称的`LUN`级别连接切换环境
- `*MMIR`：将切换具有此环境名称的`PPRC Metro Mirroring`环境：
  - 此环境可以是单个镜像环境，也可以是多目标复制配置的一部分
  - 如果它用于多目标复制，则必须同时存在`GMIR`和`GMIR2`环境，或者同时存在`MMIR2`和`MMIR3`环境
- `*MMIR2`：将切换具有此环境名称的多目标`PPRC Metro Mirroring`环境。此环境必须是具有`MMIR`和 `MMIR3`环境的多目标复制环境的一部分
- `*MMIR3`：将切换具有此环境名称的多目标`PPRC Metro Mirroring`环境。此环境必须是具有`MMIR`和 `MMIR2`环境的多目标复制环境的一部分

#### Auto Vary On选项说明
此可选参数提供了在PPRC切换完成后使IASP在备份节点上脱机的方法。可选的值：
- `*YES`：将自动为备份节点上的此IASP发出`VRYCFG *ON`命令
- `*NO`：IASP配置不会`varied on`

#### Auto replicate选项说明
&#8195;&#8195;此可选参数提供了一种方法来覆盖在使用`CHGCSEDTA`命令时为某些类型和切换配置的自动复制设置。可选的值有：
- `*DFT`：使用特定切换类型的默认设置：
    - 对于计划的切换，使用`CHGCSEDTA`命令配置的设置
    - 对于计划外切换，此默认值为`*NO`
- `*YES`：使用`TPC-R`的`MMIR`切换和`GMIR`切换将自动复制
- `*NO`：使用`TPC-R`的`MMIR`切换和`GMIR`切换将不会自动复制

补充说明：
- TPC和TPC-R被Spectrum Control或独立的IBM Copy Services Manager(CSM)取代

#### Switch paused MMIR参数说明
&#8195;&#8195;该可选参数提供了一种在PPRC暂停后切换`MMIR`的方法。仅适用于计划外切换。可以设置参数说明如下：
- `*NO`：PPRC按预期运行以完成切换
- `*YES`：PPRC可能会暂停以完成切换

### CHKPPRC命令
#### 命令示例
例如检查`*GMIR`，输入`CHKPPRC`命令，按F4在参选选项中选择需求参数：
- `Environment name`：指定要切换的环境的名称。此名称也指Independent ASP CRG名称
- `Type`选择`*GMIR`
- 运行完成后确认返回结果`chkpprc complete successfully`

&#8195;&#8195;还有两个附加选项是跟Message相关的。`Environment name`和`Type`选项的内容说明同`SWPPRC`命令，参考`SWPPRC`命令说明即可。

## 待补充

