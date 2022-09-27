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

## 待补充

