# GitHub-使用命令行
&#8195;&#8195;刚接触的一般会使用GitHub的web界面对自己的Repositories进行操作，简单方便。但是有些操作在web界面是无法完成的，例如就遇到想改文件夹名字，发现没找到，只能用命令行进行；在大批量的修改上传代码的时候，用命令行操作也能体现优势。
## 安装Git
官方安装文档：[https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
### Debian和Ubuntu安装
官方下载链接:[https://git-scm.com/downloads](https://git-scm.com/downloads)      
Windows下直接运行安装文件即可。        
Debian和Ubuntu中安装方法如下：
```shell
apt-get install git
```
###  RHEL安装
&#8195;&#8195;RHEL可以下载tarball并从源代码进行构建：[https://www.kernel.org/pub/software/scm/git/](https://www.kernel.org/pub/software/scm/git/)， 或从GitHub网站上的镜像来获得：[https://github.com/git/git/releases](https://github.com/git/git/releases)

下载的最新版本（git-2.29.2.tar.gz）上传到服务器并解压,安装步骤如下：
```
[root@redhat8 python]# tar -zxvf git-2.29.2.tar.gz
[root@redhat8 python]# ls
git-2.29.2  git-2.29.2.tar.gz  test_pdb.py
[root@redhat8 python]# cd git-2.29.2
[root@redhat8 git-2.29.2]# ./configure
...
checking for POSIX Threads with '-pthread'... yes
configure: creating ./config.status
config.status: creating config.mak.autogen
config.status: executing config.mak.autogen commands
[root@redhat8 git-2.29.2]# make
    CC fuzz-commit-graph.o
In file included from object-store.h:4,
                 from commit-graph.h:5,
                 from fuzz-commit-graph.c:1:
cache.h:21:10: fatal error: zlib.h: No such file or directory
 #include <zlib.h>
          ^~~~~~~~
compilation terminated.
make: *** [Makefile:2433: fuzz-commit-graph.o] Error 1
[root@redhat8 git-2.29.2]# rpm -qa|grep zlib
zlib-1.2.11-10.el8.x86_64
```
查询可能是缺少zlib-devel,我配置的yum源里面没有：
```
[root@redhat8 git-2.29.2]# yum install zlib-devel -y
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription
-manager to register.redhat8_app                                               0.0  B/s |   0  B     00:00    
redhat8_os                                                0.0  B/s |   0  B     00:00    
Failed to synchronize cache for repo 'redhat8_app', ignoring this repo.
Failed to synchronize cache for repo 'redhat8_os', ignoring this repo.
No match for argument: zlib-devel
Error: Unable to find a match
```
官网下载最新rpm包然后安装：
```
[root@redhat8 python]# ls
git-2.29.2  git-2.29.2.tar.gz  test_pdb.py  zlib-devel-1.2.11-23.fc33.x86_64.rpm
[root@redhat8 python]# rpm -ivh zlib-devel-1.2.11-23.fc33.x86_64.rpm
warning: zlib-devel-1.2.11-23.fc33.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 9570
ff31: NOKEYerror: Failed dependencies:
	rpmlib(PayloadIsZstd) <= 5.4.18-1 is needed by zlib-devel-1.2.11-23.fc33.x86_64
	zlib(x86-64) = 1.2.11-23.fc33 is needed by zlib-devel-1.2.11-23.fc33.x86_64
[root@redhat8 python]# rpm -qa|grep zlib
zlib-1.2.11-10.el8.x86_64
[root@redhat8 python]# ls
git-2.29.2         test_pdb.py         zlib-devel-1.2.11-23.fc33.x86_64.rpm
git-2.29.2.tar.gz  zlib-1.2.11.tar.gz
[root@redhat8 python]# tar -zxvf zlib-1.2.11.tar.gz
[root@redhat8 python]# cd zlib-1.2.11
[root@redhat8 zlib-1.2.11]# ./configure
Checking for gcc...
Checking for shared library support...
Building shared library libz.so.1.2.11 with gcc.
...
Checking whether to use vs[n]printf() or s[n]printf()... using vs[n]printf().
Checking for vsnprintf() in stdio.h... Yes.
Checking for return value of vsnprintf()... Yes.
Checking for attribute(visibility) support... Yes.
[root@redhat8 zlib-1.2.11]# make 
[root@redhat8 zlib-1.2.11]# make install
[root@redhat8 python]# rpm -qa |grep zlib
zlib-1.2.11-10.el8.x86_64
```
&#8195;&#8195;版本还是没变，然后下载了zlib-1.2.11-13.el8.x86_64.rpm，直接安装版本还是不会变，重启也没用，尝试升级就可以了，如下所示：
```
[root@redhat8 python]# rpm -Uvh zlib-1.2.11-13.el8.x86_64.rpm
warning: zlib-1.2.11-13.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: N
OKEYVerifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:zlib-1.2.11-13.el8               ################################# [ 50%]
Cleaning up / removing...
   2:zlib-1.2.11-10.el8               ################################# [100%]
[root@redhat8 python]# rpm -qa |grep zlib
zlib-1.2.11-13.el8.x86_64
[root@redhat8 python]# rpm -ivh zlib-devel-1.2.11-13.el8.x86_64.rpm
warning: zlib-devel-1.2.11-13.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c
65d: NOKEYVerifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:zlib-devel-1.2.11-13.el8         ################################# [100%]
[root@redhat8 python]# rpm -qa |grep zlib
zlib-devel-1.2.11-13.el8.x86_64
zlib-1.2.11-13.el8.x86_64
```
继续安装git：
```
[root@redhat8 git-2.29.2]# make
...
    CC t/helper/test-trace2.o
    CC t/helper/test-urlmatch-normalization.o
    CC t/helper/test-wildmatch.o
...
    GEN bin-wrappers/git-cvsserver
    GEN bin-wrappers/test-fake-ssh
    GEN bin-wrappers/test-tool
[root@redhat8 git-2.29.2]# make install
[root@redhat8 git-2.29.2]# make install
    SUBDIR git-gui
    SUBDIR gitk-git
    SUBDIR templates
install -d -m 755 '/usr/local/bin'
install -d -m 755 '/usr/local/libexec/git-core'
...
```
安装成功后尝试使用git命令：
```
[root@redhat8 git-2.29.2]# git version
git version 2.29.2
[root@redhat8 git-2.29.2]# git
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
...
```
参考博客：[安装失败：error: zlib.h: No such file or directory](http://blog.chinaunix.net/uid-20344928-id-5751083.html)     
gzip网站：[http://www.gzip.org/](http://www.gzip.org/)      
zlib官网：[http://www.zlib.net/](http://www.zlib.net/)      
zlib各版本下载链接：[http://rpmfind.net/linux/rpm2html/search.php?query=zlib(x86-64)] (http://rpmfind.net/linux/rpm2html/search.php?query=zlib(x86-64))
zlib-devel下载链接：[http://rpmfind.net/linux/rpm2html/search.php?query=zlib-devel](http://rpmfind.net/linux/rpm2html/search.php?query=zlib-devel)      

### 其它系统安装
其它Linux或者Unix参考官方链接：[https://git-scm.com/download/linux](https://git-scm.com/download/linux)
## 配置Git
我使用的是Windows，安装成功后点击开始菜单，找到Git Bash点击打开命令行
依次输入如下命令：
```shell
#初始化
git init
#生成密钥，输入GitHub注册邮箱
ssh–keygen –t rsa –C <email add>
```
生成的密钥在目录<User directory>/.ssh下面，打开.pub的文件，复制里面的所有内容。
在GitHub WEB界面依次进行如下操作：
- 点击右上角头像，选择选项“setting”
- 点击左边导航栏“SSH and GPG keys"
- 点击选项“New SSH key”
- “Title”自定义，“Kek”栏里把刚才复制的内容粘贴上去
- 点击选项“Add SSH key”

添加成功后回到Git Bash，输入如下命令进行验证：
```shell
ssh -T git@github.com
```
提示成功即可以了。
## 克隆仓库
我是预先再GitHub已经有Repositories了，所以不需要新建，直接clone过来即可。
再GitHub WEB界面依次进行如下操作：
- 进入对应需要clone的Repositories界面
- 点击绿色“Code”选项
- 如果显示“Clone with HTTPS”，则点击右边“Use SSH"
- 显示为“Clone with SSH”时，在链接右边有复制按钮，点击复制链接

进入到Git，输入如下命令进行克隆：
```shell
git clone git@github.com:bond-huang/huang.git
```
等待片刻，即克隆到本地了，输入ls命令也查看目录，有Repositories名字的目录，CD进入目录，就可以对远程Repositories进行相应操作。
