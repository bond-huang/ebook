# Cognos-常见问题
### 无法登录到数据源
报表无法查看,检查发现Cognos有报错，报错信息和现场如下：
- Failure QE-DEF-0285 登录失败。XQE-DS-0006 无法登录到数据源。ORA-28000: the account is locked 	
- Failure QE-DEF-0285 登录失败。XQE-DS-0006 无法登录到数据源。ORA-01005: null password given; logon denied
- 尝试重启Cognos后，Cognos可以正常连接到oracle数据库，报表可以正常查看，但是不定时会出现此报错，再次无法连接oracle数据库，

##### 初步分析过程
初步操作建议：
- 建议在发生故障时候不重启Cognos，可以单独重启Cognos中的Query Service。如果确认有效，可以在故障发生时候快速恢复，不需要重启整个Cognos耗费比较长时间。重启Query Service步骤：
    - 依次选择：IBM Cognos Administration-->系统-->服务-->查询
    - 选中Query Service右边下三角，然后选中stop immediately先关掉，然后在选中start进行启动
 
- 此台Cognos服务采用JDBC (Java DataBase Connectivity,java数据库连接)方式连接到oracle数据库，建议进行JDBC trace，可以追踪到一下Cognos发送到oracle端的具体内容信息，JDBC trace更改操作见[Cognos-数据收集](https://bond-huang.github.io/huang/06-IBM_Database&Middleware&Other/05-Cognos/01-Cognos-%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html) 

- 收集Cognos的XQE dump日志，需要更改日志收集策略，方法见[Cognos-数据收集](https://bond-huang.github.io/huang/06-IBM_Database&Middleware&Other/05-Cognos/01-Cognos-%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html) 

##### 收取日志
&#8195;&#8195;在根据上面步骤修改好配置后，当再次发生报表系统使用不了，检查依然是Cognos连接oracle出现报错，故障类型一样，根据IBM Cognos工程师建议，进行如下操作：
- 重启Query Service，重启后尝试进行连接oracle，测试正常，报表系统正常使用；
通过此操作可以在未准确定位故障前快速进行临时恢复，整个过程不超过1分钟；
- 在Cognos的...log/xqe目录下取出故障点的JDBC trace日志和XQE dump日志；
- 将Query Service重启后的结果反馈给IBM Cognos工程师，同时将抓取到的日志上传到IBM专用日志分析服务器给IBM Cognos工程师进行分析。

##### 故障分析和解决
&#8195;&#8195;Cognos采用JDBC (Java DataBase Connectivity,java数据库连接)方式进行连接oracle数据库，连接属性配置等都是在JDBC里面配置，根据分析，JDBC驱动问题也会导致此类型报错，建议ojdbc5.jar 换成 ojdbc6.jar（或者ojdbc7.jar），并且对应相应oracle数据库版本。          
&#8195;&#8195;在示例故障中JDBC版本不匹配导致（数据库版本是11.2.0.4.0，JDBC的驱动版本是11.2.0.3.0），可能是近期用户升级了oracle数据库：                        
- 建议在闲暇时段停止Cognos服务，在Cognos连接的oracle（版本11.2.0.4.0）数据库的程序文件中取出了JDBC驱动，替换了Cognos中的JDBC驱动，此目的是为了保证一致性；同时从oracle官网下载11.2.0.4.0版本的JDBC驱动备用，下载地址[oracle JDBC](https://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html)          
- 更新驱动后Cognos能正常连接到数据库，但是由于之前故障是间歇性的，故需要进行持续观察，长时间观察后，并没有再次发生故障，报错ORA-01005: null password given; logon denied的问题得到解决。

数据库用户锁死分析和临时解决方法:                  
&#8195;&#8195;报错日志中ORA-28000: the account is locked报错，此报错是因为数据库密码锁死导致。Oracle工程师根据分析是由于Cognos频繁发送无效密码导致。Cognos工程师分析是由于Cognos报错ORA-01005: null password given; logon denied导致；      
&#8195;&#8195;在操作系统层面Oracle数据库用户密码有重试次数限制，由于Oracle影响其它应用访问此数据库，故可以将此数据库的密码策略临时进行更改（不设重试错误次数被锁限制），避免Cognos连接不上oracle时候持续发生无效密码导致数据被锁。在后续观察中，Cognos出现ORA-01005: null password given; logon denied报错后，数据库用户没有被锁死，临时避免了Cognos连接数据库故障时候锁死用户。    

问题解决后建议：           
- 在前面内容中提到的抓取JDBC trace日志和XQE dump日志进行的修改进行还原，因为日志持续增长，服务器存储空间有限；
- 在JDBC连接问题观察一段时间后不再出现，说明问题解决了，那么可以将oracle用户的密码策略恢复到初始状态;
- 如果数据库进行了更新，建议同时更新Cognos端的JDBC驱动；
- 如还有其它平台Cognos在使用，建议检查JDBC版本，尽量和数据库版本保持一致。

### 其它问题
##### 内容库报错
日志中有内容库的报错时候，建议做一个consistency check，检查内部即可。检查方法连接如下：[Creating and running a content store consistency check](https://www.ibm.com/support/knowledgecenter/en/SSEP7J_11.0.0/com.ibm.swg.ba.cognos.ug_cra.doc/t_asg_mt_consistencycheck.html)

##### CAM报错
在日志中有中有CAM的报错，建议重新生成crypto keys，官方参考链接如下：[How to Regenerate Cryptographic Keys as of Cognos](http://www-01.ibm.com/support/docview.wss?uid=swg21694322)

##### 缓存清理
清除query service缓存，清除方法如下：
- 在IBM Cognos Administration管理界面找到配置选项并点开，找到Query Service Caching项；
- 选中Query Service Caching项，在右下角点击Clear cache选项，清除状态会显示出来。

官方参考链接：[Clear everything in the cache](https://www.ibm.com/support/knowledgecenter/en/SSEP7J_11.0.0/com.ibm.swg.ba.cognos.ug_cra.doc/t_asg_clear_qs_cache.html#ASG_clear_qs_cache)
