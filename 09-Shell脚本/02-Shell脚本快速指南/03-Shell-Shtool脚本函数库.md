# Shell-Shtool脚本函数库
&#8195;&#8195;在开源世界中，共享代码才是关键，可以下载大量各式各样的函数，并将其用于自己的应用程序中。本节介绍如何下载、安装、使用GNU shtool shell脚本函数库。
### 下载及安装
Shtool软件包下载地址：[ftp://ftp.gnu.org/gnu/shtool](ftp://ftp.gnu.org/gnu/shtool)

在RHEL系统中直接下载shtool-2.0.8.tar.gz版本：
```
[root@redhat8 function]# wget ftp://ftp.gnu.org/gnu/shtool/shtool-2.0.8.tar.gz
--2020-08-21 11:57:05--  ftp://ftp.gnu.org/gnu/shtool/shtool-2.0.8.tar.gz
           => ‘shtool-2.0.8.tar.gz’
Resolving ftp.gnu.org (ftp.gnu.org)... 103.116.4.197
Connecting to ftp.gnu.org (ftp.gnu.org)|103.116.4.197|:21... failed: Connection refused.
```
失败了，用windows浏览器下载了FTP传到RHEL,建议将文件主目录中,然后提取文件：
```
[root@redhat8 function]# cp shtool-2.0.8.tar.gz /
[root@redhat8 /]# tar -zxvf shtool-2.0.8.tar.gz
[root@redhat8 /]# cd shtool-2.0.8
[root@redhat8 shtool-2.0.8]# ls
AUTHORS       Makefile.in  sh.echo     sh.mkshadow  sh.scpp       shtoolize.pod  test.sh
ChangeLog     NEWS         sh.fixperm  sh.move      sh.slo        shtool.m4      THANKS
configure     RATIONAL     sh.install  sh.path      sh.subst      shtool.pod     VERSION
configure.ac  README       sh.mdate    sh.platform  sh.table      shtool.spec
COPYING       sh.arx       sh.mkdir    sh.prop      sh.tarball    sh.version
INSTALL       sh.common    sh.mkln     sh.rotate    shtoolize.in  test.db
```
### 构建库
源码的安装一般由3个步骤组成：配置(configure)、编译(make)、安装(make install)。

在shtool库文件目录下，使用标准configure和make命令来配置：
```
[root@redhat8 shtool-2.0.8]# ./configure
Configuring GNU shtool (Portable Shell Tool), version 2.0.8 (18-Jul-2008)
Copyright (c) 1994-2008 Ralf S. Engelschall <rse@engelschall.com>
checking whether make sets $(MAKE)... no
checking for perl interpreter... /usr/bin/perl
checking for pod2man conversion tool... /usr/bin/pod2man
configure: creating ./config.status
config.status: creating Makefile
config.status: creating shtoolize
config.status: executing adjustment commands
[root@redhat8 shtool-2.0.8]# make
bash: make: command not found...
Failed to search for file: /mnt/cdrom/BaseOS was not found
```
我的RHEL8.0没有make这个命令，需要安装，可以在GNU的主要FTP服务器上找到：[ftp://ftp.gnu.org/gnu/make](ftp://ftp.gnu.org/gnu/make)。

我有配置本地yum源，刚开始安装报错，查找也能找到make安装包，是因为光盘没挂载上去，挂载上去即可：
```
[root@redhat8 make-4.3]# mount /dev/cdrom /mnt/cdrom
[root@redhat8 make-4.3]# df -m
[root@redhat8 make-4.3]# yum install make
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription
-manager to register.Last metadata expiration check: 0:09:32 ago on Fri 21 Aug 2020 12:50:21 PM EDT.
Dependencies resolved.
==========================================================================================
 Package         Arch              Version                    Repository             Size
==========================================================================================
Installing:
 make            x86_64            1:4.2.1-9.el8              redhat8_os            498 k

Transaction Summary
==========================================================================================
Install  1 Package

Total size: 498 k
Installed size: 1.4 M
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                  1/1 
  Installing       : make-1:4.2.1-9.el8.x86_64                                        1/1 
  Running scriptlet: make-1:4.2.1-9.el8.x86_64                                        1/1 
  Verifying        : make-1:4.2.1-9.el8.x86_64                                        1/1 
Installed products updated.

Installed:
  make-1:4.2.1-9.el8.x86_64                                                               

Complete!
[root@redhat8 function]# make
make: *** No targets specified and no makefile found.  Stop.
```
如果有报错，需要先安装gcc：
```
[root@redhat8 make-4.3]# yum install gcc
```
回到shtool-2.0.8目录，继续之前步骤：
```
[root@redhat8 /]# cd shtool-2.0.8
[root@redhat8 shtool-2.0.8]# make
building program shtool
./shtoolize -o shtool all
Use of assignment to $[ is deprecated at ./shtoolize line 60.
Generating shtool...(echo 11808/12742 bytes)...(mdate 3695/4690 bytes)...(table 1818/2753 
bytes)...(prop 1109/2038 bytes)...(move 2685/3614 bytes)...(install 4567/5495 bytes)...(mkdir 2904/3821 bytes)...(mkln 4429/5361 bytes)...(mkshadow 3260/4193 bytes)...(fixperm 1471/2403 bytes)...(rotate 13425/14331 bytes)...(tarball 5297/6214 bytes)...(subst 5255/6180 bytes)...(platform 21739/22662 bytes)...(arx 2401/3312 bytes)...(slo 4139/5066 bytes)...(scpp 6295/7206 bytes)...(version 10234/11160 bytes)...(path 4041/4952 bytes)building manpage shtoolize.1
...
shtool-mdate.tmp around line 222: You forgot a '=back' before '=head1'
POD document had syntax errors at /usr/bin/pod2man line 69.
building manpage shtool-table.1
...
building manpage shtool-mkdir.1
shtool-mkdir.tmp around line 186: You forgot a '=back' before '=head1'
POD document had syntax errors at /usr/bin/pod2man line 69.
building manpage shtool-mkln.1
building manpage shtool-mkshadow.1
shtool-mkshadow.tmp around line 191: You forgot a '=back' before '=head1'
POD document had syntax errors at /usr/bin/pod2man line 69.
building manpage shtool-fixperm.1
...
building manpage shtool-path.1
```
有一些报错，表示看不懂，可以用make命令测试这个库文件，测试都是通过的：
```
[root@redhat8 shtool-2.0.8]# make test
Running test suite:
echo...........ok
mdate..........ok
table..........ok
prop...........ok
move...........ok
install........ok
mkdir..........ok
mkln...........ok
mkshadow.......ok
fixperm........ok
rotate.........ok
tarball........ok
subst..........ok
platform.......ok
arx............ok
slo............ok
scpp...........ok
version........ok
path...........ok
OK: passed: 19/19
```
要完成安装,要用make命令的install选项（需要root用户）：
```
[root@redhat8 shtool-2.0.8]# make install
./shtool mkdir -f -p -m 755 /usr/local
...
./shtool mkdir -f -p -m 755 /usr/local/share/shtool
...
./shtool install -c -m 644 sh.slo /usr/local/share/shtool/sh.slo
./shtool install -c -m 644 sh.scpp /usr/local/share/shtool/sh.scpp
./shtool install -c -m 644 sh.version /usr/local/share/shtool/sh.version
./shtool install -c -m 644 sh.path /usr/local/share/shtool/sh.path
```
### shtool库函数
下表列出了可用的库函数：

函数|描述
:---|:---
arx|创建归档文件（包含一些扩展功能）
echo|显示字符串，并提供了一些扩展构件
fixperm|改变目录树中的文件权限
mdate|显示文件或目录的修改时间
Table|以表格的形式显示由字段分隔的数据
prop|显示一个带有动画效果的进度条
move|带有替换功能的文件移动
install|安装脚本或文件
mkdir|创建一个或多个目录
mkln|使用相对路径创建链接
mkshadow|创建一颗阴影树
rotate|转置日志文件
tarball|从文件和目录中创建tar文件
subst|使用sed的替换操作
platform|显示平台标识
slo|根据库的类别，分离链接器选项
scpp|共享的C预处理器
version|创建版本信息文件
path|处理程序路径

### 使用库
使用platform库函数的例子：
```sh
#!/bin/bash
shtool platform 
```
运行后示例如下：
```
[root@redhat8 function]# sh test12.sh
os 8.0 (AMD64)
```
在命令行中使用：
```
[root@redhat8 function]# shtool mdate test12.sh
21 August 2020
[root@redhat8 function]# shtool path make
/usr/bin/make
[root@redhat8 function]# ls -al | shtool prop -p "Please waiting..."
Please waiting...
```
&#8195;&#8195;在上面示例中，prop函数可用使用`\、|、/和-`等字符创建一个旋转的进度条，可用提示用户正在进行后台处理工作，能看到多少进度条取决于CPU的性能，选项`-p`运行用户自定义输出文本。
