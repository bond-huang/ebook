# AS400 Commitment Control
相关参考链接：
- 官方Commitment Control文档中心主页：[IBM i 7.3 Commitment control](https://www.ibm.com/docs/zh/i/7.3?topic=database-commitment-control)
- 库QRECOVERY作用介绍：[IBM Support QRECOVERY Library](https://www.ibm.com/support/pages/node/683805?mhsrc=ibmsearch_a&mhq=QRECOVERY)
- 独立磁盘池中Commitment Control注意事项：[Independent disk pool considerations for commitment definitions](https://www.ibm.com/docs/zh/i/7.3?topic=pools-independent-disk-pool-considerations-commitment-definitions)

## Commitment Control概念
### 概念
&#8195;&#8195;`Commitment Control`是确保数据完整性的功能。它将对资源（例如数据库文件或表）的一组更改定义和处理为一个事务。可以使用`Commitment Control`来设计应用程序，以便在作业、作业中的激活组或系统异常结束时系统可以重新启动应用程序。通过`Commitment Control`，可以确保当应用程序再次启动时，数据库中不会因为先前失败的不完整事务而导致部分更新。

### Commitment Control工作原理
&#8195;&#8195;例如：当用户将资金从储蓄账户转移到支票账户时，会发生多个变化。对用户来说，这种转移似乎是一次更改。但是，由于储蓄账户和支票账户都更新了，数据库发生了不止一个变化。为了保持两个账户的准确性，支票和储蓄账户必须发生所有变化或不发生任何变化。

### 库QRECOVERY或库QRCYxxxxx
&#8195;&#8195;启动`Commitment Control`控制时，会在`QRECOVERY`库中创建`Commitment`定义。每个独立磁盘池或独立磁盘池组都有自己的`QRECOVERY`库版本。在独立磁盘池上，`QRECOVERY`库的名称为`QRCYxxxxx`，其中`xxxxx`为独立磁盘池编号。例如，独立磁盘池33的`QRECOVERY`库的名称是`QRCY00033`。

库`QRECOVERY`或库`QRCYxxxxx`说明：
- 如果独立磁盘池是磁盘池组的一部分，则只有主磁盘池具有`QRCYxxxxx`库
- 与磁盘池组内的对象关联的任何恢复对象存储在该组的主磁盘池的此库中。磁盘池组联机时，这些对象可能需要用于恢复
- 用户不应使用库`QRECOVERY`，因为它仅供系统使用。库`QRECOVERY`包含系统正常运行所需的对象
- 不能使用`SAV`命令来保存`QRECOVERY`或库`QRCYxxxxx`

## 启动Commitment Control
参考链接：[IBM i 7.5 Starting commitment control](https://www.ibm.com/docs/en/i/7.5?topic=control-starting-commitment)
### Commit lock level
参考链接：[IBM i 7.5 Commit lock level](https://www.ibm.com/docs/en/i/7.5?topic=control-commit-lock-level)

&#8195;&#8195;在`STRCMCTL`(Start Commitment Control)命令上为`LCKLVL`参数指定的值将成为打开并置于承诺定义的承诺控制下的数据库文件的默认记录锁定级别。打开本地数据库文件时，无法覆盖默认级别的记录锁定。但是，SQL访问的数据库文件使用当前的SQL隔离级别，该级别在针对它发布第一条SQL语句时生效。锁定级别必须根据用户需要、允许的等待时间和最常用的释放过程来指定：
- `*CHG Lock Level`：如果要保护已更改的记录不受同时运行的其他作业的更改，请使用此值。对于在承诺控制下打开的文件，锁在事务期间保持。对于未在承诺控制下打开的文件，记录上的锁仅从记录被读取到更新操作完成时保持
- `*CS Lock Level`：使用此值可以保护已更改的记录和已检索的记录不受同时运行的其他作业的更改。检索到的未更改的记录仅在发布或检索到其他记录之前受到保护。CS锁定级别确保其他作业无法读取此作业已读取的更新记录。此外，程序无法读取已在另一个作业中使用记录锁定类型`*update`锁定的更新记录，直到该作业访问不同的记录
- `*ALL Lock Level`：使用此值可以保护受承诺控制的已更改记录和已检索记录，使其不受同时在承诺控制下运行的其他作业的更改。检索或更改的记录将受到保护，直到下一次提交或回滚操作。`*ALL`锁定级别确保其他作业无法访问此作业已读取的更新记录。这与正常的锁定协议不同。当锁定级别指定为`*ALL`时，如果在另一个作业中使用记录锁定类型`*update`锁定记录，则即使未读取更新的记录也无法访问

## Commitment Control性能
参考链接：
- [Optimizing performance for commitment control](https://www.ibm.com/docs/en/i/7.2?topic=control-optimizing-performance-commitment)

## ROLLBACK
参考链接：
- [IBM i 7.2 ROLLBACK](https://www.ibm.com/docs/en/i/7.2?topic=statements-rollback) 

## Commitment Control常用操作
### 查看Commitment Definitions
查看有多种方式进行：
- 运行命令`WRKCMTDFN`按`F4`输入相应作业信息查看
- 直接运行命令，例如：`WRKCMTDFN JOB(104071/CBC/CBCVR38081)`,选择`5=Display status`
- `WRKJOB`选择`16. Display commitment control status, if active`，然后选择`5=Display`

## Commitment Control常见问题
常见问题官方参考链接：
- [Commitment control during abnormal system or job end](https://www.ibm.com/docs/en/i/7.2?topic=control-commitment-during-abnormal-system-job-end) 
- [Trigger and application programs that are not under commitment control](https://www.ibm.com/docs/en/i/7.2?topic=oiiawtp-trigger-application-programs-that-are-not-under-commitment-control)    
- [Commitment control during activation group end](https://www.ibm.com/docs/en/i/7.2?topic=control-commitment-during-activation-group-end)