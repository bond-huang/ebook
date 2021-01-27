# WAS-日志收集
&#8195;&#8195;WAS作为中间件，如果出现问题，很可能涉及到业务，故障原因除WAS本身，还可能涉及到操作系统和应用，根据WAS不同的故障现象，收集相应的日志给IBM WAS工程师可以尽快并且准确分析出问题原因，有助于尽快解决业务问题。
主要分为以下几种情况：
- 线程挂起
- Java堆内存溢出
- hang or high CPU问题
- crash宕机

## 线程挂起_收集数据
&#8195;&#8195;当WAS线程挂起的时候，需要收集数据有javacore、垃圾回收日志、目录&#60;profile_root&#62;/logs下所有日志以及网络端口信息。
### javacore
一定要在故障发生时收集
收集用途：
- 查看线程信息
- 监控锁信息

AIX & Linux平台收集方法如下：
生成三个javacore，每两个间隔两分钟执行一次：
```shell
#查看WAS java 进程id
ps -ef |grep java
#手动生成三个javacore
kill -3 <pid>
```
如果没有自定义过，生成日志默认存放在&#60;profile_root&#62;/javacore*。
### 垃圾回收日志
即native_stderr.log，需要手动启动，收集用途：
- 垃圾回收效率
- 有没有OutOfMemoryError及其它JVM异常

### 目录<profile_root>/logs下所有日志
收集用途：
- 运行时日志：SystemOut.log,SystemErr.log
- ffdc日志

### 端口信息
收集用途：
- 查看端口使用情况

收集运行命令netstat -an >netstat.txt

## Java堆内存溢出_收集数据
&#8195;&#8195;当出现Java堆内存溢出的时候，需要收集数据有heapdump、垃圾回收日志、javacore文件、目录&#60;profile_root&#62;/logs下的其它日志和server.xml文件
### 堆内存转储heapdump文件
收集用途：
- 分析堆内存的具体使用情况
- 默认生成在&#60;profile_root&#62;下

### 垃圾回收日志
即native_stderr.log，需要手动启动，收集用途：
- 分析出现内存溢出的过程
- 确认触发内存溢出的直接原因
- 评估垃圾回收性能，找出合适的GC策略和调优参数

### Java线程转储javacore文件
在Java内存溢出问题中自动生成，收集用途：
- Java线程信息、环境变量及Java变量设置，类加载信息
- 查看java.lang.OutOfMemoryError

### 其它日志
目录&#60;profile_root&#62;/logs下的其它日志和server.xml文件

## hang or high CPU问题数据收集
如果在使用WAS的过程中遇到性能，挂起或高CPU问题,可能原因有：
- 操作系统CPU资源不足
- 垃圾回收消耗大量CPU资源
- 应用程序出现死循环/复杂递归调用
- 配置不恰当

可以通过下面方法收集诊断和解决问题所需的数据。
### AIX系统收集方法
IBM提供了一个脚本去收集，脚本名称：aixperf.sh
需要用root用户执行，并且确保有执行权限，脚本收集信息如下：
- aixperf_RESULTS.tar.gz
- 三个javacore
- 目录&#60;profile_root&#62;/logs下所有

官方链接：
[AIX系统中WAS性能问题](https://www.ibm.com/support/pages/node/660891)
### Linux系统收集方法
IBM同样提供了一个脚本去收集，脚本名称：linperf.sh
需要用root用户执行，并且确保有执行权限，脚本收集信息如下：
- linperf_RESULTS.tar.gz
- 三个javacore
- 目录&#60;profile_root&#62;/logs下所有

官方链接：
[Linux系统中WAS性能问题](https://www.ibm.com/support/pages/node/72419?mhsrc=ibmsearch_a&mhq=%20hang%2C%20or%20high%20CPU%20issues%20with%20WebSphere%20Application%20Server%20on%20linux)
### Windows系统收集方法
官方链接：
[Windows系统中WAS性能问题](https://www.ibm.com/support/pages/node/71631)
## crash宕机_收集数据
&#8195;&#8195;crash是应用程序服务器因为软件的原因进程意外终止；crash和线程挂起的区别是crash时候进程不在了，线程挂起进程还在，一般出现此故障原因：
- 垃圾回收异常
- JIT(Just-In-Time)异常
- 应用程序访问了错误的内存地址
- 本地内存溢出
- 栈指针超出了线程栈的限制
- JNI(Java Native Interface)调用异常

### 主要需要收集
- javacore，通常自动生成
- 目录&#60;profile_root&#62;/logs下所有日志
  - 运行时日志：SystemOut.log,;SystemErr.log
  - JVM日志：native_stderr.log;native_stdout.log
  - ffdc日志
- 系统core文件：&#60;profile_root&#62;目录下；系统/tmp目录下；&#60;user_home&#62;目录下

### 操作系统数据收集
AIX 系统需要收集信息如下：
- errpt信息：记录系统事件和系统报错
- dbx输出：提供native堆栈信息

Linux 系统需要收集信息如下：
- gdb输出
- libsgrabber

## Installation Manager数据收集
### Installation Manager图形（GUI）界面收集数据
步骤如下：
- 启动Installation Manager
- 在菜单栏中，转到“Help”，然后选择“Export data for problem analysis”
- 在“ Export Data”窗口中，选择名称和位置以保存包含问题分析数据的.zip文件
- 单击“OK”，然后退出安装管理器

&#8195;&#8195;如果是第一次在Linux上安装Installation Manager，或者在Linux上升级Installation Manager的现有版本，请运行以下命令。此命令旨在单行显示：
```
rpm -qa --qf  "%-20{NAME} %-10{VERSION} %-10{RELEASE} %-10{ARCH}\n" >/tmp/rpmlist.txt
```
&#8195;&#8195;将以下文件组收集到单个zip或tar文件中，然后将它们发送给IBM进行进一步分析:
- 从Installation Manager生成的.zip文件
- WAS_HOME/logs中的所有文件（如果存在“ logs”目录）
- 静默响应文件（如果使用响应文件安装产品）
- 如果适用于您的方案，则在先前步骤中生成的/tmp/rpmlist.txt文件
- （可选）收集下面“可选其它数据”部分中列出的信息

### Installation Manager命令行（imcl）界面收集数据
步骤如下：
- 浏览至安装Installation Manager的位置中的eclipse/tools子目录
- 运行imcl命令将数据导出到选择的文件中（文件名使用“ .zip”扩展名）：
  - Windows: imcl.exe exportInstallData &#60;outputFileName&#62;
  - UNIX and Linux: ./imcl exportInstallData &#60;outputFileName&#62;
  - z/OS: ./imcl exportInstallData &#60;outputFileName&#62;

剩下步骤同上，需要传送给IBM的数据信息也是同上。
### 在z/OS上收集Installation Manager的数据
使用imcl实用程序为z/OS上的Installation Manager收集数据，见上面描述。
### 可选其它数据
&#8195;&#8195;安装程序将写入一些其它数据，这些数据可能有助于验证安装是否完成，收集方法和步骤如下：
收集以下两个文件：
- WAS_HOME/properties/version/installed.xml
- WAS_HOME/properties/version/history.xml
收集.was.installlocation.registry文件，位置会有所不同，取决于运行的操作系统的用户：
- 管理员或root用户
  - AIX：/usr/.ibm
  - HP-UX, Linux, Solaris：/opt/.ibm
  - Windows：Windows directory
- 非管理员或非root用户
  - AIX, HP-UX, Linux, Solaris：&#60;user_home&#62;/.ibm
  - Windows：&#60;user_home&#62;\\.ibm

### 当Installation Manager无法自动收集数据时手动收集
&#8195;&#8195;如果Installation Manager发生故障并且无法生成用于问题分析的数据，那么可以手动收集问题分析所需的文件。找到Installation Manager的Agent Data directory，然后压缩该目录的全部内容,建议使用zip格式。    
&#8195;&#8195;注意，Agent Data directory与安装Installation Manager的位置不同。具体取决于您是以管理员身份（根用户），非管理员用户身份（非root用户）还是以组模式安装。如果不知道具体目录，可以参考官方文档：[default Agent Data location for Installation Manager](https://www.ibm.com/support/knowledgecenter/SSDV2W_1.8.5/com.ibm.silentinstall12.doc/topics/r_app_data_loc.html)

### 官方参考
官方参考链接：[MustGather:Installation Manager issues for installing and updating WebSphere Application Server V8.0, V8.5, and V9.0](https://www.ibm.com/support/pages/node/157799#)

## 类加载问题
收集步骤：  
1) 使用以下跟踪字符串启用和收集Application Server traces：
```
com.ibm.ws.classloader.*=all
```
Static trace:
- 登录到管理控制台
- 在左侧面板中，展开`Troubleshooting`,单击`Logs and Trace`
- 选择要跟踪的应用程序服务器，然后在下一页上单击`Diagnostic Trace`链接
- 选择`Configuration `选项卡。
- 在`Trace Output`下，选择`File`单选按钮并指定`File Name`。另外，将`Maximum file size`增加到100 MB，并将`Maximum number of historical files`增加到10
- 选择`Basic (Compatible) Trace Output Format`,除非IBM支持人员另有说明
- 在同一面板上，单击右侧面板上`Additional Properties`下的`Change Log Detail Levels`
- 在`Configuration tab`选项卡下，通过输入特定于您要为其收集数据的MustGather文档的跟踪字符串来指定`Trace Specification`
- 单击` Apply `和`OK`,然后`Save`配置（选择`Synchronize changes with Nodes`选项）
- 重新启动服务器以开始tracing

Dynamic trace:
- 登录到管理控制台
- 在左侧面板中，展开`Troubleshooting`,单击`Logs and Trace`
- 选择要跟踪的应用程序服务器，然后在下一页上单击`Diagnostic Trace`链接
- 选择`Runtime`选项卡（服务器应已启动并正在运行，此选项卡才能显示）
- 在`Trace Output`下，选择`File`单选按钮并指定`File Name`。另外，将`Maximum file size`增加到100 MB，并将`Maximum number of historical files`增加到10
  - 重要提示：如果不希望此设置成为永久设置，不要选择`Save Runtime Changes to Configuration as well`
- 在同一面板上，单击右侧面板上`Additional Properties`下的`Change Log Detail Levels`
- 选择`Runtime`选项卡
- 在`Trace Specification`下，输入要为其收集数据的特定MustGather的跟踪字符串
- 单击` Apply `和`OK`,然后`Save`配置（选择`Synchronize changes with Nodes`选项）
- 服务器不需要重新启动

Stopping trace:
- 在适当的跟踪选项卡（Configuration and/or Runtime）中，从`Trace Specification`中删除跟踪字符串
- 单击` Apply `和`OK`,然后`Save`配置（选择`Synchronize changes with Nodes`选项）

2) 通过管理控制台启用Java™虚拟机（JVM）类加载器跟踪:
- 选择`Servers`，选择` Application servers`，然后选择要配置的服务器
- 在`Server Infrastructure`部分中，打开`ava and Process Management`，然后选择`Process Definition`
- 在`Additional Properties`下，选择`Java Virtual Machine`
- 选中`Verbose class loading`复选框。
- 将以下字符串添加到`Generic JVM arguments field`字段中：`Dws.ext.debug = true  -Dws.osgi.debug`
- 单击`OK`

3) 保存更改,停止服务器;   
4) 清除JVM和OSGi类缓存,有关更多详细信息，请参阅如何清除WebSphere类高速缓存;    
5) 备份和删除此WebSphere进程的所有WebSphere日志文件（SystemOut * .log，SystemErr * .log，native_stderr.log，native_stdout.log，startServer.log，stopServer.log和trace.log）。

日志文件位于以下目录中：
```
<profile_root> / logs / server_name / * 
```
清除所有WebSphere Application Server FFDC日志。FFDC文件位于以下目录中：
```
<profile_root> / profile_name / logs / ffdc / *
```
注意：如果已配置为将FFDC日志文件写入其他位置，请相应地清除它们。

6 )重新启动服务器,重现该问题。确保在WebSphere日志（SystemOut.log）和跟踪文件（trace.log）中生成了类加载器异常

7 )针对问题概要文件运行收集器工具:
- [Gathering information with the collector tool - V8.5](https://www.ibm.com/support/knowledgecenter/SSEQTP_8.5.5/com.ibm.websphere.base.doc/ae/ttrb_runct.html)
- [Gathering information with the collector tool - V9.0](https://www.ibm.com/support/knowledgecenter/SSEQTP_9.0.5/com.ibm.websphere.base.doc/ae/ttrb_runct.html)

### 参考链接
官方参考链接：
- [Classloader problems for WebSphere Application Server](https://www.ibm.com/support/pages/node/337371)
- [Setting up a trace in WebSphere Application Server](https://www.ibm.com/support/pages/node/89123)
- [How to clear the WebSphere class caches](www-01.ibm.com/support/docview.wss?uid=swg21607887)
