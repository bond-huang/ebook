# WAS-日志收集
WAS作为中间件，如果出现问题，很可能涉及到业务，故障原因除WAS本身，还可能涉及到操作系统和应用，根据WAS不同的故障现象，收集相应的日志给IBM WAS工程师可以尽快并且准确分析出问题原因，有助于尽快解决业务问题。
主要分为以下几种情况：
- 线程挂起
- Java堆内存溢出
- hang or high CPU问题
- crash宕机

### 线程挂起_收集数据
当WAS线程挂起的时候，需要收集数据有javacore、垃圾回收日志、目录<profile_root>/logs下所有日志以及网络端口信息。
##### javacore
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
如果没有自定义过，生成日志默认存放在<profile_root>/javacore*。
##### 垃圾回收日志
即native_stderr.log，需要手动启动，收集用途：
- 垃圾回收效率
- 有没有OutOfMemoryError及其它JVM异常

##### 目录<profile_root>/logs下所有日志
收集用途：
- 运行时日志：SystemOut.log,SystemErr.log
- ffdc日志

##### 端口信息
收集用途：
- 查看端口使用情况

收集运行命令netstat -an >netstat.txt

### Java堆内存溢出_收集数据
当出现Java堆内存溢出的时候，需要收集数据有heapdump、垃圾回收日志、javacore文件、目录<profile_root>/logs下的其它日志和server.xml文件
##### 堆内存转储heapdump文件
收集用途：
- 分析堆内存的具体使用情况
- 默认生成在<profile_root>下

##### 垃圾回收日志
即native_stderr.log，需要手动启动，收集用途：
- 分析出现内存溢出的过程
- 确认触发内存溢出的直接原因
- 评估垃圾回收性能，找出合适的GC策略和调优参数

##### Java线程转储javacore文件
在Java内存溢出问题中自动生成，收集用途：
- Java线程信息、环境变量及Java变量设置，类加载信息
- 查看java.lang.OutOfMemoryError

##### 其它日志
目录<profile_root>/logs下的其它日志和server.xml文件

#### hang or high CPU问题数据收集
如果在使用WAS的过程中遇到性能，挂起或高CPU问题,
可能原因有：
- 操作系统CPU资源不足
- 垃圾回收消耗大量CPU资源
- 应用程序出现死循环/复杂递归调用
- 配置不恰当

可以通过下面方法收集诊断和解决问题所需的数据。
##### AIX系统收集方法
IBM提供了一个脚本去收集，脚本名称：aixperf.sh
需要用root用户执行，并且确保有执行权限，脚本收集信息如下：
- aixperf_RESULTS.tar.gz
- 三个javacore
- 目录<profile_root>/logs下所有

官方链接：
[AIX系统中WAS性能问题](https://www.ibm.com/support/pages/node/660891)
##### Linux系统收集方法
IBM同样提供了一个脚本去收集，脚本名称：linperf.sh
需要用root用户执行，并且确保有执行权限，脚本收集信息如下：
- linperf_RESULTS.tar.gz
- 三个javacore
- 目录<profile_root>/logs下所有

官方链接：
[Linux系统中WAS性能问题](https://www.ibm.com/support/pages/node/72419?mhsrc=ibmsearch_a&mhq=%20hang%2C%20or%20high%20CPU%20issues%20with%20WebSphere%20Application%20Server%20on%20linux)
##### Windows系统收集方法
官方链接：
[Windows系统中WAS性能问题](https://www.ibm.com/support/pages/node/71631)
### crash宕机_收集数据
crash是应用程序服务器因为软件的原因进程意外终止；
crash和线程挂起的区别是crash时候进程不在了，线程挂起进程还在
一般出现此故障原因：
- 垃圾回收异常
- JIT(Just-In-Time)异常
- 应用程序访问了错误的内存地址
- 本地内存溢出
- 栈指针超出了线程栈的限制
- JNI(Java Native Interface)调用异常

##### 主要需要收集
- javacore，通常自动生成
- 目录<profile_root>/logs下所有日志
  - 运行时日志：SystemOut.log,;SystemErr.log
  - JVM日志：native_stderr.log;native_stdout.log
  - ffdc日志
- 系统core文件：<profile_root>目录下；系统/tmp目录下；<user_home>目录下

##### 操作系统数据收集
AIX 系统需要收集信息如下：
- errpt信息：记录系统事件和系统报错
- dbx输出：提供native堆栈信息

Linux 系统需要收集信息如下：
- gdb输出
- libsgrabber
