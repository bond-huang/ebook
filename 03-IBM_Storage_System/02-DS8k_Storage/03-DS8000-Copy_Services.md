# Copy Services
官方参考链接：
- [DS8870 7.3 Copy Services](https://www.ibm.com/docs/en/ds8870/7.3?topic=managing-copy-services)
- [DS8880 8.5.4 Copy Services](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=help-copy-services)
- [DS8880 8.5.4 IBM Copy Services Manager](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=interfaces-copy-services-manager)
- [DS8880 8.5.4 Copy Services commands](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=commands-copy-services)
- [DS8880 8.5.4 Copy Services functions](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=interface-copy-services-functions)
- [DS8880 8.5.4 Copy Services functions with IBM i operating system](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=services-copy-functions-i-operating-system)

官方参考红皮书：
- IBM DS8000 Copy Services Updated for IBM DS8000 Release 9.1
- IBM DS8000 Safeguarded Copy(Updated for DS8000 Release 9.2.1)

## Copy Services简介
### Point-in-time copy functions
####  FlashCopy
&#8195;&#8195;`FlashCopy`支持在DS8000存储系统中创建卷或一组数据（卷的子集）的时间点副本。使用 `FlashCopy`，两个副本都可以立即用于读取和写入操作：
- `FlashCopy`也称为时间点复制、快速复制或零时间复制（t0复制）
- 设置`FlashCopy`操作时，会在源卷和目标卷之间建立关系，并创建源卷的位图
- 创建此关系和位图后，可以访问目标卷，就好像所有数据都已物理复制一样
- 如果使用后台复制选项建立`FlashCopy`，则实际数据会从源卷复制到目标卷：
  - 如果在后台复制期间访问源卷或目标卷，`FlashCopy`将管理这些I/O请求，并促进对源副本和目标副本的读取和写入
  - 当所有数据都复制到目标时，`FlashCopy`关系结束，除非它被设置为持久关系（例如，用于增量复制）
  - 在所有数据复制到目标之前，用户可以随时撤销`FlashCopy`关系
- `FlashCopy`操作可以在任何类型的卷之间执行，无论是完全配置还是精简配置，具有大范围或小范围

#### Remote Pair FlashCopy (Preserve Mirror)
&#8195;&#8195;`Remote Pair FlashCopy`或`Preserve Mirror`克服了之前将`FlashCopy`复制到高速镜像源卷上的解决方案的缺点：
- 此配置可以减少`FlashCopy`后台复制和高速镜像重新同步正在进行时存在的恢复点目标(RPO)
- `Remote Pair FlashCopy`为数据复制、数据迁移、远程复制和灾难恢复任务提供了解决方案
- `Preserve Mirror`保留了现有的`FULL DUPLEX`的`Metro Mirror`状态

#### Cascading FlashCopy
&#8195;&#8195;从DS8000 8.3版本开始，卷既可以是一个`FlashCopy`关系中的源，也可以是第二个`FlashCopy`关系中的目标，称为`Cascading FlashCopy`。
### Business-continuity functions
&#8195;&#8195;DS8000提供了一组灵活的数据镜像技术，允许在两个或多个存储系统上的卷之间进行复制，可以将这些功能用于数据备份和灾难恢复等目的。
#### Metro Mirror
&#8195;&#8195;`Metro Mirror`是两个DS8000之间的同步复制解决方案，其中在`I/O`被认为完成之前，本地和远程卷上的写入操作都已完成：
- `Metro Mirror`用于在存储系统发生故障时不会丢失数据的环境。
- 由于在考虑写入完成之前，数据会同步传输到二级存储系统，因此主存储和二级存储系统之间的距离会影响应用程序对写入的响应时间
- `Metro Mirror`支持的距离为300公里（186 英里）

#### Global Copy
&#8195;&#8195;`Global Copy`是一种异步远程复制功能，可用于比高速镜像更长的距离。 `Global Copy`适用于远程数据迁移、异地备份以及几乎无限距离传输非活动数据库日志：
- `Global Copy`也用作`Global Mirror`的数据传输机制
- 使用`Global Copy`，在将数据复制到从存储系统之前，在主存储系统上完成写操作，从而避免主系统的性能受到写到从存储系统所需的时间的影响。这种方法允许站点相隔很远的距离
- 写入主DS8000的所有数据都会传输到辅助DS8000，但不一定按照写入主DS8000的顺序。这种方法意味着辅助节点上的数据不是时间一致的
- 使用辅助卷上的数据需要使用一些技术来确保一致性

#### Global Mirror
&#8195;&#8195;`Global Mirror`是两站点、长距离、异步、远程复制技术。该解决方案基于现有的`Global Copy`和`FlashCopy`功能：
- 使用`Global Mirror`，主机写入主站点存储系统的数据异步镜像到备站点存储系统，在远程站点自动维护一致的数据副本
- `Global Mirror`操作提供了支持本地和远程站点之间无限距离的操作的好处，这些操作仅受网络能力和通道扩展技术的限制
- 它还可以在远程站点提供一致且可重新启动的数据副本，对本地站点的应用程序影响最小
- 通过支持故障转移和故障恢复模式保持本地和远程站点的有效同步的能力有助于减少在计划内或计划外中断后切换回本地站点所需的时间
- 通过特殊的管理步骤（在本地主存储单元的控制下），通过在远程站点的存储单元上使用`FlashCopy`，自动维护和定期更新数据的一致副本

#### Three-site Metro/Global Mirror with Incremental Resync
&#8195;&#8195;`Metro/Global Mirror`将`Metro Mirror`和`Global Mirror`结合在一起，以提供实施`3`站点灾难恢复解决方案的可能性：
- 生产系统正在使用本地站点的存储，该存储使用`Metro Mirror`同步复制到中间站点
- `Metro Mirror`关系的辅助卷进一步用作级联`Global Mirror`关系的主卷，将数据复制到远程灾难恢复站点

&#8195;&#8195;此配置为在各种灾难情况下的恢复提供了弹性和灵活的解决方案。用户还可以从将数据同步复制到充当中间站点的关闭位置中受益。它还可以跨越几乎无限的距离复制数据，可以在每个位置随时提供数据一致性。

&#8195;&#8195;使用增量重新同步，可以更改复制关系的复制目标目的地，而无需数据的完整副本。 例如，当中间站点因灾难而发生故障时，可以使用此功能：
- 在这种情况下，会建立一个从本地到远程站点的`Global Mirror`，绕过中间站点
- 当中间站点再次可用时，增量重新同步用于将其带回`Metro/Global Mirror`设置

#### IBM Multiple Target Peer-to-Peer Remote Copy
&#8195;&#8195;多目标对等远程复制(PPRC)通过提供在单个主卷上具有两个`PPRC`关系的能力以及另一个远程站点提供额外数据保护的能力，从而增强了多站点灾难恢复环境。多目标`PPRC`提供以下增强功能：
- 将数据从一个主（本地）站点镜像到两个辅助（远程）站点
- 在以下灾难恢复解决方案中提供增强的功能和灵活性：
    - 同步复制
    - 异步复制
    - 同步复制和异步复制配置的组合
- 改进了级联`Metro/Global Mirror`配置并简化了一些过程
- 在多目标`PPRC`之前，主卷可以仅将数据镜像到一个辅助卷。使用多目标`PPRC`，同一个主卷可以有多个目标，从而允许将数据从单个主站点镜像到两个目标站点

IBM官方参考链接：
- [DS8880 8.5.4 How Metro/Global Mirror works](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=mirror-how-metroglobal-works)
- [DS8880 8.5.4 Metro/Global Mirror planning considerations](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=mirror-metroglobal-planning-considerations)

#### SafeGuarded Copy
&#8195;&#8195;`SafeGuarded Copy`(SGC)是从DS8880开始提供的一项功能，可为关键数据提供逻辑损坏保护。`SafeGuarded Copy`功能提供了多个备份副本，可以在恶意软件、黑客攻击、恶意破坏以及任何可能导致逻辑损坏或破坏主数据的操作的情况下恢复：
- 主机无法访问受保护的副本
- 受保护的备份副本（SG备份）可以安排为每天定期创建多次（例如，每小时备份副本）
- 这些备份副本可用于将数据恢复到指定的时间点
- 受保护的备份副本可用于诊断从验证到恢复的生产问题

&#8195;&#8195;`SafeGuarded Copy`通过IBM Copy Services Manager(CSM)6.2.3及更高版本或GDPS 4.2及更高版本进行管理。通过任一管理工具，都可以定义`SG`备份的过期规则、创建和恢复。
## 待补充