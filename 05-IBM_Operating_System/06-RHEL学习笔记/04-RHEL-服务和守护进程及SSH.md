# RHEL-服务和守护进程及SSH
## 控制服务和守护进程
### systemctl命令摘要
&#8195;&#8195;`systemct`l命令可以在运行中的系统上启动和停止服务，也可启用或禁用服务在系统引导时自动启动。下表是服务管理实用命令：

命令|任务描述
:---|:---
systemctl status UNIT|查看有关单元状态的详细信息
systemctl stop UNIT|在运行中的系统上停止一项服务
systemctl start UNIT|在运行中的系统上启动一项服务	
systemctl restart UNIT|在运行中的系统上重新启动一项服务
systemctl reload UNIT|重新加载运行中服务的配置文件
systemctl mask UNIT|彻底禁用服务，使其无法手动启动或在系统引导时启动
systemctl unmask UNIT|使屏蔽的服务变为可用
systemctl enable UNIT|将服务配置为在系统引导时启动
systemctl disable UNIT|禁止服务在系统引导时启动
systemctl list-dependencies UNIT|列出指定单元需要的单元

### 识别自动启动的系统进程
#### systemd进程
&#8195;&#8195;`systemd`守护进程管理Linux的启动，一般包括服务启动和服务管理。它可在系统引导时以及运行中的系统上激活系统资源、服务器守护进程和其他进程。守护进程是在执行各种任务的后台等待或运行的进程：
- 一般情况下，守护进程在系统引导时自动启动并持续运行至关机或被手动停止
- 按照惯例，许多守护进程的名称以字母`d`结尾
- `systemd`意义上的服务通常指的是一个或多个守护进程，但启动或停止一项服务可能会对系统的状态进行一次性更改，不会留下守护进程之后继续运行（称为oneshot

在RHEL中，第一个启动的进程(PID 1)是`systemd`，提供的几项功能如下：
- 并行化功能（同时启动多个服务），它可提高系统的启动速度
- 按需启动守护进程，而不需要单独的服务
- 自动服务依赖关系管理，可以防止长时间超时。例如，只有在网络可用时，依赖网络的服务才会尝试启动
- 利用Linux控制组一起追踪相关进程的方式

#### 描述服务单元
`systemd`使用单元来管理不同类型的对象。常用单元类型： 
- 服务单元具有`.service`扩展名，代表系统服务。这种单元用于启动经常访问的守护进程，如web服务器
- 套接字单元具有`.socket`扩展名，代表`systemd`应监控的进程间通信(IPC)套接字。如果客户端连接套接字，`systemd`将启动一个守护进程并将连接传递给它。套接字单元用于延迟系统启动时的服务启动，或者按需启动不常使用的服务
- 路径单元具有`.path`扩展名，用于将服务的激活推迟到特定文件系统更改发生之后。这通常用于使用假脱机目录的服务，如打印系统

&#8195;&#8195;`systemctl`命令用于管理单元。可通过`systemctl -t help`命令来显示可用的单元类型。命令示例如下：
```
[root@redhat8 ~]# systemctl -t help
Available unit types:
service socket target device mount automount swap timer path slice scope
```
#### 列出服务单元
&#8195;&#8195;可以使用`systemctl`命令来探索系统的当前状态。以下命令会列出所有当前加载的服务单元，通过`--type=service`选项将列出的单元类型限制为服务单元：
```
[root@redhat8 ~]# systemctl list-units --type=service
  UNIT                               LOAD   ACTIVE SUB     DESCRIPTION   
  accounts-daemon.service            loaded active running Accounts Service
  atd.service                        loaded active running Job spooling tools
  auditd.service                     loaded active running Security Auditing Service
  avahi-daemon.service               loaded active running Avahi mDNS/DNS-SD Stack 
  bluetooth.service                  loaded active running Bluetooth service
  bolt.service                       loaded active running Thunderbolt system service
  chronyd.service                    loaded active running NTP client/server
...output omitted...
```
`systemctl list-units`命令输出中的列描述：
- UNIT：服务单元名称
- LOAD：systemd是否正确解析了单元的配置并将该单元加载到内存中
- ACTIVE：单元的高级别激活状态。此信息表明单元是否已成功启动
- SUB：单元的低级别激活状态。此信息指示有关该单元的更多详细信息。信息视单元类型、状态以及单元的执行方式而异
- DESCRIPTION：单元的简短描述

命令`systemctl`更多说明：
- 默认情况下，`systemctl list-units --type=service`命令仅列出激活状态为`active`的服务单元
- `--all`选项可列出所有服务单位，不论激活状态如何
- 可以使用`--state=`选项可按照`LOAD`、`ACTIVE`或`SUB`字段中的值进行筛选
- 不带任何参数运行`systemctl`命令可列出已加载和活动的单元

&#8195;&#8195;`systemctl list-units`命令显示`systemd`服务尝试解析并加载到内存中的单元；不显示已安装但未启用的服务。要查看所有已安装的单元文件的状态，使用`systemctl list-unit-files`命令，命令中`STATE`字段的有效条目有`enabled`、`disabled`、`static`和`masked`：
```
[root@redhat8 ~]# systemctl list-unit-files --type=service
UNIT FILE                                        STATE    
accounts-daemon.service                          enabled  
alsa-restore.service                             static    
arp-ethers.service                               disabled 
atd.service                                      enabled   
auth-rpcgss-module.service                       static   
avahi-daemon.service                             enabled  
...output omitted...
```
#### 查看服务状态
&#8195;&#8195;使用`systemctl status name.type`来查看特定单元的状态。如果未提供单元类型，则 `systemctl`将显示服务单元的状态（如果存在）：
```
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-05-14 23:05:22 EDT; 18h ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 1091 (sshd)
    Tasks: 1 (limit: 11366)
   Memory: 5.7M
   CGroup: /system.slice/sshd.service
           └─1091 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc>
May 15 10:39:48 redhat8 sshd[19043]: Accepted password for huang from 192.168.100.1 port 64838 ssh2
May 15 10:39:48 redhat8 sshd[19043]: pam_unix(sshd:session): session opened for user huang by (uid=0)
...output omitted...
```
此命令将显示服务的当前状态。各个字段的含义如下：

字段|描述
:---:|:---
Loaded|服务单元是否已加载到内存中
Active|服务单元是否正在运行，若是，它已经运行了多久 
Main PID|服务的主进程ID，包括命令名称
Status|有关该服务的其他信息

`systemctl`输出中的服务状态：

关键字|描述
:---:|:---
loaded|单元配置文件已处理
active (running)|正在通过一个或多个持续进程运行
active (exited)|已成功完成一次性配置
active (waiting)|运行中，但正在等待事件
inactive|不在运行
enabled|在系统引导时启动
disabled|未设为在系统引导时启动
static|无法启用，但可以由某一启用的单元自动启动

#### 验证服务的状态
&#8195;&#8195;`systemctl`命令提供了一些方法来验证服务的具体状态。可使用以下命令验证服务单元当前是否处于活动状态（正在运行）：
```
[root@redhat8 ~]# systemctl is-active sshd.service
active
```
验证服务单元是否已启用在系统引导期间自动启动：
```
[root@redhat8 ~]# systemctl is-enabled sshd.service
enabled
```
验证单元是否在启动过程中失败：
```
[root@redhat8 ~]# systemctl is-failed sshd.service
active
```
命令说明：
- 如果单元运行正常，该命令将返回`active`
- 如果启动过程中发生了错误，则返回`failed`
- 如果单元已被停止，将返回`unknown`或`inactive` 

要列出所有失败的单元，命令及示例如下：
```
[root@redhat8 ~]# systemctl --failed --type=service
  UNIT           LOAD   ACTIVE SUB    DESCRIPTION                           
● mcelog.service loaded failed failed Machine Check Exception Logging Daemon

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

1 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```
### 控制系统服务
#### 启动和停止服务
&#8195;&#8195;要启动服务，首先通过`systemctl status`验证它是否未在运行。然后，以root用户身份（必要时使用sudo）使用`systemctl start`命令。要停止当前正在运行的服务，使用`stop`参数来运行`systemctl`命令。示例如下：
```
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset>
   Active: active (running) since Sat 2022-05-14 23:05:22 EDT; 19h ago
...output omitted...
[root@redhat8 ~]# systemctl stop sshd
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset>
   Active: inactive (dead) since Sun 2022-05-15 18:13:40 EDT; 3s ago
...output omitted...
[root@redhat8 ~]# systemctl start sshd.service
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset>
   Active: active (running) since Sun 2022-05-15 18:15:32 EDT; 2s ago
...output omitted...
```
#### 重新启动和重新加载服务
&#8195;&#8195;重新启动服务使用`restart`参数来运行systemctl命令。重新加载使用`reload`参数来运行`systemctl`命令。重新启动和重新加载服务示例如下：
```
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
...output omitted...
 Main PID: 23572 (sshd)
...output omitted...
[root@redhat8 ~]# systemctl restart sshd
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
...output omitted...
 Main PID: 23579 (sshd)
 ...output omitted...
[root@redhat8 ~]# systemctl reload sshd
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
...output omitted...
 Main PID: 23579 (sshd)
...output omitted...
``` 
重新启动和重新加载服务说明：
- 在重新启动正在运行的服务期间，服务将停止然后启动。在重新启动服务时，进程`ID`会改变，并且在启动期间会关联新的进程`ID`
- 某些服务可以重新加载其配置文件，而无需重新启动。这个过程称为服务重新加载。重新加载服务不会更改与各种服务进程关联的进程`ID`

&#8195;&#8195;如果不确定服务是否具有重新加载配置文件更改的功能，使用`reload-or-restart`参数来运行 `systemctl`命令。如果重新加载功能可用，该命令将重新加载配置更改。否则，该命令将重新启动服务以实施新的配置更改。示例如下：
```
[root@redhat8 ~]# systemctl reload-or-restart sshd
[root@redhat8 ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
...output omitted...
 Main PID: 23579 (sshd)
...output omitted...
```
#### 列出单元依赖项
&#8195;&#8195;某些服务要求首先运行其他服务，从而创建对其他服务的依赖项。其他服务并不在系统引导时启动，而是仅在需要时启动。在这两种情况下，`systemd`和`systemctl`根据需要启动服务，不论是解决依赖项，还是启动不经常使用的服务。`systemctl list-dependencies UNIT`命令显示启动服务单元所需的依赖项的层次结构映射，使用`--reverse`选项列出反向依赖项。示例：
```
[root@redhat8 ~]# systemctl list-dependencies sshd.service
sshd.service
● ├─system.slice
● ├─sshd-keygen.target
● │ ├─sshd-keygen@ecdsa.service
● │ ├─sshd-keygen@ed25519.service
● │ └─sshd-keygen@rsa.service
● └─sysinit.target
●   ├─dev-hugepages.mount
...output omitted...
[root@redhat8 ~]# systemctl list-dependencies sshd.service --reverse
sshd.service
● └─multi-user.target
●   └─graphical.target
```
#### 屏蔽未屏蔽的服务
&#8195;&#8195;有时系统中安装的不同服务之间可能彼此冲突，屏蔽服务可防止管理员意外启动与其他服务冲突的服务。例如，有多种方法可以管理邮件服务器（如postfix和sendmail等）。示例如下：
```
[root@redhat8 ~]# systemctl mask sendmail.service
Created symlink /etc/systemd/system/sendmail.service → /dev/null.
[root@redhat8 ~]# systemctl list-unit-files --type=service
UNIT FILE                                        STATE       
sendmail.service                                 masked   
[root@redhat8 ~]# systemctl start sendmail.service
Failed to start sendmail.service: Unit sendmail.service is masked.
[root@redhat8 ~]# systemctl unmask sendmail.service
Removed /etc/systemd/system/sendmail.service.
```
示例说明及注意事项：
- 使用`systemctl mask`命令执行屏蔽服务单元
- 屏蔽操作会在配置目录中创建指向`/dev/null`文件的链接，该文件可阻止服务启动
- 尝试启动已屏蔽的服务单元会失败
- 使用`systemctl unmask`命令可取消屏蔽服务单元
- 禁用的服务可以手动启动，或通过其他单元文件启动，但不会在系统引导时自动启动。屏蔽的服务无法手动启动，也不会自动启动

#### 使服务在系统引导时启动或停止
&#8195;&#8195;在`systemd`配置目录中创建链接，使服务在系统引导时启动。`systemctl`命令可以创建和删除这些链接。示例命令如下：
```
[root@redhat8 ~]# systemctl enable cups.service
Created symlink /etc/systemd/system/printer.target.wants/cups.service → /usr/lib/sy
stemd/system/cups.service.Created symlink /etc/systemd/system/sockets.target.wants/cups.socket → /usr/lib/sys
temd/system/cups.socket.Created symlink /etc/systemd/system/multi-user.target.wants/cups.path → /usr/lib/sy
stemd/system/cups.path.
[root@redhat8 ~]# systemctl disable cups.service
Removed /etc/systemd/system/multi-user.target.wants/cups.path.
Removed /etc/systemd/system/sockets.target.wants/cups.socket.
Removed /etc/systemd/system/printer.target.wants/cups.service.
[root@redhat8 ~]# systemctl is-enabled cups.service
disabled
```
示例说明及注意事项：
- 在系统引导时启动服务，使用`systemctl enable`命令
- 执行引导启动命令后，会从服务单元文件（通常位于`/usr/lib/systemd/system`目录）创建一个符号链接，指向磁盘上供`systemd`寻找文件的位置，即`/etc/systemd/system/TARGETNAME.target.wants`目录
- 要禁用服务自动启动，使用`systemctl disable`命令，该命令将删除在启用服务时创建的符号链接。请注意禁用服务不会停止该服务
- 要验证服务是启用还是禁用状态，使用`systemctl is-enabled`命令

## 配置和保护SSH
### 使用SSH访问远程命令行
&#8195;&#8195;OpenSSH在RHEL系统上实施Secure Shell或SSh协议。SSH协议使系统能够通过不安全的网络以加密和安全的方式进行通信：
- 可以使用ssh命令来创建与远程系统的安全连接、以特定用户身份进行身份验证，并以该用户身份在远程系统上获取交互式shell会话
- 也可以使用ssh命令在远程系统上运行单个命令，而不运行交互式shell 

#### 安全Shell示例
使用与当前本地用户相同的用户名登录远程服务器`remotehost`，会提示进行身份验证：
```
[root@redhat8 ~]# ssh remotehost
```
使用用户名`huang`登录远程服务器`remotehost`。会提示进行身份验证：
```
[root@redhat8 ~]# ssh huang@remotehost
```
在`remotehost`远程系统上以`huang`用户身份运行`hostname`命令，而不访问远程交互式shell：
```
[root@redhat8 ~]# ssh huang@remotehost hostname
```
使用`exit`命令来注销远程系统。 
#### 识别远程用户
&#8195;&#8195;`w`命令可显示当前登录到计算机的用户列表。这对于显示哪些用户使用`ssh`从哪些远程位置进行了登录以及执行了何种操作等内容特别有用。示例：
```
[root@redhat8 ~]# w
 21:25:35 up 22:20,  3 users,  load average: 0.06, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
huang    tty2     tty2             18:12   22:20m 36.18s  0.12s /usr/libexec/tracke
root     pts/2    192.168.100.1    20:13    0.00s  0.22s  0.02s w
huang    pts/3    192.168.100.1    21:02   23:28   0.02s  0.02s -bash
```
#### SSH主机密匙