# RHEL-文件访问及进程管理
## 控制对文件访问
### 解释Linux文件系统权限
#### Linux文件系统权限
&#8195;&#8195;文件权限控制对文件的访问。可以为所属用户、所属组和系统上的非用户和非所属组成员的其他用户设置不同的权限。用户权限覆盖组权限，组权限覆盖other权限。有三种权限类别可应用：读取、写入和执行。如下表所示：

权限|对文件的影响|对目录的影响
:---|:---|:---
r(读取)|可以读取文件内容|可以列出目录的内容(文件名)
w(写入)|可以更改文件内容|可以创建或删除目录中的任一文件
x(执行)|可以作为命令执行文件|目录可以成为当前工作目录(可以cd，但需读取权限才能列出其中文件) 

文件权限更多说明：
- 用户通常对只读目录具有读取和执行权限，可以列出该目录，并且对其内容具有完整的只读访问权限
- 如果用户仅对某目录具有读取访问权限，可以列出其中文件的名称，但是其他信息（包括权限或时间戳）都不可用，也不可访问
- 如果用户仅对目录具有执行权限，则无法列出目录中的文件名。
- 如果知道有读取权限的文件的名称，可通过显式指定相对文件名，以便从目录外部访问该文件的内容
- 在文件所在的目录中拥有所有权或写入权限的任何人都可以删除此文件，不论此文件本身的所有权或权限如何（可以通过特殊权限粘滞位将其覆盖） 

#### 查看文件和目录的权限及所有权
&#8195;&#8195;命令`ls`的`-l`可以显示文件权限及所有权详细信息，`-d`选项可显示有关目录本身(非其内容)的详细信息。示例如下：
```
[root@redhat8 ~]# ls -l ls.ps
-rw-r--r--. 1 root root 20051 May  2 14:39 ls.ps
[root@redhat8 ~]# ls -ld Downloads
drwxr-xr-x. 2 root root 6 Mar 19  2021 Downloads
```
字段说明：
- 长列表的第一个字符表示文件类型，解译如下：
    - `-`是常规文件
    - `d`是目录
    - `l`是软链接
    - 其他字符代表硬件设备(b和c)或其他具有特殊用途的文件(p和s)
- 接下来的九个字符是文件权限。它们分为三组，每组三个字符，分别对应：
    - 应用于拥有该文件的用户的权限
    - 应用于拥有该文件的组的权限
    - 应用于其他所有用户的权限
    - 如果组中显示`rwx`，说明该类别具有读取、写入和执行三种权限。如果是`-`，则表示该类别没有这个权限
- 在链接数之后，第一个名称指定拥有该文件的用户，第二个名称指定拥有该文件的组

### 从命令行管理文件系统权限
#### 更改文件和目录权限
&#8195;&#8195;更改权限的命令为`chmod`，意为`change mode`（更改模式，权限也称为文件的模式）。可使用符号（符号法）或数值（数值法）来发布权限说明。通过符号法更改权限：
```
chmod WhoWhatWhich file/directory
```
示例说明：
- `Who`指`u`、`g`、`o`、`a`，分别代表用户、组、其他、全部
- `What`指`+`、`-`、`=`，根本代表添加、删除、精确设置
- `Which`指`r`、`w`、`x`，分别代表读取、写入、执行

说明：
- 在使用`chmod`通过符号法来更改权限时，仅当文件是目录或者已经为用户、组或其他人设置了执行权限时，使用大写的`X`作为权限标志才会添加执行权限
- `chmod`命令支持`-R`选项以递归方式对整个目录树中的文件设置权限。

&#8195;&#8195;在使用`-R`选项时，使用`X`选项以符号形式设置权限会非常有用。这将能够对目录设置执行（搜索）权限，以便在不更改大部分文件权限的情况下，访问这些目录的内容。使用`X`选项时要谨慎，如果某个文件设置有任何执行权限，则`X`也将会对该文件设置指定的执行权限。示例：
```
[root@redhat8 ~]# chmod -R g+rwX Videos
```
&#8195;&#8195;示例中以递归方式为组所有者设置对`Videos`及其所有子代的读、写访问权限，但将仅向已为用户、组或其他人设置了执行权限的目录和文件应用组执行权限。

示例如下：
```
[root@redhat8 testdir]# ls -l
total 0
-rw-r--r--. 1 root root 0 May  5 08:26 testfile1
-rw-r--r--. 1 root root 0 May  5 08:57 testfile2
[root@redhat8 testdir]# chmod go+wx testfile1
[root@redhat8 testdir]# chmod u-w testfile2
[root@redhat8 testdir]# ls -l
total 0
-rw-rwxrwx. 1 root root 0 May  5 08:26 testfile1
-r--r--r--. 1 root root 0 May  5 08:57 testfile2
```
通过数值法更改权限(`*`字符代表一个数字)：
```
chmod *** file|directory
```
示例说明：
- 每个数字代表一个访问级别的权限：用户、组、其他。
- 将所要添加的每个权限的数值加在一起，其中`4`代表读取，`2`代表写入，`1`代表执行。 权限由三位（或在设置高级权限时为四位）八进制数来表示
- 对于权限`-rwxr-x---`计算方法如下：
    - 对于用户，`rwx`计算为`4+2+1=7`
    - 对于组，`r-x`计算为`4+0+1=5`，
    - 对于其他用户，`---`表示为`0`
    - 将这三个放在一起，这些权限的数值法表示为`750`

示例如下：
```
[root@redhat8 testdir]# chmod 755 testfile2
[root@redhat8 testdir]# chmod 644 testfile1
[root@redhat8 testdir]# ls -l
total 0
-rw-r--r--. 1 root root 0 May  5 08:26 testfile1
-rwxr-xr-x. 1 root root 0 May  5 08:57 testfile2
```
#### 更改文件和目录的用户
&#8195;&#8195;只有root用户可以更改拥有文件的用户。组所有权可以由root用户或文件的所有者来设置。root用户可将文件所有权授予任何组，而普通用户仅可将文件所有权授予他们所属的组。使用`chown`命令可更改文件所有权。示例更改文件所有权：
```
[root@redhat8 testdir]# chown huang testfile1
[root@redhat8 testdir]# ls -l
total 0
-rw-r--r--. 1 huang root 0 May  5 08:26 testfile1
```
使用`-R`选项，以递归更改整个目录树的所有权：
```
[root@redhat8 ~]# chown -R christ testdir
[root@redhat8 ~]# ls -ld testdir
drwxr-xr-x. 2 christ root 40 May  5 08:57 testdir
```
在组名称之前加上冒号`:`，可以更改文件的组所有权：
```
[root@redhat8 testdir]# chown :huang testfile1
[root@redhat8 testdir]# ls -l
total 0
-rw-r--r--. 1 christ huang 0 May  5 08:26 testfile1
```
同时更改所有者和组，使用语法`owner:group`：
```
[root@redhat8 testdir]# chown root:root testfile1
[root@redhat8 testdir]# ls -l
total 0
-rw-r--r--. 1 root   root 0 May  5 08:26 testfile1
```
&#8195;&#8195;可以使用`chgrp`命令来更改组所有权。该命令的作用与`chown`类似，不同之处在于它仅用于更改组所有权，而且不需要组名前的冒号`:`。
### 管理默认权限和文件访问
#### 特殊权限
&#8195;&#8195;特殊权限构成了除了基本用户、组和其他类型之外的第四种权限类型。这些权限提供了额外的访问相关功能，超出了基本权限类型允许的范畴。特殊权限对文件和目录的影响如下表：

特殊权限|对文件的影响|对目录的影响
:---:|:---|:---
u+s (suid)|以拥有文件的用户身份，而不是以运行文件的用户身份执行文件|无影响
g+s (sgid)|以拥有文件的组身份执行文件|在目录中最新创建的文件将其组所有者设置为与目录的组所有者相匹配
o+t (sticky)|无影响|对目录具有写入访问权限的用户仅可以删除其所拥有的文件，而无法删除或强制保存到其他用户所拥有的文件

&#8195;&#8195;对可执行文件的`setuid`权限表示将以拥有该文件的用户的身份运行命令，而不是以运行命令的用户身份。如果所有者不具有执行权限，将由大写的`S`取代。示例如下：
```
[root@redhat8 ~]# ls -l /etc/passwd
-rw-r--r--. 1 root root 3027 May  5 04:59 /etc/passwd
[root@redhat8 ~]# chmod u+s /etc/passwd
[root@redhat8 ~]# ls -l /etc/passwd
-rwSr--r--. 1 root root 3027 May  5 04:59 /etc/passwd
[root@redhat8 ~]# chmod u-s /etc/passwd
[root@redhat8 ~]# chmod u+x /etc/passwd
[root@redhat8 ~]# chmod u+s /etc/passwd
[root@redhat8 ~]# ls -l /etc/passwd
-rwsr--r--. 1 root root 3027 May  5 04:59 /etc/passwd
```
&#8195;&#8195;对某目录的特殊权限`setgid`表示在该目录中创建的文件将继承该目录的组所有权，而不是继承自创建用户。这通常用于组协作目录，将文件从默认的专有组自动更改为共享组，或者当目录中的文件始终都应由特定的组所有时使用。典型示例目录是`/run/log/journal`：
```
[root@redhat8 ~]# ls -ld /run/log/journal
drwxr-sr-x. 3 root systemd-journal 60 May 13 04:54 /run/log/journal
```
&#8195;&#8195;如果对可执行文件设置了`setgid`，则命令以拥有该文件的组运行，而不是以运行命令的用户身份运行，其方式与`setuid`类似。如果组不具有执行权限，将会由大写`S`取代。`locate`命令为例：
```
[root@redhat8 ~]# ls -ld /usr/bin/locate
-rwx--s--x. 1 root slocate 47128 Aug 12  2018 /usr/bin/locate
```
&#8195;&#8195;针对目录的粘滞位将对文件删除设置特殊限制。只有文件的所有者（以及root）才能删除该目录中的文件。如果其他权限中不具有执行权限，将会由大写`T`取代。例如`/tmp`目录： 
```
[root@redhat8 ~]# ls -ld /tmp
drwxrwxrwt. 19 root root 4096 May 13 09:07 /tmp
```
#### 设置特殊权限
设置方式：
- 用符号表示：setuid=u+s；setgid=g+s；sticky=o+t
- 用数值表示（第四位）：setuid=4；setgid=2；sticky=1 

示例如下：
```
[root@redhat8 ~]# chmod g+s testdir
[root@redhat8 ~]# ls -ld testdir
drwxr-sr-x. 2 christ root 40 May  5 08:57 testdir
[root@redhat8 ~]# chmod 2770 Videos
[root@redhat8 ~]# ls -ld Videos
drwxrws---. 2 root root 6 Mar 19  2021 Videos
```
#### 默认文件权限
&#8195;&#8195;创建新文件或目录时，会为其分配初始权限。有两个因素会影响这些初始权限。首先是您要创建常规文件还是目录。其次是当前的`umask`：
- 创建新目录，操作系统首先会为其分配八进制权限`0777`(drwxrwxrwx)
- 创建新的常规文件，操作系统则为其分配八进制权限`0666`(-rw-rw-rw-)
- shell会话还会设置一个`umask`，以进一步限制初始设置的权限：
    - 这是一个八进制位掩码，用于清除由该进程创建的新文件和目录的权限。
    - 如果在`umask`中设置了一个位，则新文件中的对应的权限将被清除
    - 例如`umask`为`0002`可清除其他用户的写入位。前导零表示特殊的用户和组权限未被清除。`umask`为`0077`时，清除新创建文件的所有组和其他权限
-  通过一个数字参数使用`umask`命令，可以更改当前shell的`umask`。`umask`中的任何前导零均可以省略
- Bash shell用户的系统默认`umask`值在`/etc/profile`和`/etc/bashrc`文件中定义
- 用户可以在其主目录的`.bash_profile` 和`.bashrc`文件中覆盖系统默认值

`umask`示例如下：
```
[huang@redhat8 ~]$ umask
0002
[huang@redhat8 ~]$ touch umask.test
[huang@redhat8 ~]$ ls -l umask.test
-rw-rw-r--. 1 huang huang 0 May 13 10:17 umask.test
[huang@redhat8 ~]$ mkdir umasktest
[huang@redhat8 ~]$ ls -ld umasktest
drwxrwxr-x. 2 huang huang 6 May 13 10:18 umasktest
```
umask更多示例及说明：
- 将`umask`值设置为`0`，其他的文件权限将从读取改为读取和写入。其他权限中的目录权限将从读取和执行改为读取、写入和执行
- 将`umask`值设置为`007`，可以屏蔽其他权限中的所有文件和目录权
- `umask`为`027`可确保新文件的用户具有读写权限，并且组具有读取权限。新目录的组具有读取和写入权限，而其他权限中则没有访问权限
- 用户的默认`umask`由shell启动脚本设置。默认情况下，如果用户帐户的`UID`为`200`或以上，并且用户名和主要组名相同，就会分配一个值为`002`的`umask`。否则`umask`将为`022`

&#8195;&#8195;root用户可以通过添加`/etc/profile.d/local-umask.sh`的shell启动脚本来更改此设置。配置文件脚本示例如下：
```
[root@redhat8 profile.d]# cat /etc/profile.d/local-umask.sh
# Overrides default umask configuration
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 007
else
    umask 022
fi
```
## 监控和管理Linux进程
### 列出进程
#### 进程的定义
&#8195;&#8195;进程是已启动的可执行程序的运行中实例。在RHEL8系统上，第一个系统进程是`systemd`。进程由以下组成部分：
- 已分配内存的地址空间
- 安全属性，包括所有权凭据和特权
- 程序代码的一个或多个执行线程
- 进程状态 

进程的环境包括：
- 本地和全局变量
- 当前调度上下文
- 分配的系统资源，如文件描述符和网络端口

#### 进程状态
&#8195;&#8195;多任务处理操作系统中，每个CPU（或CPU核心）在一个时间点上处理一个进程。在进程运行时，它对CPU时间和资源分配的直接要求会有变化。进程分配有一个状态，它随着环境要求而改变。状态如下表所示：

名称|标志|内核定义的状态名称和描述
:---:|:---:|:---
运行|R|TASK_RUNNING：进程正在CPU上执行，或者正在等待运行。处于Running(或可运行)状态时，进程可能正在执行用户例程或内核例程(系统调用)，或者已排队并就绪
睡眠|S|TASK_INTERRUPTIBLE：进程正在等待某一条件：硬件请求、系统资源访问或信号。当事件或信号满足该条件时，该进程将返回到Running
睡眠|D|TASK_UNINTERRUPTIBLE：此进程也在睡眠，与S状态不同，不会响应信号。仅在进程中断可能会导致意外设备状态的情况下使用
睡眠|K|TASK_KILLABLE：与不可中断的D状态相同，但有所修改，允许等待中的任务响应要被中断（彻底退出）的信号。实用程序通常将可中断的进程显示为D状态
睡眠|I|TASK_REPORT_IDLE：D状态的一个子集。在计算负载平均值时，内核不会统计这些进程。用于内核线程。设置了 TASK_UNINTERRUPTABLE和TASK_NOLOAD标志。类似于TASK_KILLABLE，也是D状态的一个子集。它接受致命信号
已停止|T|TASK_STOPPED：进程已被停止(暂停)，通常是通过用户或其他进程发出的信号。进程可以通过另一信号返回到Running状态，继续执行(恢复)
已停止|T|TASK_TRACED：正在被调试的进程也会临时停止，并且共享同一个T状态标志
僵停|Z|EXIT_ZOMBIE：子进程在退出时向父进程发出信号。除进程身份(PID)之外的所有资源都已释放
僵停|X|EXIT_DEAD：当父进程清理(获取)剩余的子进程结构时，进程现在已彻底释放。此状态从不会在进程列出实用程序中看到

&#8195;&#8195;在创建进程时，系统会为进程分配一个状态。`top`命令的`S`列或`ps`的`STAT`列显示每个进程的状态。在单CPU系统上，一次只能运行一个进程。示例： 
```
[root@redhat8 ~]# top
top - 08:40:03 up  9:34,  1 user,  load average: 0.00, 0.00, 0.00
Tasks: 275 total,   1 running, 274 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.5 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1806.1 total,    229.5 free,    506.6 used,   1069.9 buff/cache
MiB Swap:   2048.0 total,   1869.7 free,    178.2 used.   1265.7 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   954 root      20   0  144636   8252   6684 S   0.7   0.4   0:28.66 vmtoolsd
   952 root      20   0  100456   1404   1240 S   0.3   0.1   0:03.32 irqbalance 
  1105 root      20   0   36088   3164   2272 S   0.3   0.2   0:00.41 crond
...output omitted... 
```
#### 列出进程
命令`ps`用于列出当前的进程，可以提供详细的进程信息，包括：
- 用户识别符(UID)，它确定进程的特权
- 唯一进程识别符(PID)
- CPU和已经花费的实时时间
- 进程在各种位置上分配的内存数量
- 进程stdout的位置，称为控制终端
- 当前的进程状态

&#8195;&#8195;`ps`命令中，`aux`是常见的选项集，可显示包括无控制终端的进程在内的所有进程。选项`lax`(长列表)提供更多技术详细信息，但可通过避免查询用户名来加快显示。相似的UNIX语法使用选项`-ef`来显示所有进程。示例：
```
[root@redhat8 ~]# ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          2  0.0  0.0      0     0 ?        S    May14   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        I<   May14   0:00 [rcu_gp]
...output omitted...
[root@redhat8 ~]# ps lax
F   UID    PID   PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
1     0      2      0  20   0      0     0 -      S    ?          0:00 [kthreadd]
1     0      3      2   0 -20      0     0 -      I<   ?          0:00 [rcu_gp]
...output omitted...
[root@redhat8 ~]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          2      0  0 May14 ?        00:00:00 [kthreadd]
root          3      2  0 May14 ?        00:00:00 [rcu_gp]
...output omitted...
```
### 控制作业
#### 描述作业和会话
&#8195;&#8195;作业控制是shell的一种功能，允许单个shell实例运行和管理多个命令。作业与在shell提示符中输入的每个管道相关联。该管道中的所有进程均是作业的一部分，并且是同一个进程组的成员：
- 如果在shell提示符处仅输入了一条命令，则这条命令可视为命令的最小“管道”，创建仅含有一个成员的一个作业
- 一次只能有一个作业从特定终端窗口中读取输入和键盘生成的信号：
    - 属于该作业的进程是该控制终端的前台进程
    - 该控制终端的后台进程是与该终端相关联的任何其他作业的成员。终端的后台进程无法从终端读取输入或接收键盘生成的中断，但可以写入终端。
    - 后台中的作业可能已停止（暂停），也可能正在运行。如果某个正在运行的后台作业尝试从终端读取内容，则该作业将自动暂停
- 每个终端是其自身的会话，并且可以具有一个前台进程和任意数量的独立后台进程。一个作业只能属于一个会话，也就是属于其控制终端的会话
- 命令`ps`在`TTY`列中显示进程的控制终端的设备名称。某些进程（如系统守护进程）由系统启动，并不是从shell提示符启动。这些进程没有控制终端，也不是作业的成员，并且无法转至前台。`ps`命令在`TTY`列中针对这些进程显示一个问号`?`

#### 在后台运行作业
&#8195;&#8195;任何命令或管道都可以在后台启动，在命令行的结尾处附加符号`&`即可。Bash shell显示作业编号（特定于会话的唯一编号）和新建子进程的PID。可以使用`jobs`命令显示Bash为特定会话跟踪的作业列表。示例：
```
[root@redhat8 ~]# sleep 60 &
[1] 17824
[root@redhat8 ~]# ps -ef |grep sleep
root      17823   1002  0 09:14 ?        00:00:00 sleep 60
root      17824   2947  0 09:14 pts/0    00:00:00 sleep 60
root      17826   2947  0 09:14 pts/0    00:00:00 grep --color=auto sleep
[root@redhat8 ~]# jobs
[1]+  Running                 sleep 60 &
```
&#8195;&#8195;如果利用`＆`符号将包含管道的命令行发送到后台，管道中最后一个命令的`PID`将用作输出。管道中的所有进程仍是该作业的成员。示例：
```
[root@redhat8 ~]# ls -l |sort|sleep 30 &
[1] 17869
[root@redhat8 ~]# jobs
[1]+  Running                 ls --color=auto -l | sort | sleep 30 &
[root@redhat8 ~]# ps -ef |grep sleep
root      17866   1002  0 09:17 ?        00:00:00 sleep 60
root      17869   2947  0 09:17 pts/0    00:00:00 sleep 30
root      17871   2947  0 09:17 pts/0    00:00:00 grep --color=auto sleep
```
&#8195;&#8195;可以使用`fg`命令和后台作业的`ID`（%作业编号）将该后台作业转至前台。要将前台进程发送到后台，首先在终端中按键盘生成的暂停请求`Ctrl+z`，该作业将立即置于后台并暂停。要启动在后台运行的已暂停进程，可使用`bg`命令。示例如下：
```
[root@redhat8 ~]# sleep 30 &
[1] 18033
[root@redhat8 ~]# fg %1
sleep 30
^Z
[1]+  Stopped                 sleep 30
[root@redhat8 ~]# bg %1
[1]+ sleep 30 &
[root@redhat8 ~]# jobs
[1]+  Running                 sleep 30 &
```
&#8195;&#8195;`ps j`命令显示与作业相关的信息。`PID`是进程的唯一进程ID。`PPID`是此进程的父进程（即启动（分叉）此进程的进程）的`PID`。`PGID`是进程组首进程的`PID`，通常是作业管道中的第一个进程。`SID`是会话首进程的`PID`，对于作业而言，这通常是正在其控制终端上运行的交互`shell`。示例`sleep`命令当前已暂停，因此其进程状态为`T`。示例：
```
[root@redhat8 ~]# sleep 30 &
[3] 18071
[root@redhat8 ~]# fg %3
sleep 30
^Z
[3]+  Stopped                 sleep 30
[root@redhat8 ~]# ps j
  PPID    PID   PGID    SID TTY       TPGID STAT   UID   TIME COMMAND
 18001  18009  18009   2885 pts/0     18080 S        0   0:00 -bash
 18009  18071  18071   2885 pts/0     18080 T        0   0:00 sleep 30
 18009  18080  18080   2885 pts/0     18080 R+       0   0:00 ps j
```
&#8195;&#8195;上面示例中`[1]`后面的`+`符号表示此作业是当前的默认作业。如果使用的某个命令需要参数`%作业编号`，但并没有提供作业编号，那么将对具有`+`指示符的作业执行该操作。
### 中断进程
#### 使用信号控制进程
&#8195;&#8195;信号是传递至进程的软件中断。信号向执行中的程序报告事件。生成信号的事件可以是错误或外部事件（I/O 请求或定时器过期），或者来自于显式使用信号发送命令或键盘序列。系统管理员用于日常进程管理的基本信号如下表所示：

信号编号|短名称|定义|用途
:---:|:---:|:---:|:---
1|HUP|挂起|用于报告终端控制进程的终止。也用于请求进程重新初始化(重新加载配置)而不终止
2|INT|键盘中断|导致程序终止。可以被拦截或处理。通过按INTR键序列(Ctrl+c)发送
3|QUIT|键盘退出|与SIGINT相似；在终止时添加进程转储。通过按QUIT键序列(Ctrl+\\)发送
9|KILL|中断，无法拦截|导致立即终止程序。无法被拦截、忽略或处理。对程序是致命的
15默认|TERM|终止|导致程序终止。可以被拦截、忽略或处理。友好的方式让程序终止；允许自我清理
18|CONT|继续|发送至进程使其恢复(若已停止)。无法被拦截。即使被处理，也始终恢复进程
19|STOP|停止，无法拦截|暂停进程。无法被拦截或处理
20|TSTP|键盘停止|和SIGSTOP不同，可以被拦截、忽略或处理。通过按SUSP键序列(Ctrl+z)发送

每个信号都有一个默认操作，通常是如下之一：
- Term：导致程序立即终止（退出）
- Core：导致程序保存内存镜像（核心转储），然后终止
- Stop：导致程序停止执行（暂停），再等待继续（恢复） 

&#8195;&#8195;向当前的前台进程发送信号，可以按下键盘控制序列以暂停`Ctrl+z`、中止`Ctrl+c`或核心转储`Ctrl+\`该进程。发送命令向后台进程或另一会话中的进程发送信号需使用信号。用户可以中断自己的进程，终止由其他人拥有的进程需要root特权。`kill`命令根据PID编号向进程发送信号。`kill -l`命令列出所有可用信号的名称和编号。示例如下：
```
[root@redhat8 ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
...output omitted...	
[root@redhat8 ~]# sleep 60 &
[1] 18361
[root@redhat8 ~]# sleep 120 &
[2] 18362
[root@redhat8 ~]# ps -ef |grep sleep
root      18361  18009  0 10:01 pts/0    00:00:00 sleep 60
root      18362  18009  0 10:01 pts/0    00:00:00 sleep 120
root      18364  18009  0 10:01 pts/0    00:00:00 grep --color=auto sleep
[root@redhat8 ~]# ps aux |grep sleep
root      18361  0.0  0.0   7284   732 pts/0    S    10:01   0:00 sleep 60
root      18362  0.0  0.0   7284   712 pts/0    S    10:01   0:00 sleep 120
root      18374  0.0  0.0  12112  1080 pts/0    S+   10:01   0:00 grep --color=auto sleep
[root@redhat8 ~]# kill 18361
[root@redhat8 ~]# ps -ef |grep sleep
root      18362  18009  0 10:01 pts/0    00:00:00 sleep 120
root      18377  18009  0 10:02 pts/0    00:00:00 grep --color=auto sleep
[1]-  Terminated              sleep 60
[root@redhat8 ~]# kill -SIGTERM 18362
[root@redhat8 ~]# ps -ef |grep sleep
root      18379  18009  0 10:02 pts/0    00:00:00 grep --color=auto sleep
[2]+  Terminated              sleep 120
```
&#8195;&#8195;命令`pkill`向一个或多个符合选择条件的进程发送信号。选择条件可以是命令名称、特定用户拥有的进程，或所有系统范围进程。`pkill`命令包括高级选择条件：
- 命令：具有模式匹配的命令名称的进程
- UID：由某一 Linux 用户帐户拥有的进程，无论是有效的还是真实的
- GID：由某一 Linux 组帐户拥有的进程，无论是有效的还是真实的
- 父级：特定父进程的子进程
- 终端：运行于特定控制终端的进程

示例如下：
```
[huang@redhat8 ~]$ sleep 120 &
[1] 18677
[huang@redhat8 ~]$ sleep 110 &
[2] 18678
[huang@redhat8 ~]$ exit
logout
[root@redhat8 ~]# ps -ef |grep sleep
huang     18677      1  0 10:29 pts/0    00:00:00 sleep 120
huang     18678      1  0 10:29 pts/0    00:00:00 sleep 110
root      18688  18009  0 10:29 pts/0    00:00:00 grep --color=auto sleep
[root@redhat8 ~]# pkill -U huang
[root@redhat8 ~]# ps -ef |grep sleep
root      18832  18809  0 10:30 pts/0    00:00:00 grep --color=auto sleep
```
#### 以管理员身份注销用户
出于各种各样的原因，用户可能需要注销其他用户。例如：
- 用户做出了安全违规行为
- 用户可能过度使用了资源
- 用户的系统可能不响应
- 用户不当访问了资料

&#8195;&#8195;要注销某个用户，首先确定要终止的登录会话。命令`w`列出用户登录和当前运行的进程。记录`TTY`和`FROM`列，以确定要关闭的会话。所有用户登录会话都与某个终端设备`TTY`相关联：
- 如果设备名称的格式为`pts/N`，说明这是一个与图形终端窗口或远程登录会话相关联的伪终端
- 如果格式为`ttyN`，则说明用户位于一个系统控制台、替代控制台或其他直接连接的终端设备上

&#8195;&#8195;首先使用`pgrep`确定要中断的`PID`编号，与`pkill`相似，使用相同的选项，但`pgrep`是列出进程而非中断进程。示例如下：
```
[root@redhat8 ~]# w
 10:35:53 up 11:30,  2 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
huang    pts/0    192.168.100.1    10:30    9.00s  0.02s  0.02s -bash
root     pts/1    192.168.100.1    10:32    0.00s  0.03s  0.01s w
[root@redhat8 ~]# pgrep -l -u huang
18727 systemd
18735 (sd-pam)
18741 pulseaudio
18742 sshd
18747 bash
18799 dbus-daemon
[root@redhat8 ~]# pkill -SIGKILL -u huang
[root@redhat8 ~]# pgrep -l -u huang
[root@redhat8 ~]# w
 10:40:40 up 11:35,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/1    192.168.100.1    10:32    0.00s  0.04s  0.00s w
```
&#8195;&#8195;当需要注意的进程在同一登录会话中时，可能不需要中断用户的所有进程。通过`w`命令确定会话的控制终端，然后仅中断引用同一终端`ID`的进程。除非指定了`SIGKILL`，否则会话首进程（此处为Bash登录shell）可以成功处理终止请求并保留，但所有其他会话进程将被终止。示例如下：
```
[root@redhat8 ~]# pgrep -l -u huang
19315 systemd
19322 (sd-pam)
19330 pulseaudio
19331 sshd
19337 bash
19388 dbus-daemon
19658 sleep
[root@redhat8 ~]# w -h -u huang
huang    pts/0    192.168.100.1    10:47   16.00s  0.06s  0.01s -bash
[root@redhat8 ~]# pkill -t pts/0
[root@redhat8 ~]# w -h -u huang
huang    pts/0    192.168.100.1    10:47   23.00s  0.04s  0.04s -bash
[root@redhat8 ~]# pgrep -l -u huang
19315 systemd
19322 (sd-pam)
19330 pulseaudio
19331 sshd
19337 bash
19388 dbus-daemon
[root@redhat8 ~]# pkill -SIGKILL -t pts/0
[root@redhat8 ~]# pgrep -l -u huang
[root@redhat8 ~]# 
```
示例说明：
- 在`pts/0`终端中，登录了`huang`用户，然后后台运行了`sleep`命令，切换到了`root`用户，同样后台运行了`sleep`命令
- 另外一个终端`pts/1`登录了`root`用户，示例中操作均在此终端执行
- 运行`pkill -t pts/0`时，`pts/0`终端中root用户退出到`huang`用户，并且`huang`用户的`sleep`进程终止了，但是会话还保持，`huang`用户还在登录
- 运行`pkill -SIGKILL -t pts/0`时，用户注销，会话结束

&#8195;&#8195;也可利用父进程和子进程关系来应用相同的选择性进程终止。使用`pstree`命令查看系统或单个用户的进程树。使用父进程的`PID`中断其创建的所有子进程。此时，父进程`Bash`登录 shell可以保留，因为信号仅定向至它的子进程。示例：
```
[root@redhat8 ~]# pstree -p huang
sshd(19809)───bash(19817)─┬─sleep(19880)
                          └─sleep(19881)
systemd(19789)─┬─(sd-pam)(19798)
               ├─dbus-daemon(19866)───{dbus-daemon}(19867)
               └─pulseaudio(19808)───{pulseaudio}(19865)
[root@redhat8 ~]# pkill -P 19817
[root@redhat8 ~]# pgrep -l -u huang
19789 systemd
19798 (sd-pam)
19808 pulseaudio
19809 sshd
19817 bash
19866 dbus-daemon
[root@redhat8 ~]# pstree -p huang
sshd(19809)───bash(19817)
systemd(19789)─┬─(sd-pam)(19798)
               ├─dbus-daemon(19866)───{dbus-daemon}(19867)
               └─pulseaudio(19808)───{pulseaudio}(19865)
```
### 监控进程活动
#### 负载平均值
&#8195;&#8195;负载平均值是Linux内核提供的一种度量方式，它可以简单地表示一段时间内感知的系统负载。可以用它来粗略衡量待处理的系统资源请求数量，并确定系统负载是随时间增加还是减少：
- 根据处于可运行和不可中断状态的进程数，内核会每五秒钟收集一次当前的负载数
- 通过报告CPU上准备运行的进程数以及等待磁盘或网络I/O完成的进程数，Linux可以确定负载平均值
- 负载数是准备运行的进程数（进程状态为R）或等待 I/O 完成的进程数（进程状态为D）的运行平均值
- 负载平均值可粗略衡量在可以执行其他任何作业之前，有多少进程当前在等待请求完成：
    - 请求可能是用于运行进程的CPU时间。
    - 或者，请求可能是让关键磁盘I/O操作完成；
    - 在请求完成之前，即使CPU空闲，也不能在CPU上运行该进程
    - 无论是哪种方式，都会影响系统负载；系统的运行看起来会变慢，因为有进程正在等待运行
- 一些UNIX系统仅考虑CPU使用率或运行队列长度来指示系统负载。Linux还包含磁盘或网络利用率，因为它们与CPU负载一样会对系统性能产生重大影响。遇到负载平均值很高但CPU活动很低时，请检查磁盘和网络活动

&#8195;&#8195;`uptime`命令是显示当前负载平均值的一种方法。它可显示当前时间、计算机启动时长、运行的用户会话数以及当前的负载平均值。后面三个负载平均值代表了最近1、5和15分钟的负载情况。很直观指出系统负载似乎在增高还是降低。示例如下：
```
[root@redhat8 ~]# uptime
 11:11:22 up 12:06,  2 users,  load average: 0.02, 0.01, 0.00
```
&#8195;&#8195;如果等待CPU处理的进程是负载平均值的主要贡献因素，则可以计算近似的每CPU负载值以判断系统是否在遭遇显著的等待。`lscpu`命令可以确定系统又多少CPU。`w`和`top`工具也可报告负载平均值。 示例：
```
[root@redhat8 ~]# lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              2
On-line CPU(s) list: 0,1
Thread(s) per core:  1
Core(s) per socket:  2
Socket(s):           1
NUMA node(s):        1
...output omitted...
[root@redhat8 ~]# w
 11:24:00 up 12:18,  2 users,  load average: 0.10, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
huang    pts/0    192.168.100.1    11:00   21:39   0.03s  0.03s -bash
root     pts/1    192.168.100.1    10:32    0.00s  0.14s  0.01s w
[root@redhat8 ~]# top
top - 11:24:22 up 12:19,  2 users,  load average: 0.07, 0.02, 0.00
Tasks: 275 total,   1 running, 274 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1806.1 total,    183.0 free,    523.7 used,   1099.4 buff/cache
MiB Swap:   2048.0 total,   1875.0 free,    173.0 used.   1247.6 avail Mem
...output omitted...
```
#### 实时进程监控
&#8195;&#8195;`top`程序是系统进程的动态视图，显示一个摘要标题，以及与`ps`信息类似的进程或线程列表。与`ps`输出不同，`top`以可配置的间隔持续刷新，而且也提供列重新排列、排序和突出显示功能。用户配置可以保存，变为永久。默认输出列：
- 进程ID(PID)
- 用户名称(USER)是进程所有者
- 虚拟内存(VIRT)是进程正在使用的所有内存，包括常驻集合、共享库，以及任何映射或交换的内存页（ps命令中为VSZ）
- 常驻内存(RES)是进程所用的物理内存，包括任何驻留的共享对象（ps命令中为RSS）
- 进程状态(S) 显示为：
    - D：不可中断睡眠
    - R：运行中或可运行
    - S：睡眠中
    - T：已停止或已跟踪
    - Z：僵停
- CPU时间(TIME)是进程启动以来总的处理时间。可以切换为包含所有过去子进程的累计时间
- 进程命令名称(COMMAND)

`top`中的基本击键操作如下表：

按键|用途
:---:|:---
?或h|交互式击键操作的帮助
l、t、m|切换到负载、线程和内存标题行
1|标题中切换显示单独CPU信息或所有CPU的汇总
s|更改刷新（屏幕）率，以带小数的秒数表示（如 0.5、1、5）。安全模式不可用
b|切换反向突出显示Running的进程；默认为仅粗体。
Shift+b|在显示中使用粗体，用于标题以及Running的进程
Shift+h|切换线程；显示进程摘要或单独线程
u、Shift+u|过滤任何用户名称（有效、真实）
Shift+m|按照内存使用率，以降序方式对进程列表排序
Shift+p|按照处理器使用率，以降序方式对进程列表排序
k|中断进程。若有提示，输入PID，再输入signal。安全模式不可用
r|调整进程的nice值。若有提示，输入PID，再输入nice_value。安全模式不可用
Shift+w|写入（保存）当前的显示配置，以便在下一次重新启动 top 时使用
q|退出，ctrl+c也可以
f|通过启用或禁用字段的方式来管理列。同时还允许用户为top设置排序字段
 
### 练习
#### 生成人为CPU负载
&#8195;&#8195;下面脚本会一直运行到被终止为止。它通过执行五万个加法题来生成人为CPU负载。然后休眠一秒钟，重置变量，再重复。脚本内容如下：
```shell
#!/bin/bash
while true; do
  var=1
  while [[ var -lt 50000 ]]; do
    var=$(($var+1))
  done
  sleep 1
done
```