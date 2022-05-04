# 文本及用户
## RHEL-文本基本操作
### 标准输入、标准输出及标准错误
&#8195;&#8195;一个运行的程序（或称为进程）需要从某位置读取输入并将输出写入某位置。从shell提示符运行的命令通常会从键盘读取其输入，并将输出发送到其终端窗口。进程使用称为文件描述符的编号通道来获取输入并发送输出。所有进程在开始时至少要有三个文件描述符。`标准输入`（通道 0）从键盘读取输入。`标准输出`（通道 1）将正常输出发送到终端。`标准错误`（通道 2）将错误消息发送到终端。如果程序打开连接至其他文件的单独连接，则可能要使用更大编号的文件描述符。

通道（文件描述符）:   
编号|通道名称|描述|默认连接|用法
:---:|:---:|:---:|:---:|:---:
0|stdin|标准输入|键盘|仅读取
1|stdout|标准输出|终端|仅写入
2|stderr|标准错误|终端|仅写入
3 +|filename|其他文件|无|读取和/或写入

### 将输出重定向到文件
输出重定向操作符:  
用法|说明
:---:|:---
\> file|重定向stdout以覆盖文件
\>> file|重定向stdout以附加到文件
2> file|重定向stderr以覆盖文件
2> /dev/null|将stderr错误消息重定向到/dev/null，从而将它丢弃
\> file 2>&1 or &> file|重定向stdout和stderr以覆盖同一个文件
\>> file  2>&1 or &>> file|重定向stdout和stderr以附加到同一个文件

重定向操作的顺序非常重要：
- `> file 2>&1`：将标准输出重定向到file，然后将标准错误作为标准输出重定向到相同位置 (file)
- `2>&1 > file`：相对上示例以相反的顺序执行重定向。这会将标准错误重定向到标准输出的默认位置（终端窗口，因此没有任何更改），然后仅将标准输出重定向到file

通过重定向，可以简化许多日常管理任务。示例：
```shell
### 保存时间错以供日后使用
date > /tmp/saved-timestamp
### 将日志文件的最后200行复制到另一文件
tail -n 200 /var/log/messages > /tmp/last-200-messages
### 将三个文件连接为一个
cat file1 file2 file3 > /tmp/all-three-in-one
### 将主目录的隐藏文件名和常规文件名列出到文件中
ls -a > /tmp/my-file-names
### 将输出附加到现有文件
echo "Miracles happen every day" >> /tmp/many-lines-of-info
diff previous-file current-file >> /tmp/tracking-changes-made
### 忽略并丢弃错误消息
find /etc -name passwd > /tmp/output 2> /dev/null
### 将输出和生成的错误消息存放在一起
find /etc -name passwd &> /tmp/save-both
### 将输出和生成的错误附加到现有文件
find /etc -name paswd >> /tmp/save-both 2>&1
```
将进程输出和错误消息保存到单独的文件中：
```
[huang@redhat8 ~]$ find /etc -name passwd > /tmp/output 2> /tmp/errors
[huang@redhat8 ~]$ cat /tmp/errors
find: ‘/etc/pki/rsyslog’: Permission denied
......
```
### 构建管道
&#8195;&#8195;管道是一个或多个命令的序列，用竖线字符`|`分隔。管道将第一个命令的标准输出连接到下一个命令的标准输入。示例如下：
```
[huang@redhat8 ~]$ ls -l /usr/bin |less
[huang@redhat8 ~]$ cat /etc/passwd |more
[huang@redhat8 ~]$ ls -l |wc -l
11
[huang@redhat8 ~]$ ls -l |head -n 5 > /tmp/five-last-files
``` 
### 管道、重定向和tee命令
&#8195;&#8195;当重定向与管道组合时，shell会首先设置整个管道，然后重定向输入/输出。如果在管道的中间使用了输出重定向，则输出将转至文件，而不是前往管道中的下一个命令。下面示例不会在终端上显示任何内容：
```
[huang@redhat8 ~]$ ls -al > /tmp/saved-output | less
```
&#8195;&#8195;`tee`命令克服了这个限制。在管道中，`tee`将其标准输入复制到其标准输出中，并且还将标准输出重定向到指定为命令参数的文件。示例将`ls`命令的输出重定向到文件，并且将输出传递到`less`以便在终端上以一次一屏的方式显示：
```
[huang@redhat8 ~]$ cat /etc/hosts |tee /tmp/saved-hosts |less
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
(END)
```
在管道末尾使用了`tee`，可以保存命令的最终输出并且同时输出到终端： 
```
[huang@redhat8 ~]$ ls -t |head -n 3 |tee /tmp/five-last-files
test
testfile
Desktop
```
&#8195;&#8195;可通过管道来重定向标准错误，但是不能使用合并重定向运算符（`&>`和`&>>`）执行此操作。以下是通过管道重定向标准输出和标准错误的正确方法：
```
[huang@redhat8 ~]$ find -name / passwd 2>&1 | less
```
### Vim
Vim通过单字符击键操作进入各个其他模式，访问特定的编辑功能：
- 按`i`键进入插入模式，其中键入的所有文本将变为文件内容
- `u`键可撤销最近的编辑
- 按`x`键可删除单个字符
- 按`Esc`键可退出插入模式，返回到命令模式
- 按`v`键进入可视模式，可在其中选择多个字符进行文本操作：
    - 字符模式：v
    - 行模式：Shift+v，可以选择多行
    - 块模式：Ctrl+v，可以选择文本块
- 按`:`键启动扩展命令模式，可以执行的任务包括写入文件（进行保存），以及退出Vim编辑器等：
    - `:w`命令可写入（保存）文件，并保留在命令模式中以进行更多编辑
    - `:wq`命令可写入（保存）文件并退出Vim
    - `:q!`命令可退出Vim，同时放弃上次写入以来进行的所有更改
- 在Vim 中，复制和粘贴称为拖拉和放置，使用的命令字符是`y`和`p`：
    - 首先将光标定位到要选择的第一个字符，然后进入可视模式
    - 使用箭头键扩展可视选择。准备好时，按`y`将所选内容拖拉到内存中
    - 将光标定位到新位置上，然后按`p`将所选内容放置到光标处

&#8195;&#8195;最初开发vi时，用户无法依赖箭头键或箭头键的键盘映射来移动光标。因此，vi设计了使用标准字符键的命令来移动光标：`H`、`J`、`K`和`L`。以下是记忆它们的一种方式：
- `H`：hang back
- `J`：jump down
- `K`：kick up
- `L`：leap forward

### 使用shell变量
&#8195;&#8195;Shell变量对于特定shell会话是唯一的。如果打开了两个终端窗口，或者通过两个独立的登录会话登录同一远程服务器，那么您在运行两个shell。每个shell都有自己的一组shell变量值。 使用以下语法将值分配给shell变量：
```shell
VARIABLENAME=value
```
&#8195;&#8195;变量名称可以包含大写或小写字母、数字和下划线字符`_`。使用`set命令列出当前设置的所有 shell 变量。（它还会列出所有shell函数，可以忽略它们）此列表足够长：
```
[root@redhat8 ~]# set |more
BASH=/bin/bash
BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extglob:extquote:force_fignore:histappend:int
eractive_comments:login_shell:progcomp:promptvars:sourcepath
BASHRCSOURCED=Y
BASH_ALIASES=()
BASH_ARGC=()
...output omitted...
```
&#8195;&#8195;使用变量扩展来指代已设置的变量值，需在变量名称前加上美元符号`$`。如果变量名称旁边有任何尾随字符，可能需要使用花括号来保护变量名称：
```
[root@redhat8 ~]# COUNT=100
[root@redhat8 ~]# echo $COUNT
100
[root@redhat8 ~]# echo Repeat $COUNTx
Repeat
[root@redhat8 ~]# echo Repeat ${COUNT}x
Repeat 100x
```
### 使用Shell变量配置Bash
&#8195;&#8195;一些shell变量在Bash启动时设置，但可以进行修改来调整shell的行为。例如，两个影响shell历史记录和`history`命令的变量是`HISTFILE`和`HISTFILESIZE`：
- 如果设置了`HISTFILE`，将指定文件的位置，以便在退出时保存shell历史记录
- `HISTFILE`默认情况下是用户的`~/.bash_history`文件
- `HISTFILESIZE`变量指定应将历史记录中的多少个命令保存到该文件中

&#8195;&#8195;另一个例子是`PS1`，这是一个控制shell提示符外观的shell变量。如果更改此值，它将改变shell提示符的外观，示例如下：
```
[root@redhat8 ~]$ PS1="mybash\$ "
mybash$ PS1="[\u@\h \W]# "
[root@redhat8 ~]# 
```
### 使用环境变量配置程序
&#8195;&#8195;shell提供了一个环境，供用户从该shell中运行程序。例如，此环境包括有关文件系统上当前工作目录的信息、传递给程序的命令行选项，以及环境变量的值。程序可以使用这些环境变量来更改其行为或其默认设置。
```
[root@redhat8 ~]# EDITOR=vim
[root@redhat8 ~]# export EDITOR
[root@redhat8 ~]# export EDITOR=vim
```
&#8195;&#8195;应用程序和会话使用这些变量来确定其行为。例如，shell启动时自动将`HOME`变量设置为用户主目录的文件名。`LANG`变量可以设定的是区域设置。这会调整程序输出的首选语言，字符集，日期、数字和货币的格式，以及程序的排序顺序：
```
[root@redhat8 ~]# export export LANG=en_US.UTF-8
[root@redhat8 ~]# date
Wed May  4 02:28:24 EDT 2022
```
&#8195;&#8195;`PATH`变量包含一个含有程序的目录的冒号分隔列表。运行`ls`等命令时，shell会按照顺序逐一在这些目录中查找可执行文件`ls`，并且运行找到的第一个匹配文件。可以将其他目录添加到`PATH`的末尾。例如，在`/home/huang/sbin`中可能具有像常规命令一样运行的可执行程序或脚本：
```
[root@redhat8 ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@redhat8 ~]# export PATH=${PATH}:/home/huang/sbin
[root@redhat8 ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/home/huang/sbin
```
要列出特定`shell`的所有环境变量，运行`env`命令：
```
[root@redhat8 ~]# env
HISTCONTROL=ignoredups
DISPLAY=localhost:10.0
HOSTNAME=redhat8
EDITOR=vim
...output omitted...
```
&#8195;&#8195;`EDITOR`环境变量指定要用作命令行程序的默认文本编辑器的程序。如果不指定，很多程序都使用`vi`或`vim`，也可以根据需要自定义设置：
```
[root@redhat8 ~]# export EDITOR=nano
[root@redhat8 ~]# echo $EDITOR
nano
```
### 自动设置变量
&#8195;&#8195;如果希望在shell启动时自动设置shell或环境变量，可以编辑Bash启动脚本。运行的确切脚本取决于 shell的启动方式，是交互式登录shell、交互式非登录shell 还是shell脚本。假设是默认的`/etc/profile`、`/etc/bashrc`和`~/.bash_profile`文件，如要更改启动时影响所有交互式shell提示符的用户帐户，编辑`~/.bashrc`文件。例如，可以通过编辑要读取的文件，将该帐户的默认编辑器设置为`nano`：
```
[root@redhat8 ~]# vi ~/.bashrc
[root@redhat8 ~]# cat ~/.bashrc
# .bashrc
# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
export EDITOR=nano
```
&#8195;&#8195;调整影响所有用户帐户的设置的最佳方式是添加名称以`.sh`结尾的文件，并在该文件中包含对`/etc/profile.d`目录的更改，并需要以root用户身份登录。
### 取消设置和取消导出变量
&#8195;&#8195;要完全取消设置和取消导出变量，使用`unset`命令。要取消导出变量但不取消设置它，使用`export -n`命令。示例如下：
```
[root@redhat8 ~]# echo $fileX
/tmp/testdir/fileX
[root@redhat8 ~]# unset fileX
[root@redhat8 ~]# echo $fileX

[root@redhat8 ~]# export -n PS1
```
## 管理本地用户和组
### 用户类型
用户帐户有三种主要类型：超级用户、系统用户和普通用户：
- 超级用户帐户用于管理系统。超级用户的名称为root，其帐户UID为0。超级用户对系统具有完全访问权限
- 系统的系统用户帐户供提供支持服务进程使用。这些进程（或守护进程）通常不需要以超级用户身份运行。系统会为它们分配非特权帐户，允许它们确保其文件和其他资源不受彼此以及系统上普通用户的影响。用户无法使用系统用户帐户以交互方式登录
- 大多数用户都有用于处理日常工作的普通用户帐户。与系统用户一样，普通用户对系统具有有限的访问权限

&#8195;&#8195;`id`命令显示有关当前已登录用户的信息，也可以将用户名作为参数传递给 id 命令，示例如下：
```
[root@redhat8 ~]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@redhat8 ~]# id huang
uid=1000(huang) gid=1000(huang) groups=1000(huang),40001(Archivers)
```
&#8195;&#8195;使用`ps`命令查看进程信息，默认为仅显示当前shell中的进程。使用`a`选项可查看与某一终端相关的所有进程。使用`u `选项查看与进程相关联的用户。示例如下：
```
[root@redhat8 ~]# ps -au
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
gdm        1777  0.0  2.7 498652 50240 tty1     Sl   01:45   0:00 /usr/libexec/
huang      2443  0.0  0.2  26432  5056 pts/0    Ss   01:46   0:00 -bash
root       4698  0.0  0.2  57172  3880 pts/0    R+   05:33   0:00 ps -au
```
&#8195;&#8195;以上命令的输出按名称显示用户，但操作系统内部利用UID来跟踪用户。用户名到UID的映射在帐户信息数据库中定义。默认情况下，系统使用`/etc/passwd`文件存储有关本地用户的信息。它分为七个以冒号分隔的字段。以下是`/etc/passwd`中某一用户的示例：
```
[root@redhat8 ~]# cat /etc/passwd
huang:x:1000:1000:huang:/home/huang:/bin/bash
```
示例中字段依次说明：
- 该用户`huang`的用户名
- 用户的密码曾以加密格式存储在此处。现在它已移至`/etc/shadow`文件，该字段始终应为`x`
- 该用户帐户的UID号`1000`
- 该用户帐户的主要组的GID号`1000`
- 该用户的真实姓名
- 该用户的主目录`/home/huang`。这是shell启动时的初始工作目录，其中包含有用户数据和配置设置
- 该用户的默认shell程序，会在登录时运行`/bin/bash`。如果系统用户不允许进行交互式登录，该用户可能会使用`/sbin/nologin`

### 组Group
&#8195;&#8195;组是需要共享文件和其他系统资源访问权限的用户的集合。组可用于向一组用户授予文件访问权限，而非仅仅向一个用户授予访问权限。系统通过分配的唯一标识号`GID`来区分不同的组。默认情况下，系统使用`/etc/group`文件存储有关本地组的信息。每个组条目被分为四个以冒号分隔的字段。以下是`/etc/group`中某一行的示例：
```
[root@redhat8 ~]# cat /etc/group
ftpusers:x:40000:christ,harry
```
实例中字段依次说明：
- 该组的组名称`ftpusers`
- 过时的组密码字段。该字段始终应为`x`
- 该组的GID号`40000`
- 该组成员的用户列表`christ,harry`

主要组和补充组：
- 每个用户有且只有一个主要组。对于本地用户而言，这是按`/etc/passwd`文件中的GID号列出的组。默认情况下，这是拥有用户创建的新文件的组。通常，在创建新的普通用户时，会创建一个与该用户同名的新组。该组将用作新用户的主要组，而该用户是这一用户专用组的唯一成员。这有助于简化文件权限的管理
- 用户也可以有补充组。补充组的成员资格由`/etc/group`文件确定。根据所在的组是否具有访问权限，将授予用户对文件的访问权限。具有访问权限的组是用户的主要组还是补充组无关紧要

### 超级用户
&#8195;&#8195;大多数操作系统具有某种类型的超级用户，即具有系统全部权限的用户。在红帽企业Linux中，此为`root` 用户。该用户的特权高于文件系统上的一般特权，用于管理系统。要执行诸如安装或删除软件以及管理系统文件和目录等任务，必须将特权升级到`root`用户。 
### 切换用户
&#8195;&#8195;`su`命令可让用户切换至另一个用户帐户。如果从普通用户帐户运行`su`，系统会提示您输入要切换的帐户的密码。当以 root用户身份运行`su`时，则无需输入用户密码。如果省略用户名，则默认情况下`su`或`su -`命令会尝试切换到root。示例如下：
```
[christ@redhat8 ~]$ su - huang
Password: 
[huang@redhat8 ~]$ su -
Password: 
[root@redhat8 ~]# 
```
&#8195;&#8195;命令`su`将启动非登录shell，而命令`su -`（带有短划线选项）会启动登录shell。两个命令的主要区别在于，`su -`会将shell环境设置为如同以该用户身份重新登录一样，而`su`仅以该用户身份启动shell，但使用的是原始用户的环境设置。`su`命令最常用于获得以另一个用户身份（通常是root）运行的命令行界面（shell提示符）。如果结合`-c`选项，该命令的作用将与Windows实用程序runas一样，能够以另一个用户身份运行任意程序。可以运行`info su`来查看更多详情。 
### 通过Sudo运行命令
&#8195;&#8195;有时为安全起见，root用户的帐户可能根本没有有效的密码。在这种情况下，用户无法使用密码直接以 root身份登录系统，也不能使用`su`获取交互式shell。可以用`sudo`获取root访问权限：
- 与`su`不同，`sudo`通常要求用户输入其自己的密码以进行身份验证
- `sudo`可以配置为允许特定用户像某个其他用户一样运行任何命令，或仅允许以该用户身份运行部分命令

&#8195;&#8195;例如，如果sudo已配置为允许`huang`用户以root身份运行命令`usermod`，那么就可运行以下命令来锁定或解锁用户帐户： 
```
[huang@redhat8 ~]$ sudo usermod -L christ
[sudo] password for huang: 
huang is not in the sudoers file.  This incident will be reported.
```
&#8195;&#8195;上面示例中，sudo配置不允许，命令被阻止，这次尝试也会被记录下来，并且默认情况下还会向root用户发送一封电子邮件。使用sudo的另一个优点在于，执行的所有命令都默认为将日志记录到`/var/log/secure`中。示例：
```
[root@redhat8 ~]# sudo tail /var/log/secure
sliceMay  4 09:12:07 redhat8 su[6852]: pam_unix(su-l:session): session opened for user huang by huang(uid=0)
May  4 09:12:20 redhat8 sudo[6879]: huang : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/huang ; USER=root ; COMMAND=/sbin
```
&#8195;&#8195;在红帽企业Linux 7和红帽企业Linux 8中，wheel组的所有成员都可以使用sudo以任何用户身份运行命令，包括 root 在内。系统将提示用户输入其自己的密码。
### 通过Sudo获取交互式Root Shell
&#8195;&#8195;如果系统上的非管理员用户帐户能够使用`sudo`来运行`su`命令，则可以从该帐户运行`sudo su -`来获取root用户的交互式shell。这是因为sudo将以root用户身份运行`su -`，而root用户无需输入密码即可使用su。
通过sudo访问root帐户的另一种方式是使用`sudo -i`命令。这将切换到root帐户并运行该用户的默认shell（通常为 bash）及关联的shell登录脚本。如果只想运行shell，可使用`sudo -s`命令。`sudo su -`命令与`sudo -i`的行为不完全相同。
### 配置sudo
&#8195;&#8195;sudo的主配置文件为`/etc/sudoers`。如果多个管理员试图同时编辑该文件，为了避免出现问题，只应使用特殊的`visudo`命令进行编辑：
```
[root@redhat8 ~]# cat /etc/sudoers
## Allow root to run any commands anywhere 
root	ALL=(ALL) 	ALL
## Allows people in group wheel to run all commands
%wheel	ALL=(ALL)	ALL
```
&#8195;&#8195;上面示例中，`%wheel`是规则适用的用户或组。`%`指定这是一个组，即组`wheel`。`ALL=(ALL)`指定在可能包含此文件的任何主机上，`wheel`可以运行任何命令。最后的`ALL`指定`wheel`可以像系统上的任何用户一样运行这些命令。    
&#8195;&#8195;默认情况下，`/etc/sudoers`还包含`/etc/sudoers.d`目录中所有文件的内容，作为配置文件的一部分。管理员只需将相应的文件放入`/etc/sudoers.d`目录中，即可为用户添加sudo访问权限。将文件复制到目录中或从目录中删除文件，即可启用或禁用sudo访问权限。示例如下：
```
[root@redhat8 ~]# touch /etc/sudoers.d/huang
[root@redhat8 ~]# vi /etc/sudoers.d/huang
[root@redhat8 ~]# cat /etc/sudoers.d/huang
huang ALL=(ALL)  ALL
[root@redhat8 ~]# cat /etc/sudoers.d/ftpusers
%ftpusers ALL=(ALL)  ALL
[root@redhat8 ~]# vi /etc/sudoers.d/ansible
[root@redhat8 ~]# cat /etc/sudoers.d/ansible
ansible  ALL=(ALL)  NOPASSWD:ALL
```
示例说明：
- 第一个示例为用户`huang`启用完整的sudo访问权限
- 第二个示例为组`ftpusers`启用完整的sudo访问权限
- 第三个示例设置sudo，允许`ansible`用户在不输入密码的前提下以其他用户身份运行命令

### sudo中两个命令区别
`sudo su -`和`sudo -i`这两个命令之间存在一些细微差别：
- `sudo su -`命令可以完全像正常登录那样设置root环境，因为`su -`命令会忽略sudo所做的设置并从头开始设置环境
- `sudo -i`命令的默认配置实际上会设置在一些细节上与正常登录不同的root用户环境。例如，它设置的PATH环境变量便略有不同，这会影响shell查找命令的位置

通过用`visudo`编辑`/etc/sudoers`，可以让`sudo -i`的行为与`su -`更为相似。找到行：
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```
替换为以下两行：
```
Defaults      secure_path = /usr/local/bin:/usr/bin
Defaults>root secure_path = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```
### 




