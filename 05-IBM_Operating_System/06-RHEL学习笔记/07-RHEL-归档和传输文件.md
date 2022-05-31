# RHEL-归档和传输文件
## 管理压缩的tar存档
### tar命令
&#8195;&#8195;创建备份和通过网络传输数据时，归档和压缩文件非常有用。用来创建和使用备份存档的其中一个最早也是最常见的命令是`tar`命令：
- 通过`tar`命令，用户可以将大型文件集汇集为一个文件（存档）
- `tar`存档是一个结构化的文件数据序列，其中包含有关每个文件和索引的元数据，以便可以提取单个文件
- 该存档可以使用`gzip`、`bzip2`或`xz`压缩方式进行压缩
- `tar`命令能够列出存档内容，或者将其文件提取到当前系统

### 所选的tar选项
&#8195;&#8195;`tar`命令选项划分成不同的操作，其中包括一般选项和压缩选项。下面表格显示了常用选项、选项的长版本及其说明。   
`tar`操作概述：

选项|描述
:---|:---
-c、--create|创建一个新存档
-x、--extract|从现有存档提取
-t、--list|列出存档的目录

所选的`tar`一般选项：

选项|描述
:---|:---
-v、--verbose|详细信息。显示存档或提取的文件有哪些
-f、--file=|文件名。此选项必须后接要使用或创建的存档的文件名
-p、--preserve-permissions|在提取存档时保留文件和目录的权限，而不去除umask

`tar`压缩选项概述：

选项|描述
:---|:---
-z、--gzip|使用gzip压缩方式(.tar.gz)
-j、--bzip2|使用bzip2压缩方式(.tar.bz2)。bzip2的压缩率通常比gzip高
-J、--xz|使用xz压缩方式(.tar.xz)。xz的压缩率通常比bzip2更高

### 归档文件和目录
&#8195;&#8195;创建新存档时要使用的第一个选项为`c`选项，后跟`f`选项，接着是一个空格，然后是要创建的存档的文件名，最后是应当添加到该存档中的文件和目录列表。存档会创建在当前目录中，除非另外指定。示例如下：
```
[root@redhat8 testdir]# tar -cf test.tar testfile1 testfile2
[root@redhat8 testdir]# ls -l test.tar
-rw-r--r--. 1 root root 10240 May 30 06:29 test.tar
```
以上示例`tar`命令也可以使用长版本选项执行：
```
[root@redhat8 testdir]# tar -cf --file=test.tar --create testfile1 testfile2
[root@redhat8 testdir]# ls -l test.tar
-rw-r--r--. 1 root root 10240 May 30 06:29 test.tar
```
注意事项：
- 在创建`tar`存档之前，请先验证目录中没有其他存档与要创建的新存档名称相同。`tar`命令将覆盖现有的存档，而不提供警告
- 在使用绝对路径名归档文件时，将默认从文件名中删除该路径中的前面的`/`符号。删除路径中的前导`/`可帮助用户在提取存档时避免覆盖重要文件

执行`tar`命令的用户必须要可以读取这些文件：
- 为`/etc`文件夹及其所有内容创建新存档需要root特权，只有`root`用户才可读取`/etc`目录的所有文件
- 非特权用户可以创建`/etc`目录的存档，但是该存档将忽略用户没有读取权限的文件，并且将忽略用户没有读取和执行权限的目录

示例以root身份创建名为`/root/etc.tar`且内容为`/etc`目录的tar存档：
```
[root@redhat8 ~]# tar -cf /root/etc.tar /etc
tar: Removing leading `/' from member names
[root@redhat8 ~]# ls -l etc.tar
-rw-r--r--. 1 root root 27381760 May 30 06:37 etc.tar
```
### 列出存档的内容
`t`选项指示列出存档的内容。使用`f`选项加上要查询的存档的名称。示例：
```
[root@redhat8 ~]# tar -tf etc.tar |more
etc/
etc/mtab
etc/fstab
etc/crypttab
etc/resolv.conf
etc/dnf/
etc/dnf/modules.d/
etc/dnf/modules.d/container-tools.module
...output omitted...
```
### 从存档中提取文件
从文档中提取文件注意事项：
- `tar`存档通常应当提取到空目录中，以确保它不会覆盖任何现有的文件
- 当root提取存档时，`tar`命令会保留文件的原始用户和组所有权
- 如果普通用户使用`tar`提取文件，文件所有权将属于从存档中提取文件的用户

示例将`/root/etc.tar`存档中的文件恢复到`/root/etcbackup`目录：
```
[root@redhat8 ~]# mkdir /root/etcbackup
[root@redhat8 ~]# cd /root/etcbackup
[root@redhat8 etcbackup]# tar -xf /root/etc.tar
[root@redhat8 etcbackup]# ls -l
total 12
drwxr-xr-x. 137 root root 8192 May 29 16:23 etc
```
&#8195;&#8195;默认情况下，从存档中提取文件时，将从存档内容的权限中去除`umask`。要保留存档文件的权限，可在提取存档时使用`p`选项。示例将存档`/root/scripts.tar`提取到`/root/scripts`目录，同时保留所提取文件的权限：
```
[huang@redhat8 ~]$ ls -l ~/scripts/test.sh
-rw-rw----. 1 huang huang 0 May 30 06:49 /home/huang/scripts/test.sh
[huang@redhat8 ~]$ tar -cf /root/scripts.tar /home/huang/scripts/test.sh
tar: Removing leading `/' from member names
[huang@redhat8 ~]$ ls -l /root/scripts.tar
-rw-rw----. 1 huang huang 10240 May 30 06:51 /root/scripts.tar
[huang@redhat8 ~]$ exit
logout
[root@redhat8 ~]# cd scripts
[root@redhat8 scripts]# tar -xpf /root/scripts.tar
[root@redhat8 scripts]# ls -l ./home/huang/scripts/test.sh
-rw-rw----. 1 huang huang 0 May 30 06:49 ./home/huang/scripts/test.sh
```
### 创建压缩存档
`tar`命令支持三种压缩方式：
- `gzip`压缩速度最快，历史最久，使用也最为广泛，能够跨发行版甚至跨平台使用
    - `-z`或`--gzip`进行`gzip`压缩（filename.tar.gz 或 filename.tgz） 
- `bzip2`压缩创建的存档文件通常比`gzip`创建的文件小，但可用性不如`gzip`广泛
    - `-j`或`--bzip2`进行 bzip2 压缩 (filename.tar.bz2) 
- `xz`压缩方式相对较新，但通常提供可用方式中最佳的压缩率

创建三种格式的压缩文件，内容都来自`/etc`目录：
```
[root@redhat8 tartest]# tar -czf /root/etcbackup.tar.gz /etc
tar: Removing leading `/' from member names
[root@redhat8 tartest]# tar -cjf /root/etcbackup.tar.bz2 /etc
tar: Removing leading `/' from member names
[root@redhat8 tartest]# tar -cJf /root/etcbackup.tar.xz /etc
tar: Removing leading `/' from member names
[root@redhat8 ~]# ls -l |grep etcbackup
drwxr-xr-x. 3 root   root        17 May 30 06:44 etcbackup
-rw-r--r--. 1 root   root   4800745 May 30 07:16 etcbackup.tar.bz2
-rw-r--r--. 1 root   root   6398813 May 30 07:11 etcbackup.tar.gz
-rw-r--r--. 1 root   root   4045228 May 30 07:17 etcbackup.tar.xz
```
&#8195;&#8195;创建存档后，使用`tf`选项来验证存档的内容。在列出压缩存档文件的内容时，不必强制使用压缩代理选项。例如列出`/root/etcbackup.tar.gz`文件中存档的内容：
```
[root@redhat8 ~]# tar -tf /root/etcbackup.tar.gz |more
etc/
etc/mtab
etc/fstab
etc/crypttab
etc/resolv.conf
etc/dnf/
etc/dnf/modules.d/
etc/dnf/modules.d/container-tools.module
...output omitted...
```
### 提取压缩的存档
&#8195;&#8195;提取压缩的`tar`存档时，要执行的第一步是决定存档文件应提取到的位置，然后创建并更改到目标目录。`tar`命令会判断之前使用的压缩方式。也可以在`tar`命令中添加解压缩方式，必须使用正确的解压缩类型选项；否则，`tar`会生成错误来指出选项中指定的解压缩类型与文件的解压缩类型不匹配。示例：
```
[root@redhat8 etcbackup]# tar -xjf /root/etcbackup.tar.gz |more
bzip2: (stdin) is not a bzip2 file.
tar: Child returned status 2
tar: Error is not recoverable: exiting now
```
解压三种类型对应压缩文件到`/tmp/etcbackup`目录： 
```
[root@redhat8 ~]# mkdir /tmp/etcbackup
[root@redhat8 ~]# cd /tmp/etcbackup
[root@redhat8 etcbackup]# tar -xvf /root/etcbackup.tar.gz
etc/
etc/mtab
etc/fstab
etc/crypttab
etc/resolv.conf
...output omitted...
[root@redhat8 etcbackup]# tar -xjf /root/etcbackup.tar.bz2 |more
[root@redhat8 etcbackup]# tar -xJf /root/etcbackup.tar.xz |more
```
注意事项及更多说明：
- 列出压缩的`tar`存档的工作方式与列出未压缩的`tar`存档相同
- `gzip`、`bzip2`和`xz`也可单独用于压缩单个文件。例如：
    - `gzip etc.tar` 命令将生成`etc.tar.gz`压缩文件
    - `bzip2 abc.tar`命令将生成`abc.tar.bz2`压缩文件
    - `xz myarchive.tar`命令则生成`myarchive.tar.xz`压缩文件
- 对应的解压缩命令为`gunzip`、`bunzip2`和`unxz`。例如：
    - `gunzip /tmp/etc.tar.gz`命令将生成`etc.tar`解压缩`tar`文件
    - `bunzip2 abc.tar.bz2`命令将生成`abc.tar`解压缩`tar`文件
    - `unxz myarchive.tar.xz`命令则生成`myarchive.tar`解压缩`tar`文件

## 在系统之间安全地传输文件
### 使用Secure Copy传输文件
&#8195;&#8195;Secure Copy命令`scp`是OpenSSH套件的一部分，可将文件从远程系统复制到本地系统或从本地系统复制到远程系统。此命令利用`SSH`服务进行身份验证，并在数据传输之前对其进行加密。    
&#8195;&#8195;用户可以为所要复制的文件的源或目标指定一个远程位置。远程位置的格式为 `[user@]host:/path`。命令说明如下：
- 该参数的`user@`部分是可选的，如果不指定该部分，则使用用户当前的本地用户名
- 运行此命令时，`scp`客户端将使用基于密钥的身份验证或以提示用户输入密码的方式向远程`SSH`服务器进行身份验证，像`ssh`一样

&#8195;&#8195;示例将本地文件`/root/etcbackup.tar.gz`复制到`192.168.100.131`远程系统的`/home/huang`目录，并指定用户为`huang`：
```
[root@redhat8 ~]# scp /root/etcbackup.tar.gz huang@192.168.100.131:/home/huang
huang@192.168.100.131's password: 
etcbackup.tar.gz                                  100% 6249KB 205.8MB/s   00:00  
```
&#8195;&#8195;还可以沿另一个方向复制文件，即从远程系统复制到本地文件系统。示例`192.168.100.131`上的文件`/home/huang/umask.test`复制到本地目录`/root`：
```
[root@redhat8 ~]# scp huang@192.168.100.131:/home/huang/umask.test /root
huang@192.168.100.131's password: 
umask.test                                        100%    0     0.0KB/s   00:00    
[root@redhat8 ~]# ls -l umask.test
-rwx---r-x. 1 root root 0 May 30 10:36 umask.test
```
&#8195;&#8195;要以递归方式复制整个目录树，可使用`-r`选项。示例`192.168.100.131`上的远程目录`/home/huang/test`以递归方式复制到`host`上的本地目录`/tmp/`：
```
[root@redhat8 ~]# scp -r huang@192.168.100.131:/home/huang/test /tmp
huang@192.168.100.131's password: 
cltopinfo                                         100%  274    56.3KB/s   00:00      
clshowsrv                                         100%  846     1.0MB/s   00:00       
cldump                                            100% 1486     1.3MB/s   00:00        
df.log                                            100%  539   481.8KB/s   00:00      
```
### 使用安全文件传输程序传输文件
&#8195;&#8195;以交互方式从`SSH`服务器上传或下载文件，使用安全文件传输程序`sftp`。`sftp`命令的会话使用安全身份验证机制，并将数据加密后再与`SSH`服务器来回传输。与`scp`命令一样，`sftp`使用`[user@]host`来标识目标系统和用户名。如果未指定用户，该命令将尝试使用本地用户名作为远程用户名进行登录。随后会显示`sftp>`提示符。示例如下：
```
[root@redhat8 ~]# sftp huang@192.168.100.131
huang@192.168.100.131's password: 
Connected to huang@192.168.100.131.
sftp> quit
[root@redhat8 ~]# 
```
&#8195;&#8195;交互式`sftp`会话接受各种命令，这些命令在远程文件系统上运行的方式与在本地文件系统上相同，如`ls`、`cd`、`mkdir`、`rmdir`和`pwd`。传输相关常用命令：
- `put`命令将文件上载到远程系统
- `get`命令从远程系统下载文件
- `exit`命令可退出`sftp`会话

&#8195;&#8195;示例将本地系统上的`/root/scripts.tar`文件上传到`192.168.100.131`上新建的目录 `/home/huang/hostbackup`。`sftp`会话始终假设`put`命令后跟的是本地文件系统上的文件，并且首先连接用户的主目录（此例中为`/home/huang`）：
```
[root@redhat8 ~]# sftp huang@192.168.100.131
huang@192.168.100.131's password: 
Connected to huang@192.168.100.131.
sftp> mkdir hostbackup
sftp> cd hostbackup
sftp> put /root/scripts.tar
Uploading /root/scripts.tar to /home/huang/hostbackup/scripts.tar
/root/scripts.tar                                 100%   10KB   1.1MB/s   00:00 
```
&#8195;&#8195;示例从远程主机下载`/etc/yum.conf`到本地系统上的当前目录，可执行 get /etc/yum.conf 命令，然后使用 exit 命令退出 sftp 会话：
```
huang@192.168.100.131's password: 
Connected to huang@192.168.100.131.
sftp> get /etc/yum.conf
Fetching /etc/yum.conf to yum.conf
/etc/yum.conf                                     100%   82    48.7KB/s   00:00    
sftp> exit
[root@redhat8 ~]# ls -l yum.conf
-rw-r--r--. 1 root root 82 May 30 11:05 yum.conf
```
## 在系统间安全地同步文件
### 使用rsync同步文件和目录
&#8195;&#8195;`rsync`命令是在系统之间安全复制文件的另一种方式。此工具采用的算法可通过仅同步已更改的文件部分来将复制的数据量最小化：
- 它与`scp`的区别在于，如果两个服务器间的两个文件或目录相似，`rsync`将仅复制文件系统间的差异部分，而`scp`仍复制所有内容。
- `rsync`的优点是它能够在本地系统和远程系统之间安全而高效地复制文件。虽然首次目录同步的用时与复制操作大致相同，但之后的同步只需通过网络复制差异部分，从而会大幅加快更新的速度

命令`rsync`选项说明：
- `rsync`的一个重要选项是`-n`选项，它用于执行空运行：
    - 空运行是对执行命令时所发生情况的模拟
    - 空运行显示了在命令正常运行时 `rsync`所要进行的更改。
    - 在进行实际`rsync`操作前先执行空运行，以确保重要的文件不会被覆盖或删除
- 使用`rsync`进行同步时，两个常用的选项为`-v`和`-a`选项：
    - `-v`或`--verbose`选项可提供更详细的输出。这对于故障排除和查看实时进度非常有用
    - `-a`或`--archive`选项将启用“存档模式”。这样可实现递归复制并开启很多有用的选项，以保留文件的大部分特征
- 存档模式不会保留硬链接，因为这会大幅增加同步时间。如果想保留硬链接，使用`-H`选项
-  如果使用的是高级权限，则可能还需要另外两个选项：
    - `-A`用于保留ACL
    - `-X`用于保留SELinux上下文 

存档模式与指定以下选项的作用相同，通过`rsync -a`启用的选项： 

选项|描述
:---|:---
-r、--recursive|以递归方式同步整个目录树
-l、--links|同步符号链接
-p、--perms|保留权限
-t、--times|保留时间戳
-g、--group|保留组所有权
-o、--owner|保留文件所有者
-D、--devices|同步设备文件

&#8195;&#8195;可以使用`rsync`将本地文件或目录的内容与远程计算机上的文件或目录进行同步（使用任一计算机作为源）。也可以同步两个本地文件或目录的内容。例如让`/home/huang/hostbackup`目录的内容与`/home/huang/rsynctmp`目录保持同步：
```
[root@redhat8 ~]# ls /home/huang/hostbackup
scripts.tar
[root@redhat8 ~]# ls /home/huang/rsynctmp
[root@redhat8 ~]# rsync -av /home/huang/hostbackup /home/huang/rsynctmp
sending incremental file list
hostbackup/
hostbackup/scripts.tar

sent 10,389 bytes  received 39 bytes  20,856.00 bytes/sec
total size is 10,240  speedup is 0.98
[root@redhat8 ~]# ls /home/huang/rsynctmp
hostbackup
[root@redhat8 ~]# cd /home/huang/rsynctmp/hostbackup
[root@redhat8 hostbackup]# ls
scripts.tar
```
&#8195;&#8195;通过为源目录加上尾随斜杠，可以同步该目录的内容，而不在目标目录中新建子目录。在下面示例，`hostbackup`目录不是在`/home/huang/rsynctmp`目录中创建的，仅`/home/huang/hostbackup`的内容同步：
```
[root@redhat8 ~]# rsync -av /home/huang/hostbackup/ /home/huang/rsynctmp
sending incremental file list
./
scripts.tar

sent 10,370 bytes  received 38 bytes  20,816.00 bytes/sec
total size is 10,240  speedup is 0.98
[root@redhat8 ~]# ls /home/huang/rsynctmp
scripts.tar
```
注意事项：
- 为`rsync`命令键入源目录时，目录名称中是否存在尾随斜杠至关重要。它将决定同步到目标中的是目录还是仅目录中的内容
- `Tab`补全可自动在目录名称中添加尾随斜杠

&#8195;&#8195;同`scp`和`sftp`命令一样，`rsync`使用`[user@]host:/path`格式来指定远程位置。远程位置可以是源或目标系统，但两台计算机中的一台必须是本地计算机：
- 要保留文件所有权，需要是目标系统上的root用户
- 如果目标为远程目标，则以root身份进行验证
- 如果目标为本地目标，则必须以root身份运行`rsync`

&#8195;&#8195;示例将本地的`/home/huang/hostbackup`目录同步到`192.168.100.131`远程系统上的`/home/huang/rsynctest`目录：
```
[root@redhat8 ~]# rsync -av /home/huang/hostbackup \
192.168.100.131:/home/huang/rsync
testroot@192.168.100.131's password: 
sending incremental file list
created directory /home/huang/rsynctest
hostbackup/
hostbackup/scripts.tar

sent 10,389 bytes  received 83 bytes  2,327.11 bytes/sec
total size is 10,240  speedup is 0.98
```
&#8195;&#8195;同样，`192.168.100.131`远程系统上的`/home/huang/rsynctest`远程目录也可同步到host上的`/home/huang/hostbackup`本地目录：
```
[root@redhat8 ~]# rsync -av 192.168.100.131:/home/huang/rsynctest \
/home/huang/hostba
ckup
root@192.168.100.131's password: 
receiving incremental file list
rsynctest/
rsynctest/hostbackup/
rsynctest/hostbackup/scripts.tar

sent 55 bytes  received 10,432 bytes  2,996.29 bytes/sec
total size is 10,240  speedup is 0.98
```
## 练习