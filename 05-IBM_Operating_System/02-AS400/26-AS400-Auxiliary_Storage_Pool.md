# AS400 Auxiliary_Storage_Pool
&#8195;&#8195;AS400磁盘管理内容中也有涉及此方面的，需要学习和了解的知识也比较多，磁盘池单独列出来。磁盘池在基于字符的界面中也称为`Auxiliary Storage Pool`(ASP)，是系统上一组磁盘单元的软件定义。相关官方参考链接：
- [IBM i 7.5 Disk pools](https://www.ibm.com/docs/en/i/7.5?topic=management-disk-pools)
- [Managing auxiliary storage pools](https://www.ibm.com/docs/en/i/7.5?topic=system-managing-auxiliary-storage-pools)
- [Adding disk units to an existing auxiliary storage pool](https://www.ibm.com/docs/en/i/7.5?topic=pools-adding-disk-units-existing-asp)
- [Deleting an auxiliary storage pool](https://www.ibm.com/docs/en/i/7.5?topic=pools-deleting-asp)
- [Working with ASP trace and ASP balance](https://www.ibm.com/docs/en/i/7.5?topic=pools-working-asp-trace-asp-balance)
- [Attach Independent Disk pool](https://www.ibm.com/docs/en/i/7.2?topic=pools-attach-independent-disk-pool)

## IASP
### IASP vary on
官方参考链接：
- [Display ASP Status(DSPASPSTS)](https://www.ibm.com/docs/en/i/7.5?topic=ssw_ibm_i_75/cl/dspaspsts.htm)
- [IASP: Example Output Providing Steps From Vary On and Vary Off of IASP](https://www.ibm.com/support/pages/iasp-example-output-providing-steps-vary-and-vary-iasp)
- [IBM Support iASP Vary-On Time Variables](https://www.ibm.com/support/pages/iasp-vary-time-variables)
- [Jobs Using IASP and Impact during Vary On/Off](https://www.ibm.com/support/pages/jobs-using-iasp-and-impact-during-vary-onoff)
-[IBM Support EDTRBDAP for IASP During Vary On](https://www.ibm.com/support/pages/edtrbdap-iasp-during-vary)
- [IBM Support IASP Varyon Time](https://www.ibm.com/support/pages/iasp-varyon-time)

#### IASP vary on步骤
IASP vary on涉及的步骤：
- 0 Cluster vary job submission
- 1 Waiting for devices - none are present
- 2 Waiting for devices - not all are present
- 3 DASD checker
- 4 Storage management recovery
- 5 Synchronization of mirrored data 
- 6 Synchronization of mirrored data - 2
- 7 Scanning DASD pages
- 8 Directory recovery - permanent directory
- 9 Authority recovery
- 10 Context rebuild
- 11 Journal recovery
- 12 Database recovery
- 13 Journal synchronization
- 14 Commit recovery
- 15 Database initialization
- 16 Database recovery
- 17 Commit initialization
- 18 User profile creation
- 19 UID/GID mismatch correction
- 20 Library validation (context rebuild) 
- 21 Database, journal, commit - 1
- 22 Identifying interrupted DDL operations
- 23 Recovering system managed journals
- 24 POSIX directory recovery
- 25 Database, journal, commit - 2
- 26 Commit recovery - 2
- 27 Cleaning up journal receivers
- 28 Transfer from danglers 
- 29 Database access path recovery
- 30 Database cross-reference file merge
- 31 SPOOL intialization
- 32 Image catalog synchronization
- 33 Command analyzer recovery 
- 34 Catalog validation 
- 35 End

#### IASP vary on时间
影响IASP Vary On时间的变量：
- 处理器
- 内存
- 数据库条目数
- iASP 的大小
- iASP中的对象总数
- 损坏的权限列表扩展的
- 删除损坏的权限列表扩展
- 删除损坏的用户配置文件扩展
- 创建用户配置文件或转移dangling/orphaned对象的所有权
- 对 IPL 步骤进行了任何更改或升级后的首次vary on iASP

IASP Vary On时间上数据库相关影响官方说明：
- `Vary on ASP`组所需的时间受`SYSBAS`中数据库对象数量的影响
- 每个`ASP`组代表一个单独的数据库实例，每个都包含自己的一组交叉引用和目录文件
- 由于`ASP`组数据库还包括`SYSBAS`中的文件，因此`Vary on`将来自 `SYSBAS`中的`cross-reference`和目录文件的信息合并到`ASP`组的`cross-reference`和目录文件中(varyon作业日志中的消息CPI32A1-Starting XREF merge)
- `SYSBAS`中的文件中包含的记录越多，执行`Vary on`处理的数据库合并功能所需的时间就越长
- 要尽可能实现最快的联机时间，可以限制`SYSBAS`中存在的文件
- `IASP`的推荐使用结构是将用户的应用程序数据对象的大部分放入`IASP`中，而将最少数量的非程序对象放在`SYSBAS`中，即`system disk pool`和所有已配置的`basic disk pools`。`system disk pool`和`basic user disk pools(SYSBAS)`主要包含操作系统对象、许可程序产品库和很少的用户库

## ASP数据平衡
命令参考：
- [Check ASP Balance (CHKASPBAL) - IBM 文档](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/chkaspbal.html)
- [End ASP Balance (ENDASPBAL) - IBM 文档](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/endaspbal.html)
- [Start ASP Balance (STRASPBAL) - IBM 文档](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/straspbal.html)
- [Trace ASP Balance (TRCASPBAL) - IBM 文档](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/trcaspbal.html)

### STRASPBAL命令
&#8195;&#8195;`STRASPBAL`(Start ASP Balance)命令允许用户为一个或多个ASP启动辅助存储池(ASP)平衡功能。可以启动的ASP平衡类型包括：
- `Capacity balancing`，ASP中所有单元上的数据都将进行平衡，因此每个单元的已用和未使用空间百分比相等：
  - 当将新单元添加到ASP时，此平衡很有用
- `Usage balancing`，重新分发ASP中每个“低”利用率单元上的“低”使用量数据，以平衡指定ASP中每个单元的臂利用率的未来使用情况。 
  - 在`TRCASPBAL`(Trace ASP Balance)命令控制的跟踪收集使用情况统计信息之前，无法执行`*USAGE`平衡
  - 当ASP包含大容量磁盘机时，使用率均衡非常有用
- `Hierarchical Storage Management (HSM) balancing`（分层存储管理平衡），重新分发ASP中每个单元上的“高”使用量和“低”使用率数据，以便“高”使用率数据驻留在高性能设备上，“低”使用率数据驻留在低性能单元上。
  - 在`TRCASPBAL`(Trace ASP Balance)命令控制的跟踪收集使用情况统计信息之前，无法执行 HSM 平衡
  - 当ASP包含压缩磁盘单元或包含固态磁盘(SSD)单元和硬盘驱动器(HDD)单元的混合时，允许 HSM 平衡
  - `SUBTYPE `关键字用于指定哪些单位将参与均衡
- `Media Preference (MP) balancing`(媒体首选项平衡) ，操作系统(OS)为数据分配`media preference`属性。当ASP混合具有高性能和低性能磁盘机时，OS会尝试将具有高性能媒体首选项的数据放在快速磁盘机上。由于空间限制、磁盘配置更改以及试图利用高性能单元的速度，高性能单元上的数据可能并不总是具有高性能“介质首选项”属性的专用数据。MP在高性能磁盘机上查找没有高性能介质首选项属性的数据，并将该数据移动到低性能磁盘机。
  - 以这种方式平衡的ASP必须包含固态磁盘(SSD)单元和硬盘驱动器(HDD)单元的混合
  - SUBTYPE 关键字用于指定余额的范围：
    - 指定`SUBTYPE(*CALC)`时，没有高性能介质首选项的数据将从SSD单元移动到HDD单元，具有高性能媒体首选项的数据将从HDD单元移动到SSD单元
    - 指定子类型`*SSD`时，没有高性能介质首选项的数据将从SSD单元移动到HDD单元
    - 指定子类型`*HDD`时，具有高性能介质首选项的数据将从HDD单元移动到SSD单元
- `Move data from units`，此选项可用于减少移除磁盘相关的停机时间。可以通过指定`UNIT（单元编号）`和 `TYPE(*ENDALC)`来标记计划删除的磁盘单元以结束数据分配。
  - 对于标记为`*ENDALC`的所有磁盘单元，指定`TYPE(*MOVDTA)`会将数据从标记的磁盘单元移动到同一 ASP 中的磁盘单元
  - 要恢复标有`*ENDALC`的设备分配，指定`UNIT(设备编号)`和`TYPE(*RSMALC)`。将再次允许向该单位分配新的数据
  - `CHKASPBAL`(Check ASP Balance)命令可用于确定当前标记为`*ENDALC`的单位

&#8195;&#8195;用户可以指定函数要为每个平衡的ASP运行的时间限制，也可以将平衡设置为运行完成。如果需要结束平衡函数，请使用`ENDASPBAL`(End ASP Balance)命令。
- 为每个 ASP 启动平衡功能时，将向系统历史记录 （QHST） 日志发送一条消息
- 当平衡功能完成或结束时，也会向 QHST 日志发送一条消息。

&#8195;&#8195;如果平衡功能运行几个小时后停止，则当平衡功能重新启动时，它将从中断的位置继续。这允许在几天的下班时间运行平衡。     
&#8195;&#8195;示例启动ASP 1 的`*CAPACITY` ASP平衡功能。平衡功能将运行，直到每个单元的容量都已平衡：
```
STRASPBAL  ASP(1) TIMLMT(*NOMAX) TYPE(*CAPACITY)
```
&#8195;&#8195;示例为系统ASP和已运行`TRCASPBAL`命令的每个已配置的基本ASP启动`*USAGE` ASP平衡函数。每个平衡功能的时间限制为60分钟。六十分钟后，任何未完成的平衡功能将结束。
```
STRASPBAL   ASP(*ALL)  TIMLMT(60)  TYPE(*USAGE)
```
&#8195;&#8195;示例为名为`MYASP1`的ASP设备启动`*CAPACITY` ASP平衡函数。平衡功能将一直运行到完成。
```
STRASPBAL   ASPDEV(MYASP1)  TIMLMT(*NOMAX)  TYPE(*CAPACITY)
```
&#8195;&#8195;下面示例中第一个命令将单元`11`、`12`和`13`标记为不再接收新的分配。第二个命令开始将数据移出标记的单位。建议在非高峰时间执行`*MOVDTA` ASP 平衡功能。
```
STRASPBAL   UNIT(11 12 13)  TYPE(*ENDALC)
STRASPBAL   TYPE(*MOVDTA)
```
&#8195;&#8195;示例启动`MYASP`的`*HSM` ASP平衡函数。平衡功能会将低使用率数据从高性能单位移动到低性能单位，将高使用率数据从低性能单位移动到高性能单位。它以低优先级运行。如果平衡未在600分钟内完成，则结束。
```
STRASPBAL TYPE(*HSM) ASP(MYASP) TIMLMT(600) PRIORITY(*LOW)
```
&#8195;&#8195;示例启动ASP 1 的`*HSM` ASP平衡函数。平衡功能会将低使用率数据从高性能HDD单元移动到低性能HDD单元，将高使用率数据从低性能HDD单元移动到高性能HDD单元。它以低优先级运行。如果未在240分钟内完成，则结束。
```
STRASPBAL TYPE(*HSM) ASP(1) TIMLMT(240) SUBTYPE(*HDD)
```
&#8195;&#8195;示例启动 ASP 1 的`*MP` ASP平衡函数。平衡功能会将具有“媒体首选项”属性的数据从HDD单元移动到SSD单元，将没有该属性的数据从SSD单元移动到HDD单元。它以高优先级运行。如果未在120分钟内完成，则结束。
```
STRASPBAL TYPE(*MP) ASP(1) TIMLMT(120) PRIORITY(*HIGH)
```
&#8195;&#8195;示例启动ASP 1 的`*MP`ASP平衡函数。平衡功能会将具有“媒体首选项”属性的数据从HDD单元移动到SSD单元。它以高优先级运行。如果未在120分钟内完成，则结束。
```
STRASPBAL TYPE(*MP) ASP(1) TIMLMT(240) SUBTYPE(*HDD)
```
### TRCASPBAL命令
&#8195;&#8195;启动跟踪函数，该函数将识别每个单元上的“高”和“低”使用数据。使用情况平衡活动运行完成后，将清除跟踪信息。

## 待补充
