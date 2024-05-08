# AIX-系统管理
## /etc/inittab文件
作用：控制初始化过程，init命令去调用`/etc/inittab`文件。
### 条目格式
在`/etc/inittab`文件中，条目格式如下:
```
Identifier:RunLevel:Action:Command
```
冒号既是分隔符也是注释符号，如果要注释掉inittab条目，在条目开头加上冒号：
```
:Identifier:RunLevel:Action:Command
```
字段简介：
- Identifier：唯一表示对象的字符串
- RunLevel：可以处理该条目的运行级别，运行级别有效地对应于系统中进程的配置，由数字0到9表示
- Aciton：告诉init命令如何处理在标识符字段中指定的进程
- Command：要执行的shell命令

Action简介：

Aciton|描述
:---:|:---
respawn|如果进程不存在，启动进程；进程终止后重新启动。如果进程存在，则不执行任何操作，继续扫描/etc/inittab文件
wait|当init命令输入与该条目的运行级别匹配时，启动该进程并等待其终止
once|当init命令输入与该条目的运行级别匹配时，启动该进程，而不等待其终止；当进程终止后，不重启进程
boot|仅在系统启动期间（即init命令在系统启动期间读取/etc/inittab文件时）处理该条目
bootwait|系统启动后，在init命令第一次从单用户状态变为多用户状态时，处理该条目
powerfail|仅当init命令收到电源故障信号（SIGPWR）时，才执行与此条目相关的过程
powerwait|仅当init命令收到电源故障信号（SIGPWR）时，才执行与该条目关联的过程，并等待其终止，然后继续处理/etc/inittab文件
off|如果与此条目关联的进程当前正在运行，发送警告信号（SIGTERM），并等待20秒钟，然后使用kill信号（SIGKILL）终止该进程。如果该进程未在运行，忽略此条目。
ondemand|在功能上与respawn相同，除了此操作适用于a，b或c值，不适用于运行级别
initdefault|仅在最初调用init命令时才扫描具有此操作的条目
sysinit|在init命令尝试访问控制台之前（登录之前），将执行这种类型的条目

### 条目操作
将记录添加到`/etc/inittab`文件中格式及示例：
```
mkitab Identifier:Run Level:Action:Command
mkitab tty002:2:respawn:/usr/sbin/getty /dev/tty2
```
更改`/etc/inittab`文件中记录的格式及示例（改成运行级别在2和3上运行）：
```
chitab Identifier:Run Level:Action:Command
chitab tty002:23:respawn:/usr/sbin/getty /dev/tty2
```
列出`/etc/inittab`文件中记录：
```
lsitab -a
lsltab <Identifier>
```
删除`/etc/inittab`文件中记录：
```
rmitab <Identifier>
```

IBM官方介绍：[https://www.ibm.com/support/knowledgecenter/ssw_aix_72/filesreference/inittab.html](https://www.ibm.com/support/knowledgecenter/ssw_aix_72/filesreference/inittab.html)

PowerHA中对`/etc/inittab`文件的操作: [https://www.ibm.com/support/knowledgecenter/SSPHQG_7.2/admin/ha_admin_etcinittab.html](https://www.ibm.com/support/knowledgecenter/SSPHQG_7.2/admin/ha_admin_etcinittab.html)

## /etc/security/limits文件
配置参数说明：

参数|描述
:---:|:---
fsize|soft file size in blocks
core|soft core file size in blocks
cpu|soft per process CPU time limit in seconds
data|soft data segment size in blocks
stack|soft stack segment size in blocks
rss|soft real memory usage in blocks
nofiles|soft file descriptor limit
fsize_hard|hard file size in blocks
core_hard|hard core file size in blocks
cpu_hard|hard per process CPU time limit in seconds
data_hard|hard data segment size in blocks
stack_hard|hard stack segment size in blocks
rss_hard|hard real memory usage in blocks
nofiles_hard|hard file descriptor limit

参考链接：
- [AIX 7.3 ulimit Command](https://www.ibm.com/docs/en/aix/7.3?topic=u-ulimit-command)
- [AIX: Verifying access rights and user limits](https://www.ibm.com/docs/en/spectrum-protect/8.1.9?topic=instance-aix-verifying-access-rights-user-limits)
- [Mustgather: Using stackit.sh to collect native stack trace data on AIX](https://www.ibm.com/support/pages/node/344409?mhsrc=ibmsearch_a&mhq=aix%20stack)

## /etc/services文件
&#8195;&#8195;AIX中/etc/services文件定义了系统上用于网络服务的套接字和协议。此处还定义了PowerHA SystemMirror组件使用的端口和协议。

参考链接：
- [AIX /etc/services](https://www.ibm.com/docs/en/powerha-aix/7.2?topic=SSPHQG_7.2/admin/ha_admin_etcservices.html)
- [AIX services File Format for TCP/IP](https://www.ibm.com/docs/en/aix/7.1?topic=formats-services-file-format-tcpip)

## AIX系统服务
AIX系统服务介绍：[Summary of common AIX system services](https://www.ibm.com/docs/en/aix/7.2?topic=security-summary-common-aix-system-services)

## 待补充
