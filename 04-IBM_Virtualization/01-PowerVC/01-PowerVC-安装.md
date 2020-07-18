# PowerVC-安装
PowerVC支持得系统不多，主要是RedHat和SUSE。但是当前还不不支持RHEL 8。RHEL 7.6 ALT支持PowerVC SDI版本1.4.4和1.4.4.1。
### RedHat安装PowerVC
系统环境：Red Hat Enterprise Linux 7.8 64 位
#### 安装包准备
需要准备得安装包如下：
- 系统安装包：rhel-server-7.8-x86_64-dvd.iso
- PowerVM安装包：powervc-install-x86-rhel-1.4.4.0.tgz
- 如果系统中没有Python3，需要准备Python3安装包
- Python依赖包，打包名字：redhat_relevant for PowerVC.tar

#### 检查系统环境
查看yum：
```
[root@redhat ~]# rpm -qa |grep yum
yum-rhn-plugin-2.0.1-10.el7.noarch
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-3.4.3-167.el7.noarch
```
查看yum源：
```
[root@redhat PowerVC]# yum repolist
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager
 to register.
repolist: 0
```
提示没有注册，并且没有，后面进行配置。
检查是否安装Python
```
[root@redhat ~]# python
Python 2.7.5 (default, Sep 26 2019, 13:23:47) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
#### 系统配置
##### 网络配置
PowerVC使用默认的网络接口：eth0。要使用其他网络接口HOST_INTERFACE，请在运行安装脚本之前设置环境变量。例如： export HOST_INTERFACE=eth1。
我得虚拟机是ens160，修改查看如下：
```
[root@redhat ~]# export HOST_INTERFACE=ens160
[root@redhat ~]# env
```
查看hostname，并且在/etc/hosts中添加目前IP和hostname：
```
[root@redhat PowerVC]# cat /etc/hostname
redhat
[root@redhat PowerVC]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.18.130	redhat
```
##### 配置本地yum源
对于RedHat Enterprise Linux 7，安装PowerVC的一些先决条件已从OS介质移至使用RHN连接访问的Optional Software通道。所以我们配置个本地yum源。
```
#创建镜像挂载点
[root@redhat PowerVC]# mkdir -p /mnt/cdrom
#挂载镜像
[root@redhat PowerVC]# mount /dev/cdrom /mnt/cdrom
mount: /dev/sr0 is write-protected, mounting read-only
#在/etc/yum.repos.d目录下建一个文件redhat7.repo
[root@redhat yum.repos.d]# touch /etc/yum.repos.d/redhat7.repo
[root@redhat yum.repos.d]# ls
redhat7.repo  redhat.repo
```
redhat7.repo中写入如下内容：
```
[local]
name=Red Hat Enterprise Linux 6.8
baseurl=file:///mnt/cdrom
enabled=1
gpgcheck=1
gpgkey=file:///mnt//cdrom/RPM-GPG-KEY-redhat-release
```
再次运行`yum repolist`进行验证：
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
##### 安装net.tool
我这个版本必须安装，输入如下命令安装：
```
[root@redhat powervc-1.4.4.0]# yum install net-tools
```
##### 安装依赖包
此依赖包官方统称：Red Hat Enterprise Linux packages relevant to PowerVC
共包含：python-zope-interface；python-jinja2；python-pyasn1-modules；python-webob；python-webtest；python-libguestfs；SOAPpy；pyserial；python-fpconst；python-twisted-core；python-twisted-web
看名字和Python有关，进入到存放依赖包的目录，文件名：redhat_relevant for PowerVC.tar
```
[root@redhat PowerVC]#  tar -xvf 'redhat_relevant for PowerVC.tar'
```
解压后进入生成的目录，安装所有的包，运行脚本开始安装：
```
[root@redhat redhat_relevant_for_PowerVC]# ./install.sh
```
#### 安装PowerVC
进入到存放PowerVC安装包的目录，我直接传过来文件是tar文件：powervc-install-x86-rhel-1.4.4.0.tar
将文件进行解压
```
[root@redhat PowerVC]#  tar -xvf powervc-install-x86-rhel-1.4.4.0.tar
```
解压后生成目录：powervc-1.4.4.0，进入到此目录进行安装：
```
[root@redhat powervc-1.4.4.0]# ls
cloud  install  lib      locale    RPM-GPG-KEY-PowerVC
gpfs   lap      license  packages  version.properties
[root@redhat powervc-1.4.4.0]# ./install
################################################################################
Starting the IBM PowerVC 1.4.4.0 installation on:
2020-07-18T21:20:13-04:00
################################################################################
Select the edition to install:
   1 - IBM PowerVC Standard
   2 - IBM Cloud PowerVC Manager 
   9 - Exit
```
根据安装需求选择安装类型，中途提示license的回复1同意即可，防火墙提示根据需求选择。
等待安装大约十几分钟，报错了，因为内存不够进程被killed了，看来虚拟机扛不住，不过方法应该没错。
```
./install: line 929: 29414 Killed                  /usr/sbin/semanage permissive -d $domai
nFailed to restore security context. See install log for details.
```
