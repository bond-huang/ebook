# RHEL-日志分析及管理
## 系统日志架构
### 系统日志记录
&#8195;&#8195;进程和操作系统内核为发生的事件记录日志。这些日志用于系统审核和问题的故障排除。许多系统都以文本文件的方式记录事件日志，保存在`/var/log`目录中。RHEL中内建了一个基于Syslog协议的标准日志记录统。`systemd-journald`和`rsyslog`服务处理RHEL8中的syslog消息：
- `systemd-journald`服务是操作系统事件日志架构的核心。它收集许多来源的事件消息，包括内核、引导过程早期阶段的输出、守护进程启动和运行时的标准输出及标准错误，以及syslog事件。然后，它会将它们重构为一种标准格式，并写进带有索引的结构化系统日志中。默认情况下，该日志存储在系统重启后不保留的文件系统上。
- `rsyslog`服务会从日志中读取`systemd-journald`收到的syslog消息。之后将处理syslog事件，将它们记录到日志文件中，或根据自己的配置将它们转发给其他服务：
    - `rsyslog`服务对syslog消息进行排序，并将它们写入到在系统重启后不保留的日志文件中(/var/log)
    - `rsyslog`服务会根据发送每条消息的程序类型或设备以及每条syslog消息的优先级，将日志消息排序到特定的日志文件
- 除了syslog消息文件外，`/var/log`目录中还包含系统上其他服务的日志文件

`/var/log`目录中一些有用的文件：

日志文件|存储的消息类型
:---|:---
/var/log/messages|大多数系统日志消息记录在此处。不包括与身份验证、电子邮件处理和调度作业执行相关的消息以及纯粹与调试相关的消息
/var/log/secure|与安全性和身份验证事件相关的syslog消息
/var/log/maillog|与邮件服务器相关的syslog消息
/var/log/cron|与调度作业执行相关的syslog消息
/var/log/boot.log|与系统启动相关的非syslog控制台消息

## 查看系统日志文件
### 将事件记录到系统
&#8195;&#8195;许多程序使用`syslog`协议将事件记录到系统。每一日志消息根据设备（消息的类型）和优先级（消息的严重性）分类。下表从高到低列出了八个标准`syslog`优先级：

代码|优先级|严重性
:---:|:---:|:---:
0|emerg|系统不可用
1|alert|必须立即采取措施
2|crit|临界情况
3|err|非严重错误状况
4|warning|警告情况
5|notice|正常但重要的事件
6|info|信息性事件
7|debug|调试级别消息

&#8195;&#8195;`rsyslog`服务使用日志消息的设备和优先级来确定如何进行处理。其配置规则位于`/etc/rsyslog.conf`文件和`/etc/rsyslog.d`目录中扩展名为`.conf`的任何文件。通过在`/etc/rsyslog.d`目录中安装适当的文件，软件包可以轻松地添加规则。以下示例会将发送给`authpriv`设备的任何优先级的消息记录到文件`/var/log/secure`中：
```
# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure
```
示例说明：
- 每个控制着`syslog`消息排序方式的规则都对应了其中一个配置文件中的一行
- 每行左侧表示与规则匹配的`syslog`消息的设备和严重性
- 每行右侧表示要将日志消息保存到的文件（或消息所要发送到的其他位置）
- 星号`*`是一个匹配所有值的通配符

日志消息其它说明：
- 日志消息有时会匹配`rsyslog.conf`中的多条规则。在这种情况下，一条消息会存储到多个日志文件中。为限制存储的消息，优先级字段中的关键字`none`指出不应将指定设备的消息存储在给定的文件中
- 除了将`syslog`消息记录到文件中外，也可将它们显示到所有已登录用户的终端。`rsyslog.conf`文件中有一项设置，可将优先级为`emerg`的所有`syslog`消息显示到所有已登录用户的终端。 

Rsyslog规则(查看`/etc/rsyslog.conf`文件)示例如下：
```shell
# rsyslog configuration file
#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console
# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure
# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog
# Log cron stuff
cron.*                                                  /var/log/cron
# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*
# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler
# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
```
### 日志文件轮转
&#8195;&#8195;`logrotate`工具会轮转日志文件，以防止它们在含有`/var/log`目录的文件系统中占用太多空间。轮转日志文件时，使用指示轮转日期的扩展名对其重命名。轮转原日志文件之后，会创建新日志文件，并通知对它执行写操作的服务：
- 例如，如果在2022-01-30轮转，旧的`/var/log/messages`文件可能会变成`/var/log/messages-20220130`
- 轮转若干次之后（通常在四周之后），丢弃最旧的日志文件以释放磁盘空间
- 调度作业每日运行一次`logrotate`程序，以查看是否有任何日志需要轮转
- 大多数日志文件每周轮转一次，但是`logrotate`轮转文件的速度有时较快，有时较慢，或在文件达到特定大小时进行轮转

### 分析Syslog条目
&#8195;&#8195;日志消息在文件的开头显示最旧的消息，在文件的末尾显示最新的消息。`rsyslog`服务在日志文件中记录条目时采用一种标准的格式。示例`/var/log/secure`日志文件中的日志消息：
```
[root@redhat8 ~]# cat /var/log/secure
May 23 06:28:10 redhat8 unix_chkpwd[6502]: password check failed for user (huang)
May 23 06:28:10 redhat8 sshd[6500]: pam_unix(sshd:auth): authentication failure; log
name= uid=0 euid=0 tty=ssh ruser= rhost=192.168.100.1  user=huang
```
日志消息条目依次说明：
- 记录该日志条目的时间戳
- 发送该日志消息的主机名称
- 发送该日志消息的程序或进程名称和PID编号
- 发送的实际消息 

### 监控日志
&#8195;&#8195;监控一个或多个日志文件中的事件有助于重现问题。`tail -f /path/to/file`命令输出指定文件的最后10行(或自定义行)，并在新行写入到文件中时继续输出它们。可以一个终端运行命令，另一个终端进行日志监控。示例：
```
[root@redhat8 ~]# tail -f /var/log/secure
May 23 06:28:10 redhat8 unix_chkpwd[6502]: password check failed for user (huang)
May 23 06:28:10 redhat8 sshd[6500]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.100.1  user=huang
May 23 06:28:11 redhat8 sshd[6500]: Failed password for huang from 192.168.100.1 port 52524 ssh2
May 23 06:28:14 redhat8 sshd[6500]: error: Received disconnect from 192.168.100.1 port 52524:0:  [preauth]
May 23 06:28:14 redhat8 sshd[6500]: Disconnected from authenticating user huang 192.168.100.1 port 52524 [preauth]
^C
```
### 手动发送Syslog消息
&#8195;&#8195;`logger`命令可以发送消息到`rsyslog`服务。默认情况下，它将优先级为`notice`(user.notice)的消息发送给`user`设备，除非通过`-p`选项另有指定。对测试`rsyslog`服务配置更改很有用。向`rsyslog`服务发送消息并记录在`/var/log/boot.log`日志文件中示例如下：
```
[root@redhat8 ~]# logger -p local7.notice "Log entry created on host redhat8"
[root@redhat8 ~]# tail -n 2 /var/log/boot.log
May 23 06:50:11 redhat8 root[6720]: Log entry created on host redhat8
```
