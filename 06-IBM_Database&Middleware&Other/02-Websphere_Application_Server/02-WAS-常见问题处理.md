# WAS-常见问题处理
WAS常见问题有两类，配置相关问题和性能相关问题。    
### 内存溢出问题
##### WAS使用的内存类型
Jave堆内存(Java heap)：
- 存放Java对象的内存空间
- 通过-Xms(初始堆大小)和-Xmx(最大堆大小)设置，并在运行过程中由JVM动态调整

本地内存(native memory):
- JavaD对象职位使用的一些内存
- 不能手动设置，等于进程可以总内存减去Java最大堆内存
- 64位的WAS环境寻址空间非常大，本地内存理论上也可以设置很大
- 32位的WAS：
    - AIX:2.75G-Xmx(Xmx<2304M)
    - Linux:3G-Xmx
    - Windows:2G-Xmx

##### Java堆内存溢出分类
大对象分配：
- 大于64KB即为大对象
- 可添加JVM参数找出大对象：`-Xdump:stack:events=allocation,filter=#5m`

堆内存耗尽：
- 内存泄漏
- 内存使用量短时间内达到最大值（如很大的数据库查询结果集）

堆内存碎片化（主要是V6及以前版本，几乎没有在使用的）：
- pinned objedts不可移动的对象

##### 查看堆内存使用量
查看方法：      
登录到WAS管理控制台-->监控和调整-->性能查看器-->当前活动-->(服务器名称)-->性能模块      
看到的内存使用量的曲线图一半两种状态：
- 锯齿状，说明比较稳定，是正常情况
- 持续增长或指数增长型，当增长到最大堆后，将无法分配新的内存，就会出现内存溢出。

##### 需要收集的数据
收集数据方法参考上一篇文章：[WAS-数据收集](https://bond-huang.github.io/huang/06-IBM_Database&Middleware&Other/02-Websphere_Application_Server/01-WAS-%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

##### 本地内存溢出
常见原因及建议：
- 最大堆设置过大：减小最大堆
- java.lang.TreadLocal泄露本地内存：设置WebContainer线程池最大值等于最小值
- AIO:DirectByteBurrer泄露本地内存：设置WebContainer定制属性：`com.ibm.ws.webcontainer.channelwritety pe=sync`
- JIT(Just-In-Time)编辑器内存泄露：禁用JIT
- Classloader及其它JNI调用内存泄露：分析系统core文件

现象：
- 通常不会生成heapdump，生成系统core，然后导致crash
- 64位环境可能表现为WAS进程的总内存不断增大

官方介绍链接：
- [Native Memory Issues on Linux](https://www.ibm.com/support/pages/node/331133)
- [Native Memory Issues on AIX](https://www.ibm.com/support/pages/node/127703)

### WAS响应慢&线程挂起&CPU高
##### WAS响应慢&线程挂起
一般情况下WAS响应慢原因：
- HTTP服务器和插件导致
- Java堆内存配置不合理
- 数据连接池问题
- WebContainer线程池
- 网络质量问题
- 数据库性能问题

线程挂起是JVM不再响应客户端请求的一种状态，现象可表现为：
- WAS访问端口(9080)连接数高
- 应用程序首页无法打开
- SystemOut.log日志中出现WSVR0605W

可能导致线程挂起的原因：
- 此案从死锁：A线程等待B线程正在使用的某个资源，同时B线程也在等待A线程正在使用的某个资源
- 应用程序代码问题：部分代码效率不高，在业务压力大的时候称为性能瓶颈
- 线程池和&或数据源配置不合理
- 数据库和&或其它后天系统存在性能问题
- 垃圾回收效率低下，GC开销过大
- 系统物理资源瓶颈：物理内存不足，换页空间使用率高，I/O过高

##### 高CPU问题
造成WAS高CPU的主要原因和分析思路：
- 垃圾回收消耗大量CPU资源：
    - 通过tprof/top定位问题
    - 分析垃圾回收日志找出消耗资源的原因
    - 通过-Xgcpolicy设置合适的垃圾回收策略
    - 调整其它垃圾回收相关的JVM参数
- 应用程序出现死循环，复杂递归调用或消耗资源的操作：
    - 通过tprof/top定位问题线程
    - 对应到javacore里找到问题线程的Java堆栈信息，将此堆栈信息提供给程序开发任意，继续排查程序问题
- 配置不恰当：
    - 单个线程消耗的CPU都不高
    - 垃圾回收基本正常
    - 线程数量很多
- 系统CPU资源不够：
    - 物理CPU个数太少
    - CPU出现排队等待现象(vmstat)

##### 需要收集的数据
收集数据方法参考上一篇文章：[WAS-数据收集](https://bond-huang.github.io/huang/06-IBM_Database&Middleware&Other/02-Websphere_Application_Server/01-WAS-%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

官方介绍链接：
- [Performance,hang,orhigh CPU issues with WAS on AIX](https://www.ibm.com/support/pages/node/660891)
- [Performance,hang,orhigh CPU issues with WAS on Linux](https://www.ibm.com/support/pages/node/72419?mhsrc=ibmsearch_a&mhq=%20hang%2C%20or%20high%20CPU%20issues%20with%20WebSphere%20Application%20Server%20on%20linux)
- [Performance,hang,or high CPU issues on Windows](https://www.ibm.com/support/pages/node/71631)

### WAS carsh(宕机)
应用程序服务器因为软件方便的原因进程意外终止的一种故障。carsh和线程挂起的区别：carsh进程不在，线程挂起进程还在。

WAS carsh常见原因：
- Segmentation Violation：应用程序访问了错误的内存地址
- Native Stack Overflow：栈指针超出线程栈的限制
- 本地内存溢出：OutOfMemortError,无法使用malloc方法分配到内存
- 垃圾回收异常：
    - 垃圾回收的过程中出现异常
    - 垃圾回收后出现异常，说明可能存储内存故障
- JIT(Just-In-Time)异常：
    - 编译过程中出现异常
    - 编译输出的本地代码异常，导致WAS执行时出错
- JNI(Java Native Interface)调用异常
    - 程序中调用到了本地库文件，或程序中的第三方代码使用了本地库文件，如JDBC驱动，MQ库文件，CM库文件

##### 需要收集的数据
收集数据方法参考上一篇文章：[WAS-数据收集](https://bond-huang.github.io/huang/06-IBM_Database&Middleware&Other/02-Websphere_Application_Server/01-WAS-%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)
