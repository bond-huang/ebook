# DB2-常见问题
记录一些日常运维过程中遇到的SQL messages，方便查阅。          
IBM 官方SQL messages说明网站：[IBM DB2 11.1 SQL messages](https://www.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/doc/rsqlmsg.html)
### SQL6048N
&#8195;&#8195;在使用db2start命令启动DB2的时候报错SQL6048N，一般是hosts表有问题，检查下`/etc/hosts`，注意大小写。此情况通常出现在数据库在新的操作系统上启动时候发生。当然还可能是其它原因，IBM官方详细描述：[db2start failing with SQL6048N error](https://www.ibm.com/support/pages/db2start-failing-sql6048n-error)

### SQL0912N
官方说明：[DB2 10.5 SQL0912N](https://www.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/com.ibm.db2.luw.messages.sql.doc-gentopic2.html#sql0912n)

由于没有足够的内存用于锁定请求，因此已达到数据库的最大锁定数量。出现此错误时候可能会导致应用批处理失败，在db2diag.log出现大量错误，错误示例如下：
```
2020-09-10-08.46.12.137491+480 I3480316188A583      LEVEL: Error
PID     : 20119590             TID : 21562          PROC : db2sysc 0
INSTANCE: ftpinst              NODE : 000           DB   : testDB
APPHDL  : 0-35376              APPID: *LOCAL.ftpinst.200910004557
AUTHID  : FTPUSR               HOSTNAME: testDB1
EDUID   : 21562                EDUNAME: db2agent (FTPDB) 0
FUNCTION: DB2 UDB, catalog services, sqlrlNonRestartCheckIndex, probe:100
RETCODE : ZRC=0x8510000A=-2062548982=SQLP_LFUL
          "Lock list full - SQL0912 Reason code 1"
          DIA8310C Lock list was full.
```
根据不同的Reason code选择解决方法：
- Reason code 1：增加LOCKLIST数据库配置参数，该参数管理分配给本地锁管理器的锁内存
- Reason code 2：增加CF_LOCK_SZ数据库配置参数，该参数管理分配给全局锁管理器的锁内存

关于LOCKLIST官方说明:[DB2 LOCKLIST](https://www.ibm.com/support/knowledgecenter/zh/SSEPGG_11.5.0/com.ibm.db2.luw.admin.config.doc/doc/r0000267.html)
### SQL20048N 
官方说明：[DB2 SQL20048N](https://www.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/com.ibm.db2.luw.messages.sql.doc-gentopic15.html#sql20048n)

示例故障现象和报错：
某系统每晚22:00跑定时任务存储过程生成报表数据，但每月的6-7号晚上跑存储过程都会失败，导致报表数据为空。经查，是调用存储过程`QU_REPORT_DATA_LEVEL_TWO()`时候报错，错误信息为：`Error: SQLCODE=-20448,SQLSTATE=22007, SQLERRMC=;hh24:mi:ss,DRIVER=4.0.100`。

解决方法：      
&#8195;&#8195;报错DB2版本是9.5FP10，使用JDBC版本是4.0.100，此JDBC版本是DDB9.5 FP0GA版本时候发布的，建议下载9.5FP10对应的最新版本的JDBC。同理其它版本有类似不匹配问题建议使用对应版本。

IBM DB2 JDBC Driver版本和下载：[https://www.ibm.com/support/pages/node/382667](https://www.ibm.com/support/pages/node/382667)

### SQL0727N
官方说明：[DB2 SQL0727N](https://www.ibm.com/support/knowledgecenter/zh/SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/com.ibm.db2.luw.messages.sql.doc-gentopic2.html#sql0727n)

描述：隐式系统操作类型`action-type`期间发生错误。返回的错误信息包括SQLCODE`sqlcode`，SQLSTATE `sqlstate`和消息令牌`token-list`。

&#8195;&#8195;在执行一个操作的时候，因为触发了一个内部操作，然而在执行这个内部操作的时候失败了
所以根据客户实际的报错，根据`action type`和具体的`code`去判断引起的原因，报错示例：
```
$ db2 rebind package <package name>
SQL0727N An error occurred during implicit system action type "3"
Information returned for the error includes SQLCODE "-117",SQLSTATE "42802"
and massage tokens "". LINE NUMBER=664. SQLSTATE=56098
```
在报错示例中可以看到`action type`为3：implicit revalidation of an object，`SQLCODE`为"-117",`SQLSTATE`为"42802"，初步判断是SQL0117N引起的，SQL0117N详细描述单独说明。

### SQL0117N
官方说明：[DB2 SQL0117N](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/com.ibm.db2.luw.messages.sql.doc-gentopic1.html#sql0117n)

描述：赋值数目与指定的或隐含的列数或变量数不一样，将无法处理该语句。

在下列情况下，值的数目可能会不同：
- 在`INSERT`语句值列表中的插入值的数目与指定的或隐含的列数不相同。如果未指定任何列列表，那么会隐含包括表（隐式隐藏的表除外）或视图中所有列的列列表。
- 在`SET`语句或`UPDATE`语句的`SET`子句中，赋值符号右侧的值数目与左侧的列数或变量数不匹配。

解决方法：
更正该语句以便为每一个指定的或隐含的列或变量指定一个值。

sqlcode：-117
sqlstate：42802

### SQL1224N
官方说明：[Db2 11.1 SQL1000 - SQL1249](https://www.ibm.com/docs/en/db2/10.5?topic=SSEPGG_10.5.0/com.ibm.db2.luw.messages.sql.doc/sql1000-sql1249.html)

描述：由于数据库管理器发生了错误或者被强制中断，从而无法接受新的请求，已终止正在处理的所有请求或者已终止所指定的请求。

可能原因：
- 客户机/服务器配置问题
- 数据库管理器代理程序不可用
- 此用户标识没有足够的权限
- 数据库目录冲突
- 达到已配置的数据库限制或系统资源限制
- 所请求的功能不受支持
- 在新添加的成员上激活数据库

用户响应：
- 仅限于联合环境：确定是联合数据源返回了错误还是联合数据库服务器返回了错误
- 确保客户机/服务器配置正确
- 确保数据库管理器已启动并处于运行状态
- 确保此用户标识有权执行下列操作：
- 消除任何数据库目录冲突

&#8195;&#8195;在DB2 pureScale环境中，如果已添加新成员，那么必须先等待激活操作完成，然后才能从新成员中打开后续数据库连接。否则，请从另一个现有成员进行连接

sqlcode：-1224
sqlstate：55032

### 待补充
