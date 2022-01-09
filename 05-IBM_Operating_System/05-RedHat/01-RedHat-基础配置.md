# RedHat-基础配置
RedHat接触不多，近期安装了RHEL8.0和RHEL7.8，RHEL8.0发现重启后IP不自动up，hostname也会变。配置很简单，不常用容易忘掉，记录下来。
## 网络和主机名
### RHEL8.0修改hostname
临时修改可以用hostname命令，但是重启后失效：
```
[root@192 ~]# hostname redhat8
[root@192 ~]# hostname
redhat8
```
想永久修改，可以修改文件`/etc/sysconfig/network`文件，加上如下内容：
```
HOSTNAME=redhat8
```
在/etc/hosts 中增加：
```
192.168.18.129 redhat8
```
### RHEL7.8_hostname
RHEL7.8我在安装的时候配置了网络和主机名，然后默认都是永久生效的，主机名配置在`/etc/hostname`文件中，修改应该修改此文件即可：
```
[root@redhat PowerVC]# cat /etc/hostname
redhat
```
### 设置网卡自动启动
REHL8可能是装机时候没配好，重启网络不自动启动，设置方法如下;
配置文件存放目录和文件：`/etc/sysconfig/network-scripts/ifcfg-ens160`，ens160是我机器网卡设备的编号，要改其它就是对应名字。
将配置文件中选项`ONBOOT=no`修改为`ONBOOT=yes`，然后可以重启验证reboot验证。
在RHEL7.8中，修改方法一样，只不过配置文件中值加了引号，如：`ONBOOT='no'`。

## 配置本地YUM源
发现在REHL7.8和REHL8中有点不一样，本地YUM源都是用的光盘。安装软件方式有很多，除了YUM方式安装还有编译安装，还有rpm方式安装，rpm安装参考博客：[https://www.cnblogs.com/chuijingjing/p/9951267.html](https://www.cnblogs.com/chuijingjing/p/9951267.html)
### REHL8配置本地YUM源
创建镜像挂载点并挂载镜像：
```
[root@redhat8 home]# mkdir -p /mnt/cdrom
[root@redhat8 home]# mount /dev/cdrom /mnt/cdrom
mount: /dev/sr0 is write-protected, mounting read-only
[root@redhat8 home]#df -m
```
在`/etc/yum.repos.d`目录下新建一个文件`redhat8.repo`：
```
[root@redhat8 yum.repos.d]# touch /etc/yum.repos.d/redhat8.repo
```
在新建的`redhat8.repo`中写入如下内容：
```
[redhat8_os]
name=redhat8_os
baseurl=file:///mnt/cdrom/BaseOS
enable=1
gpgcheck=0

[redhat8_app]
name=redhat8_app
baseurl=file:///mnt/cdrom/AppStream
enable=1
gpgcheck=0
```
然后验证一下：
```
[root@redhat8 yum.repos.d]# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription
-manager to register.redhat8_app                                               149 MB/s | 5.3 MB     00:00    
redhat8_os                                                124 MB/s | 2.2 MB     00:00    
repo id                                   repo name                                 status
redhat8_app                               redhat8_app                               4,672
redhat8_os                                redhat8_os                                1,658
```
### REHL7.8配置本地YUM源
前面步骤和REHL8一样，只是新建的.repo写入的内容有所差别，新建文件名：`redhat7.repo`，写入如下内容：
```
[local]
name=Red Hat Enterprise Linux 6.8
baseurl=file:///mnt/cdrom
enabled=1
gpgcheck=1
gpgkey=file:///mnt//cdrom/RPM-GPG-KEY-redhat-release
```
然后验证一下：
```
[root@redhat yum.repos.d]# yum repolist
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager
 to register.
local                                                              | 2.8 kB  00:00:00     
(1/2): local/group_gz                                              |  95 kB  00:00:00     
(2/2): local/primary                                               | 2.1 MB  00:00:00     
local                                                                           5231/5231
repo id                        repo name                                            status
local                          Red Hat Enterprise Linux 6.8                         5,231
repolist: 5,231
```
### YUM常用命令
yum常用操作命令如下：     
命令|说明
:---|:---
yum repolist|显示仓库列表
yum list|显示仓库的所有软件包
yum search &#60;Packages name&#62;|搜索软件包
yum check-update |检查升级
yum info &#60;Packages name&#62;|查看软件详细信息
yum provides &#60;Packages name&#62;|查看软件包Provide信息
yum deplist &#60;Packages name&#62;|查看软件包的依赖包
yum install &#60;Program name&#62;|软件安装
yum reinstall &#60;Program name&#62;|重新安装软件
yum update &#60;Program name&#62;| 软件升级
yum downgrade &#60;Program name&#62;| 软件降级
yum remove &#60;Program name&#62;|卸载程序
yum history|查看yum安装的历史

## 设置Python3为默认
RHEL8中自带了Python3，RHEL7.8中自带Python2，Python3会成为趋势，但是每次运行Python都要输入`python3`命令，会很不习惯，可以修改下。
输入如下命令查看`python3`命令的位置：
```
[root@redhat8 bin]# whereis python3
python3: /usr/bin/python3.6 /usr/bin/python3.6m /usr/bin/python3 /usr/lib/python3.6 /usr/l
ib64/python3.6 /usr/include/python3.6m /usr/share/man/man1/python3.1.gz
```
进入到`/usr/bin`目录下，查找python：
```
[root@redhat8 bin]# ls -l |grep python
lrwxrwxrwx. 1 root root          25 Jul 18  2020 python3 -> /etc/alternatives/python3
lrwxrwxrwx. 1 root root          31 Jan 23  2019 python3.6 -> /usr/libexec/platform-pytho3.6
lrwxrwxrwx. 1 root root          32 Jan 23  2019 python3.6m -> /usr/libexec/platform-python3.6m
lrwxrwxrwx. 1 root root          24 Jul 18  2020 unversioned-python -> /etc/alternatives/python
```
可以看到`python3`链接的位置：`/etc/alternatives/python3`
删除现有的并新建一个软链接：
```
[root@redhat8 bin]# rm python3
rm: remove symbolic link 'python3'? y
[root@redhat8 bin]#ln -s /etc/alternatives/python3 python
```
输入`python`命令验证：
```
[root@redhat8 bin]# python
Python 3.6.8 (default, Jan 11 2019, 02:17:16) 
[GCC 8.2.1 20180905 (Red Hat 8.2.1-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
## 安装git
&#8195;&#8195;安装方法在Git章节中，安装过程中出现一些问题也进行了记录，参考链接：[GitHub-使用命令行](https://ebook.big1000.com/10-Git/01-GitHub&Git/01-GitHub-%E4%BD%BF%E7%94%A8%E5%91%BD%E4%BB%A4%E8%A1%8C.html)

## 安装PIP
&#8195;&#8195;在使用Python中需要安装一些第三方库，用pip安装比较方便，RHEL中默认没有，需要安装，使用脚本安装和升级pip。参考博客：[https://www.cnblogs.com/zhongyehai/p/10619917.html](https://www.cnblogs.com/zhongyehai/p/10619917.html)    
下载get-pip.py脚本：
```
[root@redhat8 ~]# wget https://bootstrap.pypa.io/get-pip.py
--2020-12-03 10:10:16--  https://bootstrap.pypa.io/get-pip.py
Resolving bootstrap.pypa.io (bootstrap.pypa.io)... 151.101.76.175, 2a04:4e42:12::175
Connecting to bootstrap.pypa.io (bootstrap.pypa.io)|151.101.76.175|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1886796 (1.8M) [text/x-python]
Saving to: ‘get-pip.py’
get-pip.py             100%[==========================>]   1.80M  42.1KB/s    in 36s     
2020-12-03 10:10:53 (50.9 KB/s) - ‘get-pip.py’ saved [1886796/1886796]
```
运行get-pip.py脚本:
```
[root@redhat8 ~]# python3 get-pip.py
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/, https://pypi.douban.com/simpl
e/Collecting pip
  Downloading https://mirrors.aliyun.com/pypi/packages/ab/11/2dc62c5263d9eb322f2f028f7b56c
d9d096bb8988fcf82d65fa2e4057afe/pip-20.3.1-py2.py3-none-any.whl (1.5 MB)     |███████████████████████▎        | 1.1 MB 3.0 MB/s eta 0:00:0
     |███████████████████████▌        | 1.1 MB 3.0 MB/s eta 0:00:0
...
     |████████████████████████████████| 1.5 MB 997 kB/s 
Collecting wheel
  Downloading https://mirrors.aliyun.com/pypi/packages/84/e8/3caa0d932d0edd9c9611065831fd0
6cf05c53671f78a0fa553a635da7e2f/wheel-0.36.0-py2.py3-none-any.whl (34 kB)Installing collected packages: pip, wheel
  Attempting uninstall: pip
    Found existing installation: pip 9.0.3
    Uninstalling pip-9.0.3:
      Successfully uninstalled pip-9.0.3
Successfully installed pip-20.3.1 wheel-0.36.0
```
查看pip版本：
```
[root@redhat8 ~]# pip -V
pip 20.3.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
```
使用pip安装Python第三方模块click示例：
```
[root@redhat8 ~]# pip install click
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/, https://pypi.douban.com/simpl
e/Collecting click
  Downloading https://pypi.doubanio.com/packages/d2/3d/fa76db83bf75c4f8d338c2fd15c8d33fdd7
ad23a9b5e57eb6c5de26b430e/click-7.1.2-py2.py3-none-any.whl (82 kB)     |███████████████████████████▊    | 71 kB 601 kB/s eta 0:0
     |███████████████████████████████▊| 81 kB 682 kB/s eta
     |████████████████████████████████| 82 kB 540 kB/s 
Installing collected packages: click
Successfully installed click-7.1.2
```
&#8195;&#8195;对于PIP其它安装方法及常用命令，在本ebook的章节[Python运维-基础知识](https://ebook.big1000.com/14-Python%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E4%B8%8E%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/01-Python%E8%BF%90%E7%BB%B4-%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/01-Python%E8%BF%90%E7%BB%B4-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.html?h=pip)中有详细的介绍和示例演示。
## 待补充
