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