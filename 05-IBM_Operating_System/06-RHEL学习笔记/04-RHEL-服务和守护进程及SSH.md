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
&#8195;&#8195;SSH通过公钥加密的方式保持通信安全。当用户使用`ssh`命令连接到SSH服务器时，该命令会检查它在本地已知主机文件中是否有该服务器的公钥副本：
- 系统管理员可能在`/etc/ssh/ssh_known_hosts`中已进行预配置，或者用户的主目录中可能有包含公钥的`~/.ssh/known_hosts`文件
- 在特定于用户的`~/.ssh/config`文件或系统范围的`/etc/ssh/ssh_config`中将`StrictHostKeyChecking`参数设为`yes`，使得`ssh`命令在公钥不匹配时始终中断SSH连接。
- 如果客户端的已知主机文件中没有公钥的副本，`ssh`命令会询问是否仍要登录。如果仍进行登录，公钥的副本就会保存到`~/.ssh/known_hosts`文件中，以便将来能自动确认服务器的身份

&#8195;&#8195;如果由于硬盘驱动器故障而导致公钥丢失或由于某些正当理由而导致公钥被更换，并由此更改了服务器的公钥，用户需要编辑已知的主机文件以确保将旧公钥条目替换为新公钥条目。公钥存储在`/etc/ssh/ssh_known_hosts`及SSH客户端上每个用户的`~/.ssh/known_hosts`文件中。示例：
```
[root@redhat8 ~]# cat ~/.ssh/known_hosts
192.168.122.106 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYA
AABBBNPBB8DiAz6yrGleDV5/py/heyo9is/Ym0mvyvZ4Ivx0+NfxP4T3jBG18mdf7cVKJROhdjH49204YB2MbJ7/UKM=
```
每个公钥各占一行，字段说明：
- 第一个字段是共享该公钥的主机名和IP地址的列表
- 第二个字段是公钥的加密算法
- 最后一个字段是公钥本身 

&#8195;&#8195;用户所连接的每个远程SSH服务器都将其公钥存储在`/etc/ssh`目录下扩展名为`.pub`的文件中。文件查看示例如下：
```
[root@redhat8 ~]# ls /etc/ssh/*key.pub
/etc/ssh/ssh_host_ecdsa_key.pub    /etc/ssh/ssh_host_rsa_key.pub
/etc/ssh/ssh_host_ed25519_key.pub
```
### 配置基于SSH密钥的身份验证
#### 基于SSH密钥的身份验证
&#8195;&#8195;用户可以配置SSH服务器，以便能通过基于密钥的身份验证在不使用密码的情况下进行身份验证。这种身份验证基于私钥-公钥方案。用户需要生成加密密钥文件的一个匹配对，一个是私钥，另一个是匹配的公钥：
- 私钥文件用作身份验证凭据，像密码一样，必须妥善保管
- 公钥复制到用户希望连接到的系统，用于验证私钥。公钥并不需要保密

&#8195;&#8195;使用`ssh-keygen`命令创建用于进行身份验证的私钥和匹配的公钥。默认情况下，私钥和公钥分别保存` ~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`文件中：
```
[root@redhat8 ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:3sVt/1S1jOf72M4GvoERmWoo/LwaN8CuQPuA0c/lDvI root@redhat8
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|             o   |
|            +   .|
| .   o   . o oo o|
|. o   * S o +.o+.|
| + + + * o . ++..|
|. = + + * . ...oo|
|   * + o o    .*+|
|    E o..     o=B|
+----[SHA256]-----+
```
&#8195;&#8195;如果未在`ssh-keygen`提示时指定密语，则生成的私钥不受保护。在这种情况下，任何拥有此私钥文件的人都可以使用它进行身份验证。如果设置了密码，则在使用私钥进行身份验证时需要输入此密语。下例中的`ssh-keygen`命令显示创建受密语保护的私钥及其公钥：
```
[root@redhat8 ~]# ssh-keygen -f .ssh/key-with-pass
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in .ssh/key-with-pass.
Your public key has been saved in .ssh/key-with-pass.pub.
The key fingerprint is:
SHA256:R7lppczeSK/cUd7vHlqzz1md2RT27OdsMQLQfmG2Fd8 root@redhat8
The key's randomart image is:
+---[RSA 2048]----+
|          .    ..|
|         . o + .o|
|          = + +oE|
|         + B o..o|
|        S X o . +|
|         = + + *=|
|          o + o*X|
|         . o .o+X|
|          o .. *X|
+----[SHA256]-----+
```
示例说明：
- `-f`选项与`ssh-keygen`命令配合可用来确定保存密钥的文件。在示例中，私钥和公钥分别保存在`/home/user/.ssh/key-with-pass`及`/home/user/.ssh/key-with-pass.pub`文件中。
- 在进一步生成SSH密钥对期间，除非指定唯一的文件名，否则系统会询问您是否允许覆盖现有的`id_rsa`和 `id_rsa.pub`文件。如果覆盖现有的`id_rsa`和`id_rsa.pub`文件，那么必须在具有旧公钥的所有SSH服务器上使用新公钥替换旧公钥
- 生成SSH密钥后，密钥将默认存储在用户主目录下的`.ssh/`目录中。私钥和公钥的权限模式必须分别为`600`和 `644`

#### 共享公钥
&#8195;&#8195;在可以使用基于密钥的身份验证之前，需要将公钥复制到目标系统上。`ssh-copy-id`命令可将 SSH密钥对的公钥复制到目标系统。如果在运行`ssh-copy-id`时省略了公钥文件的路径，会使用默认的`/home/user/.ssh/id_rsa.pub`文件。命令示例：
```
[root@redhat8 ~]# ssh-copy-id -i .ssh/key-with-pass huang@remotehost
```
&#8195;&#8195;将公钥成功传输到远程系统后，可以使用对应的私钥对远程系统进行身份验证，同时通过SSH登录远程系统。如果在运行`ssh`命令时省略了私钥文件的路径 ，会使用默认的`/home/user/.ssh/id_rsa`文件。命令示例：
```
[root@redhat8 ~]# ssh -i .ssh/key-with-pass huang@remotehost
```
#### 使用ssh-agent进行非交互式身份验证
&#8195;&#8195;如果用户的SSH私钥受密语保护，通常必须输入密语才能使用私钥进行身份验证。可以使用名为`ssh-agent`的程序临时将密语缓存到内存中，当使用SSH通过私钥登录另一个系统时，`ssh-agent`会自动提供密语：
- 如果是初始登录GNOME图形桌面环境，可能会自动启动并配置`ssh-agent`程序，具体取决于您本地系统的配置
- 如果是在文本控制台上进行登录，使用ssh进行登录，或者使用`sudo`或`su`，可能需要为该会话手动启动 `ssh-agent`
- 当用户注销启动了`ssh-agent`的会话时，将退出进程，并且用户的私钥密语也将从内存中清除 

使用以下命令手动启动： 
```
[root@redhat8 ~]# eval $(ssh-agent)
Agent pid 3812
```
&#8195;&#8195;`ssh-agent`开始运行后，需要告诉它用户的私钥密语或密钥。使用`ssh-add`命令。示例使用`ssh-add`命令添加分别来自`/home/user/.ssh/id_rsa`（默认）和`/home/user/.ssh/key-with-pass`文件的私钥：
```
[root@redhat8 ~]# ssh-add
Identity added: /root/.ssh/id_rsa (root@redhat8)
[root@redhat8 ~]# ssh-add .ssh/key-with-pass
Enter passphrase for .ssh/key-with-pass: 
Identity added: .ssh/key-with-pass (root@redhat8)
```
### 自定义OpenSSH服务配置
#### 配置OpenSSH服务器
&#8195;&#8195;`sshd`守护进程提供OpenSSH服务。主配置文件为`/etc/ssh/sshd_config`。OpenSSH服务器的默认配置也能良好运行。用户可能希望进行一些更改以增强系统的安全性。例如可能希望禁止直接远程登录root帐户，或者可能希望禁止使用基于密码的身份验证（偏向于使用SSH私钥身份验证）。 
#### 禁止超级用户使用SSH登录
最好禁止从远程系统直接登录root用户帐户。允许以root直接登录的一些风险包括：
- 所有Linux系统上都默认存在用户名root，因此潜在的攻击者只需要猜测其密码，而不必猜测有效的用户名与密码组合。这为攻击者降低了复杂度
- root用户具有不受限制的特权，因此它的泄露可能会给系统造成最大的损坏
- 从审核的角度看，很难跟踪是哪个授权用户以root的身份登录并做出了更改。如果用户必须以普通用户身份登录并切换到root帐户，则会生成一个日志事件，可用于帮助确定责任

&#8195;&#8195;OpenSSH服务使用`/etc/ssh/sshd_config`配置文件中的`PermitRootLogin`配置设置，以允许或禁止用户以root身份登录系统。查看示例如下：
```
[root@redhat8 ~]# cat /etc/ssh/sshd_config |grep PermitRootLogin
PermitRootLogin yes
# the setting of "PermitRootLogin without-password".
```
参数配置说明：
- 当`PermitRootLogin`参数默认设为`yes`，用户被允许以root身份登录系统。
- 要防止用户以root身份登录系统，可将该值设为`no`。
- 若要禁止基于密码的身份验证，但允许对root执行基于私钥的身份验证，可将`PermitRootLogin`参数设为 `without-password`

SSH服务(sshd)必须重新加载才能使更改生效。示例如下：
```
[root@host ~]# systemctl reload sshd
```
#### 禁止对SSH进行基于密码的身份验证
仅允许基于私钥登录远程命令行有诸多优点：
- 攻击者无法使用密码猜测攻击来远程入侵系统上的已知帐户
- 对于受密语保护的私钥而言，攻击者同时需要密语和私钥的副本。对于密码而言，攻击者则只需要密码
- 通过将受密语保护的私钥与`ssh-agent`配合使用，由于密语的输入频率较低，因此被泄露的几率会较小，而且对用户来说会更便于登录

&#8195;&#8195;OpenSSH服务使用`/etc/ssh/sshd_config`配置文件中的`PasswordAuthentication`参数，用于控制用户在登录系统时能否使用基于密码的身份验证。查看示例如下：
```
[root@redhat8 ~]# cat /etc/ssh/sshd_config |grep PasswordAuthentication
#PasswordAuthentication yes
PasswordAuthentication yes
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication, then enable this but set PasswordAuthentication
```
参数配置说明：
- 配置文件中`PasswordAuthentication`参数的默认值是`yes`，`SSH`服务允许用户在登录系统时使用基于密码的身份验证
- `PasswordAuthentication`的值为`no`时禁止用户使用基于密码的身份验证

注意事项
- 用户每当更改`/etc/ssh/sshd_config`文件时，都必须重新加载`sshd`服务让更改生效
- 如果为`ssh`关闭基于密码的身份验证，则需要有一种办法来确保用户在远程服务器上的`~/.ssh/authorized_keys`文件中有公钥，以便可以登录

## 练习