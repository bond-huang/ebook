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
## 查看系统日志条目
### 查找事件
&#8195;&#8195;`systemd-journald`服务将日志数据存储在带有索引的结构化二进制文件中，该文件称为日志。此数据包含与日志事件相关的额外信息。在RHEL8中，默认情况下`/run/log`目录用于存储系统日志。`/run/log`的内容在系统重启后将被清除。用户可以更改此设置。    
&#8195;&#8195;使用`journalctl`命令来查看日志中的所有消息，或根据各种选项和标准来搜索特定事件。如果以root身份运行该命令，则对日志具有完全访问权限。普通用户也可以使用此命令，但可能会被限制查看某些消息。命令示例：
```
[root@redhat8 ~]# journalctl |grep sshd
May 22 22:20:57 redhat8 systemd[1]: Reached target sshd-keygen.target.
May 22 22:20:59 redhat8 sshd[1137]: Server listening on 0.0.0.0 port 22.
May 22 23:43:49 redhat8 sshd[1137]: Received SIGHUP; restarting.
May 22 23:43:49 redhat8 sshd[1137]: Server listening on :: port 22.
May 23 06:28:10 redhat8 sshd[6500]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.100.1  user=huang
May 23 06:28:11 redhat8 sshd[6500]: Failed password for huang from 192.168.100.1 port 52524 ssh2
```
`journalctl`命令突出显示重要的日志消息：
- 优先级为`notice`或`warning`的消息显示为粗体文本
- 优先级为·`error`或以上的消息则显示为红色文本

&#8195;&#8195;默认情况下`journalctl -n`显示最后10个日志条目。可以借助可选参数对此进行调整，它可以指定要显示的日志条目数。示例显示最后3个日志条目：
```
[root@redhat8 ~]# journalctl -n 3
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:20:01 EDT. --
May 23 08:18:01 redhat8 nm-dispatcher[7546]: req:1 'dhcp4-change' [ens160]: start running ordered scripts...
May 23 08:18:27 redhat8 PackageKit[7494]: daemon quit
May 23 08:20:01 redhat8 CROND[7577]: (root) CMD (/usr/lib64/sa/sa1 1 1)
```
&#8195;&#8195;与`tail -f`命令相似，`journalctl -f`命令输出系统日志的最后10行，并在新日志条目写入到日志中时继续输出它们。退出`journalctl -f`进程使用`Ctrl+C`组合键。示例：
```
[root@redhat8 ~]# journalctl -f
-- Logs begin at Sun 2022-05-22 22:20:52 EDT. --
...output omitted...
May 23 08:18:01 redhat8 systemd[1]: Started Network Manager Script Dispatcher Service.
May 23 08:18:27 redhat8 PackageKit[7494]: daemon quit
May 23 08:20:01 redhat8 CROND[7577]: (root) CMD (/usr/lib64/sa/sa1 1 1)
^C
``` 
&#8195;&#8195;`journalctl -p`可以接受优先级的名称或编号作为参数，并显示该优先级别及以上的日志条目。`journalctl`命令支持`debug`、`info`、`notice`、`warning`、`err`、`crit`、`alert`和`emerg`优先级。示例列出优先级为`err`或以上的日志条目：
```
[root@redhat8 ~]# journalctl -p err
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:20:01 EDT. --
May 22 22:20:52 redhat8 kernel: Detected CPU family 17h model 96
May 22 22:21:05 redhat8 dnsmasq[2095]: FAILED to start up
...output omitted...
``` 
&#8195;&#8195;查找具体的事件时，可以将输出限制为特定的时间段。`journalctl`命令有两个选项，分别是 `--since`和`--until`选项，它们可以将输出限制为特定的时间范围：
- 这两个选项都采用格式为`YYYY-MM-DD hh:mm:ss`的时间参数（必须使用双引号，以保留选项中的空格）
- 如果省略日期，则命令会假定日期为当天
- 如果省略时间，则命令假定为自`00:00:00`起的一整天
- 除了日期和时间字段外，这两个选项还接受`yesterday`、`today`和`tomorrow`作为有效的参数

示例列出昨天记录中的所有日志条目：
```
[root@redhat8 ~]# journalctl --since yesterday
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:33:00 EDT. --
May 22 22:20:52 redhat8 kernel: BIOS-provided physical RAM map:
May 22 22:20:52 redhat8 kernel: vmware: using sched offset of 7866811847 ns
May 22 22:20:52 redhat8 kernel: e820: remove [mem 0x000a0000-0x000fffff] usable
May 22 22:20:52 redhat8 kernel: MTRR default type: uncachable
...output omitted...
```
列出范围从`2022-05-23 20:30:00`到`2022-05-24 12:00:00`的所有日志条目：
```
[root@redhat8 ~]# journalctl --since "2022-05-23 20:30:00" --until "2022-05-24 12:00
:00"-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:33:00 EDT. --
-- No entries --
```
可以指定相对于当前的某个时间以后的所有条目，例如上一小时所有条目：
```
[root@redhat8 ~]# journalctl --since "-1 hour"
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:40:01 EDT. --
May 23 07:48:00 redhat8 NetworkManager[1112]: <info>  [1653306480.3308] dhcp4 (ens160):   address 192.168.100.131
May 23 07:48:00 redhat8 NetworkManager[1112]: <info>  [1653306480.3312] dhcp4 (ens160):   expires in 1800 seconds
May 23 07:48:00 redhat8 NetworkManager[1112]: <info>  [1653306480.3312] dhcp4 (ens160):   nameserver '192.168.100.2'
...output omitted...
```
&#8195;&#8195;除了日志的可见内容外，日志条目中还附带了只有在打开详细输出时才可看到的字段。任何显示的额外字段都可用于过滤日志查询的输出。这可减少查找日志中特定事件的复杂搜索的输出。示例：
```
[root@redhat8 ~]# journalctl -o verbose
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:50:01 EDT. --
Sun 2022-05-22 22:20:52.615448 EDT [s=4180d79441bd4539bc3992ce20aa83ba;i=1;b=65c88922c4b5449a95f86d8d7fdf9580;m=100fd7;t=5dfa47d414d18;x=fde612134042eb16]
    _SOURCE_MONOTONIC_TIMESTAMP=0
    _TRANSPORT=kernel
    PRIORITY=5
    SYSLOG_FACILITY=0
    SYSLOG_IDENTIFIER=kernel
    MESSAGE=Linux version 4.18.0-80.el8.x86_64 (mockbuild@x86-vm-08.build.eng.bos.redhat.com) (gcc version 8.2.1 20180905 (Red Hat 8.2.1-3) (GCC)) #1 SMP Wed Mar 13 12:02>
    _BOOT_ID=65c88922c4b5449a95f86d8d7fdf9580
    _MACHINE_ID=0d4f35dc451749ecad3ae466d7cbf09a
    _HOSTNAME=redhat8
...output omitted...
```
以下是系统日志的常用字段，可用于搜索与特定进程或事件相关的行：
- `_COMM`是命令的名称
- `_EXE`是进程的可执行文件的路径
- `_PID`是进程的PID
- ` _UID`是运行该进程的用户的UID
- `_SYSTEMD_UNIT`是启动该进程的`systemd`单元

显示与`PID`为`1137`的`sshd.service systemd`进程单元相关的所有日志条目：
```
[root@redhat8 ~]# journalctl _SYSTEMD_UNIT=sshd.service _PID=1137
-- Logs begin at Sun 2022-05-22 22:20:52 EDT, end at Mon 2022-05-23 08:57:09 EDT. --
May 22 22:20:59 redhat8 sshd[1137]: Server listening on 0.0.0.0 port 22.
May 22 22:20:59 redhat8 sshd[1137]: Server listening on :: port 22.
May 22 23:43:49 redhat8 sshd[1137]: Received SIGHUP; restarting.
May 22 23:43:49 redhat8 sshd[1137]: Server listening on 0.0.0.0 port 22.
May 22 23:43:49 redhat8 sshd[1137]: Server listening on :: port 22.
```
## 保留系统日志
### 永久存储系统日志
&#8195;&#8195;默认情况下，系统日志保存在`/run/log/journal`目录中，系统重启时这些日志会被清除。可以在`/etc/systemd/journald.conf`文件中更改`systemd-journald`服务的配置设置，文件中的`Storage`参数决定系统日志以易失性方式存储，还是在系统重启后持久保留。参数选项说明如下：
- `persistent`：将日志存储在`/var/log/journal`目录中，在系统重启后持久保留：
    - 如果`/var/log/journal`目录不存在，`systemd-journald`服务会创建
- `volatile`：将日志存储在易失性`/run/log/journal`目录中：
    - 因为`/run`文件系统是临时的，仅存在于运行时内存中，存储在其中的数据（包括系统日志）不会在系统启后保留
- `auto`：如果`/var/log/journal`目录存在，那么会使用持久存储，否则使用易失性存储：
    - 如果未设置`Storage`参数，此为默认操作
- `none`：不使用任何存储。所有的日志都会被丢弃，但日志转发仍将按预期工作

&#8195;&#8195;持久系统日志的优点是系统启动后就可立即利用历史数据。然而，即便是持久日志，并非所有数据都将永久保留。限制规则说明如下：
- 该日志具有一个内置日志轮转机制，会在每个月触发
- 默认情况下，日志的大小不能超过所处文件系统的10%，也不能造成文件系统的可用空间低于15%。
- 可以在`/etc/systemd/journald.conf`中为运行时和持久日志调整这些值
- 当`systemd-journald`进程启动时，会记录当前的日志大小限额

示例命令输出显示了反映当前大小限额的日志条目：
```
[root@redhat8 ~]# journalctl | grep -E 'Runtime|System journal'
May 23 03:51:14 redhat8 systemd-journald[5081]: Runtime journal (/run/log/journal/0d4f35dc451749ecad3ae466d7cbf09a) is 24.0M, max 90.3M, 66.3M free.
May 23 04:26:30 redhat8 systemd-journald[5247]: Runtime journal (/run/log/journal/0d4f35dc451749ecad3ae466d7cbf09a) is 32.0M, max 90.3M, 58.3M free.
May 23 07:16:30 redhat8 systemd-journald[6837]: Runtime journal (/run/log/journal/0d4f35dc451749ecad3ae466d7cbf09a) is 40.0M, max 90.3M, 50.3M free.
```
#### 配置持久系统日志
&#8195;&#8195;以超级用户身份运行编辑`/etc/systemd/journald.conf`文件，将`Storage`设为`persistent`，`systemd-journald`服务配置在系统重启后持久保留系统日志。配置查询示例：
```
[root@redhat8 ~]# cat /etc/systemd/journald.conf |grep Storage
Storage=putstents
```
编辑完成后需重启`systemd-journald`服务使配置更改生效：
```
[root@redhat8 ~]# systemctl restart systemd-journald
```
&#8195;&#8195;`systemd-journald`服务重启成功后，`/var/log/journal`目录已创建好，并包含一个或多个子目录。这些子目录的长名称中包含十六进制字符，目录中有`*.journal`文件，它是存储带有索引的结构化日志条目的二进制文件：
```
[root@redhat8 ~]# ls /var/log/journal
0d4f35dc451749ecad3ae466d7cbf09a
[root@redhat8 ~]# ls /var/log/journal/0d4f35dc451749ecad3ae466d7cbf09a/
system.journal
```
&#8195;&#8195;由于系统日志在系统重启后保留，在`journalctl`命令的输出中会有大量的条目，包括当前系统引导以及之前系统引导的条目。要将输出限制为特定的系统启动，使用`-b`选项。示例检索第一次系统启动的条目：
```
[root@redhat8 ~]# journalctl -b 1
```
检索第二次系统启动的条目，当系统重启至少两次时，参数才有意义：
```
[root@redhat8 ~]# journalctl -b 2
Data from the specified boot (+2) is not available: No such boot ID in journal
```
仅检索当前系统启动的条目：
```
[root@redhat8 ~]# journalctl -b
```
&#8195;&#8195;利用永久日志调试系统崩溃时，通常需要将日志查询限制为崩溃发生之前的重新启动。可以给`-b`选项附上一个负数值，指示输出中应包含过去多少次系统引导。例如，`journalctl -b -1`将输出限制为上一次启动。
## 维护准确的时间
### 设置本地时钟和时区
&#8195;&#8195;对于在多个系统间分析日志文件而言，正确同步系统时间至关重要。网络时间协议(NTP)是计算机用于通过互联网提供并获取正确时间信息的一种标准方法。还可以通过高质量硬件时钟为本地客户端提供准确时间。

`timedatectl`命令简要显示当前的时间相关系统设置：
```
[root@redhat8 ~]# timedatectl
               Local time: Mon 2022-05-23 09:37:45 EDT
           Universal time: Mon 2022-05-23 13:37:45 UTC
                 RTC time: Tue 2022-05-24 09:44:05
                Time zone: America/New_York (EDT, -0400)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```
系统提供了包含时区的数据库，`timedatectl list-timezones`命令列出：
```
[root@redhat8 ~]# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
...output omitted...
```
&#8195;&#8195;可以使用`tzselect`命令识别正确的`zoneinfo`时区名称。它以交互方式向用户提示关于系统位置的问题，然后输出正确时区的名称。它不会对系统的时区设置进行任何更改。命令示例如下：
```
[root@redhat8 ~]# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
#? 
```
&#8195;&#8195;超级用户可以用`timedatectl set-timezone`命令更改系统设置来更新当前的时区。以下 `timedatectl`命令将当前时区更新为`Asia/Shanghai`：
```
[root@redhat8 ~]# timedatectl set-timezone Asia/Shanghai
[root@redhat8 ~]# timedatectl
               Local time: Mon 2022-05-23 21:47:39 CST
           Universal time: Mon 2022-05-23 13:47:39 UTC
                 RTC time: Tue 2022-05-24 09:53:14
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```
&#8195;&#8195;如果需要在特定服务器上使用协调世界时(UTC)，可将其时区设置为UTC。`tzselect`命令不包括UTC时区的名称。使用`timedatectl set-timezone UTC`命令可将系统的当前时区设置为`UTC`。示例：
```
[root@redhat8 ~]# timedatectl set-timezone UTC
[root@redhat8 ~]# timedatectl
               Local time: Mon 2022-05-23 13:49:32 UTC
           Universal time: Mon 2022-05-23 13:49:32 UTC
                 RTC time: Tue 2022-05-24 09:54:58
                Time zone: UTC (UTC, +0000)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```
&#8195;&#8195;使用`timedatectl set-time`命令可更改系统的当前时间。时间以`YYYY-MM-DD hh:mm:ss`格式指定，其中可以省略日期或时间。示例将时间更改为`23:57:00`：
```
[root@redhat8 ~]# timedatectl set-time 23:57:00
Failed to set time: NTP unit is active
[root@redhat8 ~]# timedatectl set-ntp false
[root@redhat8 ~]# timedatectl set-time 23:57:00
[root@redhat8 ~]# timedatectl
               Local time: Mon 2022-05-23 23:57:03 UTC
           Universal time: Mon 2022-05-23 23:57:03 UTC
                 RTC time: Mon 2022-05-23 23:57:03
                Time zone: UTC (UTC, +0000)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```
&#8195;&#8195;上面示例中NTP服务开启了无法提示设置时间失败，使用`timedatectl set-ntp`命令可启用或禁用NTP同步（自动调整时间）。选项需要`true`或`false`参数将它打开或关闭。
### 配置和监控Chronyd
&#8195;&#8195;`chronyd`服务通过与配置的`NTP`服务器进行同步，使通常不准确的本地硬件时钟(RTC)保持正确运行。如果没有可用的网络连接，`chronyd`将计算`RTC`时钟漂移，记录在`/etc/chrony.conf`配置文件指定的`driftfile`变量中：
- 默认情况下，`chronyd`服务使用`NTP Pool Project`的服务器同步时间，不需要额外的配置。当涉及的计算机位于孤立网络中时，可能需要更改`NTP`服务器
- NTP时间源的`stratum`决定其质量。`stratum`确定计算机与高性能参考时钟偏离的跃点数。参考时钟是`stratum 0`时间源。与之直接关联的`NTP`服务器是`stratum 1`，而与该`NTP`服务器同步时间的计算机则是 `stratum 2`时间源
- `server`和`peer`是用户可以在`/etc/chrony.conf`配置文件中声明的两种时间源类别。`server`比本地 `NTP`服务器高一个级别，而`peer`则属于同一级别。可以指定多个`server`和多个`peer`，每行指定一个：
    - `server`行的第一个参数是`NTP`服务器的`IP`地址或`DNS`名称
    - 在服务器`IP`地址或名称后，可以列出该服务器的一系列选项。建议使用`iburst`选项，因为在服务启动后，会在很短时间内执行四种测量，获得更加精确的初始时钟同步

`/etc/chrony.conf`文件示例：
```
[root@redhat8 ~]# cat /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
pool 2.rhel.pool.ntp.org iburst
# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3
...output omitted...
```
&#8195;&#8195;`/etc/chrony.conf`文件中的以下`server classroom.example.com iburst`行会使`chronyd`服务使用 `classroom.example.com NTP`时间源：
```
# Use public servers from the pool.ntp.org project.
...output omitted...
server classroom.example.com iburst
...output omitted...
```
`chronyd`指向本地时间源`classroom.example.com`后需重启该服务：
```
[root@redhat8 ~]# systemctl restart chronyd
```
&#8195;&#8195;`chronyc`命令充当`chronyd`服务的客户端。设置NTP同步后，使用`chronyc sources`命令验证本地系统是否使用NTP服务器无缝同步系统时钟。如需更详细输出，使用`chronyc sources -v`命令。示例如下：
```
[root@redhat8 ~]# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- makaki.miuku.net              2   6   175    96    +87ms[  +74ms] +/-  121ms
^- de-user.deepinid.deepin.>     3   6   377    31  +8635us[+8635us] +/-  118ms
^- stratum2-1.ntp.mow01.ru.>     2   6   377    32    +43ms[  +43ms] +/-  106ms
^* 111.230.189.174               2   6   377    32  -2532us[  -15ms] +/-   28ms
```
示例说明：
- `S`（源状态）字段中的`*`字符表示`111.230.189.174`服务器已被用作时间源，是计算机当前与之同步的NTP服务器

## 练习