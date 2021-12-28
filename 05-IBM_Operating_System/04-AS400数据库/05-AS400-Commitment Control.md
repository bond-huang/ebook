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