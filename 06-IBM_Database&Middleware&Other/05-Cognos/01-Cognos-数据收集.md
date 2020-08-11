# Cognos-数据收集
Cognos接触不多，目前也就收集过两种日志。
### JDBC trace
&#8195;&#8195;JDBC(Java DataBase Connectivity)连接数据库出现问题时候，可以抓JDBC trace,可以追踪到一下Cognos发送到oracle端的具体内容信息，JDBC trace更改操作如下:                     
a. 找到并打开cognos/CA11/configuration/xqe.diagnosticlogging.xml文件:
```
<component name="XQE">
<eventGroup name="QueryService" appender="IPFAudit"/>
<eventGroup name="Disp"/>
<eventGroup name="Resources"/>
<eventGroup name="Console" appender="console"/>
<eventGroup name="InitTerm"/>
<eventGroup name="QueryEngine" level="trace"/>
<eventGroup name="RequestAdapter"/>
<eventGroup name="RSAPI" level="warn"/>
<eventGroup name="MetadataService"/>
<eventGroup name="MetadataConnection"/>
<eventGroup name="ProviderManager"/>
<eventGroup name="CacheManager"/>
<eventGroup name="QueryReuseManager"/>
<eventGroup name="Connection"/>
<eventGroup name="Connection.Factory"/>
<eventGroup name="Connection.Pool"/>
<eventGroup name="JDBC"/>
<eventGroup name="JDBCAPITrace"/>
```
b. 将文件中的`<eventGroup name="JDBC"/>`更改为`<eventGroup name="JDBC" level="trace"/>`;               
c. 如果需要了解驱动程序正在进行哪些方法调用（收集此信息根据IBM工程师建议）：将`<eventGroup name="JDBCAPITrace"/>`修改为`<eventGroup name="JDBCAPITrace" level="trace"/>`；        
d. 修改完成后示例：
```
<component name="XQE">
<eventGroup name="QueryService" appender="IPFAudit"/>
<eventGroup name="Disp"/>
<eventGroup name="Resources"/>
<eventGroup name="Console" appender="console"/>
<eventGroup name="InitTerm"/>
<eventGroup name="QueryEngine" level="trace"/>
<eventGroup name="RequestAdapter"/>
<eventGroup name="RSAPI" level="warn"/>
<eventGroup name="MetadataService"/>
<eventGroup name="MetadataConnection"/>
<eventGroup name="ProviderManager"/>
<eventGroup name="CacheManager"/>
<eventGroup name="QueryReuseManager"/>
<eventGroup name="Connection"/>
<eventGroup name="Connection.Factory"/>
<eventGroup name="Connection.Pool"/>
<eventGroup name="JDBC" level="trace"/>
<eventGroup name="JDBCAPITrace" level="trace"/>
```
e. 将更改保存到文件，不需要重新启动（这是Cognos Analytics 11版本说明，有条件的话，还是建议重启Query Service或者整个Cognos去激活这个配置），在进行更改后等待30秒钟，重新问题即可;          
f. 在Cognos安装目录下的...log/xqe目录中，在故障发生后，将故障点的日志取出。在收集所需的日志文件后，通过还原原始xqe.diagnosticlogging.xml文件来禁用跟踪。           

### XQE dump日志
收集Cognos的XQE dump日志，，需要更改日志收集策略，方法如下：        
a. 找到并打开cognos/CA11/configuration/xqe.config.xml文件；             
b. 找到如下字段：
```
<vendor name="IBM Corporation"
   options="-Xscmx100m -Xshareclasses:cachedir=../dataˆname=cognos10%uˆnonfatal -Xmso512K"
   compressedrefs="-Xcompressedrefs" max="27000"
  />
```
c. 在b中的字段中加入如下字段：    
```
Xdump:system:events=systhrow+throwˆfilter=com/ibm/cognos/jdbc/adaptor/sqlexception/SQLCognosInvalidLogonExceptionˆrange=1..1ˆrequest=serial+compact+prepwalk) into the xqe.config.xml
```
d. 最终变成如下字段：            
```
<vendor name="IBM Corporation"
   options="-Xscmx100m -Xshareclasses:cachedir=../dataˆname=cognos10%uˆnonfatal -Xmso512K -Xdump:system:events=systhrow+throwˆfilter=com/ibm/cognos/jdbc/adaptor/sqlexception/SQLCognosInvalidLogonExceptionˆrange=1..1ˆrequest=serial+compact+prepwalk "
   compressedrefs="-Xcompressedrefs" max="27000"
  />
```
e. 修改目的是专门抓去invalid logon信息的，一旦无法登录到数据源错误发生，立马会在...log/xqe文件夹下产生dump，故障发生后取出即可。  
