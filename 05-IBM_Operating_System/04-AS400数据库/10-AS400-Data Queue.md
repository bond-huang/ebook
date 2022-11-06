# AS400-Data Queue
## Data Queue
官方参考链接：
- [IBM i 7.5 Data queues](https://www.ibm.com/docs/en/i/7.5?topic=classes-data-queues)
- [Managing connections in Java programs](https://www.ibm.com/docs/en/i/7.5?topic=programming-managing-connections-in-java-programs#mngcon)

数据队列类允许Java程序与服务器数据队列进行交互。IBM i数据队列具有以下特征：
- 数据队列允许作业之间的快速通信。因此，这是在作业之间同步和传递数据的绝佳方法
- 许多作业可以同时访问数据队列
- 数据队列上的消息是自由格式的。字段不是必需的，因为它们在数据库文件中
- 数据队列可用于同步或异步处理
- 数据队列上的消息可以按以下方式之一进行排序：
    - `Last-in first-out`(LIFO)。放置在数据队列上的最后(newest)消息是从队列中删除的第一条消息
    - `First-in first-out`(FIFO)。放置在数据队列上的第一条(oldest))消息是从队列中删除的第一条消息
    - `Keyed`。数据队列上的每条消息都有一个与之关联的键。只有通过指定与其关联的键，才能将消息从队列中删除

## Data Queue问题
官方参考问题查询链接：
- [Class DataQueue](https://www.ibm.com/docs/api/v1/content/ssw_ibm_i_75/rzahh/javadoc/com/ibm/as400/access/DataQueue.htm?view=embed)
- [Constant Field Values](https://www.ibm.com/docs/api/v1/content/ssw_ibm_i_75/rzahh/javadoc/constant-values.html#com.ibm.as400.access.ErrorCompletingRequestException.EXIT_PROGRAM_RESOLVE_ERROR)
- [Class ErrorCompletingRequestException](https://www.ibm.com/docs/api/v1/content/ssw_ibm_i_75/rzahh/javadoc/com/ibm/as400/access/ErrorCompletingRequestException.html)

### Data Queue Damaged
&#8195;&#8195;当数据队列损坏时无法检索数据队列条目。数据队列最常因系统关闭不当而受到损坏。使用数据队列的应用程序（包括服务器作业实例）都应在启动关闭之前干净地结束。理想情况下，发送条目的应用应在接收条目的应用之前结束，以便检索所有条目。但是，通常只要在子系统结束之前结束所有应用程序，队列就相当安全。

&#8195;&#8195;数据队列条目通常最多不应存在超过几秒钟，几分之一秒更好。理想情况下，检索队列条目的速度几乎与添加队列条目的速度一样快，因此，队列中不应该有条目丢失。如果条目需要保留较长时间，则应使用更可靠的机制，例如数据库文件。
## 其它常见问题
[Data Queue Host Server Does Not Support DDM Data Queues](https://www.ibm.com/support/pages/node/639587?mhsrc=ibmsearch_a&mhq=Data%20Queue)
## 待补充