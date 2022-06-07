# RHEL-安装和更新软件包
## 注册系统以获取红帽支持
### 红帽订阅管理
&#8195;&#8195;红帽订阅管理提供可用于向计算机授权产品订阅的工具，让管理员能够获取软件包的更新，并且跟踪系统所用支持合同和订阅的相关信息。`PackageKit`和`yum`等标准工具可以通过红帽提供的内容分发网络获取软件包和更新。可以通过红帽订阅管理工具执行下列四项基本任务： 
- 注册系统，将该系统与某一红帽帐户关联。这可以让订阅管理器唯一地清查该系统
- 订阅系统，授权它获取所选红帽产品的更新。订阅包含特定的支持级别、到期日期和默认存储库
- 启用存储库，以提供软件包。默认情况下每一订阅会启用多个存储库，但可以根据需要启用或禁用更新或源代码等其他存储库
- 审核和跟踪可用或已用的授权。可以在具体系统上本地查看订阅信息，也可在红帽客户门户的订阅页面或订阅资产管理器(SAM)查看具体帐户的订阅信息

### 注册系统
&#8195;&#8195;将系统注册到红帽客户门户的方法有很多种。可以使用`GNOME`应用程序或通过`Web`控制台服务访问相应的图形界面，也可以使用命令行工具。
#### 命令行注册
通过`subscription-manager`命令可在不使用图形环境的前提下注册系统。`subscription-manager`命令可以自动将系统关联到最适合该系统的兼容订阅。注册系统到红帽帐户：
```
[user@host ~]$ subscription-manager register --username=yourusername \
--password=yourpassword
```
查看可用的订阅：
```
[user@host ~]$ subscription-manager list --available | less
```
自动附加订阅：
```
[user@host ~]$ subscription-manager attach --auto
```
从可用订阅列表的特定池中附加订阅：
```
[user@host ~]$ subscription-manager attach --pool=poolID
```
查看已用的订阅：
```
[user@host ~]$ subscription-manager list --consumed
```
取消注册系统：
```
[user@host ~]$ subscription-manager unregister
```
注意事项：
- `subscription-manager`也可搭配激活密钥使用，以此注册和分配预定义的订阅，而不必使用用户名或密码。这种注册方式对于自动化安装和部署非常有用
- 激活密钥通常由内部订阅管理服务颁发，如订阅资产管理器或红帽卫星

### 授权证书
&#8195;&#8195;授权是附加至某一系统的订阅。数字证书用于存储本地系统上有关授权的当前信息。注册之后，授权证书存储在`/etc/pki`和其子目录中：
- `/etc/pki/product`中的证书指明系统上安装了哪些红帽产品
- `/etc/pki/consumer`中的证书指明系统所注册到的红帽帐户
- `/etc/pki/entitlement`中的证书指明该系统附加有哪些订阅 

&#8195;&#8195;可以通过`rct`实用程序直接检查证书，但`subscription-manager`工具更加容易查看系统所附加的订阅。
## RPM软件包
### 软件包和RPM
&#8195;&#8195;RPM Package Manager最初是由红帽开发的，该程序提供了一种标准的方式来打包软件进行分发。与使用从存档提取到文件系统的软件相比，采用RPM软件包形式管理软件要简单得多：
- 管理员可以通过它跟踪软件包所安装的文件，需要删除哪些软件（如果卸载）并检查确保显示支持软件包（如果安装）
- 有关已安装软件包的信息存储在各个系统的本地RPM数据库中
- 红帽为红帽企业Linux提供的所有软件都以RPM软件包的形式提供

&#8195;&#8195;RPM软件包文件名由四个元素组成（再加上`.rpm`后缀）：`name-version-release.architecture`，openssh的RPM软件包名称示例如下：
```
openssh-7.8p1-4.el8.x86_64.prm
```
字段说明：
- `Name`是描述其内容的一个或多个词语(openssh)
- `Version`是原始软件的版本号(7.8p1)
- `Release`是基于该版本的软件包的发行版号，由软件打包商设置，后者不一定是原始软件开发商(4.el8)
- `Architecture`是编译的软件包运行的处理器架构：
    - `noarch`表示此软件包的内容不限定架构
    - `x86_64`表示64位的x86_64架构
    - `aarch64`表示64位的ARM架构

 从存储库安装软件包时，只需要软件包的名称。如果存在多个版本，则会安装具有更高版本号的软件包。如果一个版本存在多个发行版，则会安装具有更高发行版号的软件包。

每个RPM软件包是包含以下三个组成部分的特殊存档：
- 软件包安装的文件
- 与软件包（元数据）有关的信息，如：
    - `name`、`version`、`release`和`arch`
    - 软件包的摘要和描述
    - 是否要求安装其他软件包
    - 授权许可信息
    - 软件包更改日志
    - 其他详细信息
- 在安装、更新或删除此软件包时可能运行的脚本，或者在安装、更新或删除其他软件包时触发的脚本 

&#8195;&#8195;软件提供商使用GPG密钥对RPM软件包进行数字签名（红帽为其发行的所有软件包签署数字签名）。RPM系统通过确认软件包已由相应的GPG密钥签名来验证软件包的完整性。如果GPG签名不匹配，RPM系统拒绝安装软件包。
#### 通过RPM软件包更新软件
红帽生成一个完整的RPM软件包来更新软件：
- 管理员安装该软件包时，仅获取该软件包的最新版本
- 红帽不要求先安装旧软件包，再打补丁
- 为了更新软件，RPM会删除旧版本的软件包，再安装新版本
- 更新通常会保留配置文件，但新版本的打包程序会定义确切的行为
- 在大多数情形中，一次仅可安装软件包的一个版本或发行版：
    - 如果软件包构建为没有冲突的文件名，则可安装多个版本
    - 最重要的相关例子是kernel软件包。由于新的内核只有通过启动至该内核才能进行测试，该软件包进行了特殊设计，以便一次能够安装多个版本。如果新内核启动失败，则旧内核依然可用并可启动

### 通过RPM软件包更新软件
&#8195;&#8195;`rpm`实用程序可获取软件包文件和已安装软件包的内容的相关信息。默认情况下，它从已安装软件包的本地数据库中获取信息。可以使用`-p`选项来指定想获取有关已下载软件包文件的信息。一般查询格式是：
```
rpm -q [select-options] [query-options]
```
RPM查询关于已安装的软件包的一般信息：
- `rpm -qa`：列出所有已安装的软件包
- `rpm -qf FILENAME`：查找提供`FILENAME`的软件包，示例：
    ```
    [root@redhat8 ~]# rpm -qf /etc/yum.repos.d
    redhat-release-8.0-0.44.el8.x86_64
    ```

RPM查询关于特定软件包的信息：
- `rpm -q`：列出当前安装的软件包的版本，示例：
    ```
    [root@redhat8 ~]# rpm -q openssh
    openssh-7.8p1-4.el8.x86_64
    ```
- `rpm -qi`：获取有关软件包的详细信息
- `rpm -ql`：列出软件包安装的文件，示例：
    ```
    [root@redhat8 ~]# rpm -ql yum
    /etc/yum.conf
    /etc/yum/pluginconf.d
    /etc/yum/protected.d
    /etc/yum/vars
    /usr/bin/yum
    /usr/share/man/man1/yum-aliases.1.gz
    /usr/share/man/man5/yum.conf.5.gz
    /usr/share/man/man8/yum-shell.8.gz
    /usr/share/man/man8/yum.8.gz
    ```
- `rpm -qc`：仅列出软件包安装的配置文件，示例：
    ```
    [root@redhat8 ~]# rpm -qc openssh-clients
    /etc/ssh/ssh_config
    /etc/ssh/ssh_config.d/05-redhat.conf
    ```
- `rpm -qd`：仅列出软件包安装的文档文件，示例：
    ```
    [root@redhat8 ~]# rpm -qd yum
    /usr/share/man/man1/yum-aliases.1.gz
    /usr/share/man/man5/yum.conf.5.gz
    /usr/share/man/man8/yum-shell.8.gz
    /usr/share/man/man8/yum.8.gz
    ```
- `rpm -q --scripts`：列出在安装或删除软件包前后运行的shell脚本，示例：
    ```
    [root@redhat8 ~]# rpm -q --scripts openssh
    preinstall scriptlet (using /bin/sh):
    getent group ssh_keys >/dev/null || groupadd -r ssh_keys || :
    ```
- `rpm -q --changelog`：列出软件包的更改信息，示例：
    ```
    [root@redhat8 ~]# rpm -q --changelog python3-pip
    * Mon Dec 03 2018 Miro Hrončok <mhroncok@redhat.com> - 9.0.3-13
    - Use the system level root certificate instead of the one bundled in certifi
    - Resolves: rhbz#1655255

    * Wed Nov 28 2018 Tomas Orsava <torsava@redhat.com> - 9.0.3-12
    - Do not show the "new version of pip" warning outside of venv
    - Resolves: rhbz#1656171
    ...output omitted...
    ```

查询本地软件包文件：
```
[root@redhat8 Downloads]# ls -l oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm 
-rwx---rwx. 1 huang huang 18204 May 14 14:12 oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm[root@redhat8 Downloads]# rpm -qlp oracle-database-preinstall-19c-1.0-1.el7.x86_64.r
pm /etc/rc.d/init.d/oracle-database-preinstall-19c-firstboot
/etc/security/limits.d/oracle-database-preinstall-19c.conf
/etc/sysconfig/oracle-database-preinstall-19c
/etc/sysconfig/oracle-database-preinstall-19c/oracle-database-preinstall-19c-verify
/etc/sysconfig/oracle-database-preinstall-19c/oracle-database-preinstall-19c.param
/usr/bin/oracle-database-preinstall-19c-verify
/var/log/oracle-database-preinstall-19c
/var/log/oracle-database-preinstall-19c/results
```
### 安装RPM软件包
`rpm`命令可用于安装本地目录的RPM软件包。示例：
```
[root@redhat8 tmp]# rpm -ivh vsftpd-3.0.3-32.el8.x86_64.rpm
warning: vsftpd-3.0.3-32.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: 
NOKEYVerifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:vsftpd-3.0.3-32.el8              ################################# [100%]
```
注意事项：
- 安装第三方软件包时一定要小心，不仅是因为它们可能安装的软件，而且还因为RPM软件包可能会含有在安装过程中以root用户身份运行的任意脚本

&#8195;&#8195;从RPM软件包文件中提取文件，而不安装此软件包。`rpm2cpio`实用程序可以将RPM的内容传递给名为`cpio`的特殊归档工具，后者可以提取所有文件或单个文件。将`rpm2cpio PACKAGEFILE.rpm`的输出传送到`cpio -id`，以提取RPM软件包中存储的所有文件。需要时，会相对于当前工作目录创建子目录树。示例：
```
[root@redhat8 Downloads]# rpm2cpio libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm \
| cpio -id
23311 blocks
[root@redhat8 Downloads]# ls
libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm             usr
oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
[root@redhat8 Downloads]# ls -ld usr
drwxr-xr-x. 5 root root 45 Jun  1 15:38 usr
```
可以通过指定文件的路径来提取各个文件：
```
[root@redhat8 Downloads]# rpm2cpio libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm \
| cpio -id "*txt"
23311 blocks
[root@redhat8 Downloads]# ls -l usr/share/doc/libstdc++-devel/
total 768
-rw-r--r--. 1 root root   233 Jun  1 15:38 README
```
### RPM查询命令摘要
&#8195;&#8195;可以直接通过`rpm`命令查询已安装的软件包。加上`-p`选项即可在安装之前查询软件包文件。RPM查询命令摘要如下表所示：

命令|任务
:---|:---
rpm -qa|列出当前安装的所有RPM软件包
rpm -q NAME|显示系统上安装的NAME版本
rpm -qi NAME|显示有关软件包的详细信息
rpm -ql NAME|列出软件包中含有的所有文件
rpm -qc NAME|列出软件包中含有的配置文件
rpm -qd NAME|列出软件包中含有的文档文件
rpm -q --changelog NAME|显示软件包新发行版的简短原因摘要
rpm -q --scripts NAME|显示在软件包安装、升级或删除时运行的shell脚本

## 使用Yum安装和更新软件包
`yum`目前已被`dnf`命令取代，用法大同小异。
### Yum命令摘要
可以根据名称或软件包组，查找、安装、更新和删除软件包：

命令|任务
:---|:---
yum list \[NAME-PATTERN]|按名称列出已安装和可用的软件包
yum group list|列出已安装和可用的组
yum search KEYWORD|按关键字搜索软件包
yum info PACKAGENAME|显示软件包的详细信息
yum install PACKAGENAME|安装软件包
yum group install GROUPNAME|安装软件包组
yum update|更新所有软件包
yum remove PACKAGENAME|删除软件包
yum history|显示事务历史记录

### 使用Yum管理软件包
&#8195;&#8195;`Yum`设计目标是在管理基于RPM的软件安装和更新方面成为一个更理想的系统。`yum`命令允许安装、更新、删除和获取有关软件包及其依赖项的信息。可以获取已执行事务的历史记录并使用多个红帽及第三方软件存储库。
#### 使用Yum查找软件
`yum help`显示用法信息：
```
[root@redhat8 ~]# yum help
Updating Subscription Management repositories.
You can use subscr iption-manager to register.usage: dnf [options] COMMAND

List of Main Commands:
alias                     List or create command aliases
autoremove                remove all unneeded packages that were originally installe
d as dependenciescheck                     check for problems in the packagedb
check-update              check for available package upgrades
clean                     remove cached data
...output omitted...
```
`yum list`显示已安装和可用的软件包，示例：
```
[root@redhat8 ~]# yum list 'http*'
httpd.x86_64                     2.4.37-10.module+el8+2764+7127e69e     @redhat8_app
httpd-filesystem.noarch          2.4.37-10.module+el8+2764+7127e69e     @redhat8_app
httpd-tools.x86_64               2.4.37-10.module+el8+2764+7127e69e     @redhat8_app
Available Packages
http-parser.i686                 2.7.1-9.el7                            ol7_latest  
http-parser.src                  2.7.1-9.el7                            ol7_latest  
...output omitted...
```
`yum search KEYWORD`根据仅在名称和摘要字段中找到的关键字列出软件包：
```
[root@redhat8 ~]# yum search 'python'
========================= Name Exactly Matched: python =========================
python.src : An interpreted, interactive, object-oriented programming language
python.x86_64 : An interpreted, interactive, object-oriented programming
              : language
======================== Name & Summary Matched: python ========================
python-ply.noarch : Python Lex-Yacc
python-ply.src : Python Lex-Yacc
...output omitted...
```
搜索名称、摘要和描述字段中包含`web server`的软件包，使用`search all`：
```
eyboardInterrupt: Terminated.
[root@redhat8 ~]# yum search all 'web server'
==================== Description & Summary Matched: web server =====================
yawn-server.noarch : Standalone web server for yawn
erlang-inets.x86_64 : A set of services such as a Web server and a ftp client etc
...output omitted...
```
`yum info PACKAGENAME`返回与软件包相关的详细信息，包括安装所需的磁盘空间：
```
[root@redhat8 ~]# yum info httpd
Available Packages
Name         : httpd
Version      : 2.4.6
Release      : 97.0.5.el7_9.5
Arch         : src
Size         : 5.0 M
Source       : None
Repo         : ol7_latest
Summary      : Apache HTTP Server
URL          : http://httpd.apache.org/
License      : ASL 2.0
Description  : The Apache HTTP Server is a powerful, efficient, and extensible
             : web server.
```
`yum provides PATHNAME`显示与指定的路径名（通常包含通配符）匹配的软件包：
```
[root@redhat8 ~]# yum provides /var/www/html
httpd-filesystem-2.4.37-10.module+el8+2764+7127e69e.noarch : The basic directory
     ...: layout for the Apache HTTP server
Repo        : @System
Matched from:
Filename    : /var/www/html

httpd-2.4.6-80.0.1.el7.x86_64 : Apache HTTP Server
Repo        : ol7_latest
Matched from:
Filename    : /var/www/html
...output omitted...
```
#### 使用yum安装和删除软件
`yum install PACKAGENAME`获取并安装软件包，包括所有依赖项：
```
[root@redhat8 ~]# yum install python3
...output omitted...
Running transaction
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                         1/5 
  Installing : python3-setuptools-39.2.0-10.el7.noarch                                2/5 
  Installing : python3-pip-9.0.3-8.el7.noarch                                         3/5 
  Installing : python3-3.6.8-18.el7.x86_64                                            4/5 
  Installing : python3-libs-3.6.8-18.el7.x86_64                                       5/5
...output omitted...
```
&#8195;&#8195;`yum update PACKAGENAME`获取并安装指定软件包的较新版本，包括所有依赖项。通常，该进程尝试适当保留配置文件，但是在某些情况下，如果打包商认为旧文件在更新后将无法使用，则可能对其进行重命名。如果未指定 `PACKAGENAME`，将安装所有相关更新：
```
[root@redhat8 ~]# yum update python
``` 
&#8195;&#8195;由于新的内核只有通过启动至该内核才能进行测试，该软件包进行了特殊设计，以便一次能够安装多个版本。如果新内核启动失败，则依然可以使用旧的内核。使用`yum update kernel`实际上会安装新的内核。配置文件中保存一份软件包列表，即使在管理员要求更新时也始终安装这些软件包。使用`yum list kernel`可列出所有已安装和可用的内核：
```
[root@redhat8 ~]# yum list kernel
Installed Packages
kernel.x86_64                    4.18.0-80.el8                            @anaconda 
Available Packages
kernel.src                       3.10.0-1160.66.1.el7                     ol7_lates
```
&#8195;&#8195;若要查看当前运行中的内核，请使用`uname`命令。`-r`选项仅显示内核的版本和发行版本，而`-a`选项显示内核发行版和其他信息。示例如下：
```
[root@redhat8 ~]# uname -r
4.18.0-80.el8.x86_64
[root@redhat8 ~]# uname -a
Linux redhat8 4.18.0-80.el8.x86_64 #1 SMP Wed Mar 13 12:02:46 UTC 2019 x86_64 x86_64
 x86_64 GNU/Linux
```
`yum remove PACKAGENAME`删除安装的软件包，包括所有受支持的软件包：
```
[root@redhat8 ~]# yum remove python3
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscr
iption-manager to register.Dependencies resolved.
====================================================================================
 Package             Arch    Version                              Repository   Size
====================================================================================
Removing:
 python36            x86_64  3.6.8-1.module+el8+2710+846623d6     @AppStream   13 k
Removing unused dependencies:
 python3-pip         noarch  9.0.3-13.el8                         @AppStream  2.5 k
 python3-setuptools  noarch  39.2.0-4.el8                         @anaconda   450 k
Transaction Summary
====================================================================================
Remove  3 Packages
Freed space: 466 k
Is this ok [y/N]: n
Operation aborted.
```
#### 使用yum安装和删除各组软件
&#8195;&#8195;`yum`也具有组的概念，即针对特定目的而一起安装的相关软件集合。在RHEL8中，有两种类型的组。常规组是软件包的集合。环境组是常规组的集合。一个组提供的软件包或组可能为：
- `mandatory`（安装该组时必须予以安装）
- `default`（安装该组时通常会安装）
- `optional`（安装该组时不予以安装，除非特别要求）

&#8195;&#8195;`yum group list`命令可显示已安装和可用的组的名称(有些组一般通过环境组安装，默认为隐藏。可通过`yum group list hidden`命令列出这些隐藏组)：
```
[root@redhat8 ~]# yum group list
Available Environment Groups:
   Minimal Install
   Infrastructure Server
   ...output omitted...
Installed Environment Groups:
   Server with GUI
Available Groups:
   Cinnamon
   Educational Software
   ...output omitted...
```
`yum group info`显示组的相关信息。它将列出必选、默认和可选软件包名称：
```
[root@redhat8 ~]# yum group info "Internet Applications"
Group: Internet Applications
 Description: Email, chat, and video conferencing software.
 Optional Packages:
   checkgmail
   konversation
   mail-notification
```
`yum group install`将安装一个组，同时安装其必选和默认的软件包，以及它们依赖的软件包：
```
[root@redhat8 ~]# yum group install "RPM Development Tools"
```
#### 查看事务历史记录
所有安装和删除事务的日志记录在`/var/log/dnf.rpm.log`中：
```
[root@redhat8 ~]# tail -5 /var/log/dnf.rpm.log
2022-06-07T14:14:19Z INFO --- logging initialized ---
2022-06-07T14:17:13Z INFO --- logging initialized ---
2022-06-07T14:19:18Z INFO --- logging initialized ---
2022-06-07T14:19:39Z INFO --- logging initialized ---
2022-06-07T14:20:38Z INFO --- logging initialized ---
```
`history undo`选项可以撤销事务：
```
[root@redhat8 ~]# yum history undo 5
```
## 启用Yum软件存储库
### 启用红帽软件存储库
查看所有可用的存储库：
```
[root@redhat8 ~]# yum repolist all
repo id                  repo name                                   status
*epel                    Extra Packages for Enterprise Linux 7 - x86 enabled: 13,753
epel-debuginfo           Extra Packages for Enterprise Linux 7 - x86 disabled
...output omitted...
ol7_developer_php72      Oracle Linux 8 PHP 7.2 Packages for Develop disabled
ol7_gluster312           Oracle Linux 8 Gluster 3.12 Packages (x86_6 disabled
ol7_latest               Oracle Linux 8 Latest (x86_64)              enabled: 24,399
ol7_latest_archive       Oracle Linux 8 Archive (x86_64)             disabled
...output omitted...
```
&#8195;&#8195;`yum config-manager`命令可用于启用或禁用存储库。为启用存储库，该命令将`enabled`参数设为`1`。示例如下：
```
[root@redhat8 ~]# yum config-manager --enable rhel-8-server-debug-rpms
```
&#8195;&#8195;要启用对新的第三方存储库的支持，可在`/etc/yum.repos.d/`目录中创建一个文件。存储库配置文件必须以`.repo`扩展名结尾。存储库定义包含存储库的`URL`和名称，也定义是否使用`GPG`检查软件包签名；如果是，则还检查`URL`是否指向受信任的`GPG`密钥。示例：
```
[root@redhat8 ~]# ls -l /etc/yum.repos.d
total 80
-rw-r--r--. 1 root root  1355 May 14 16:58 epel.repo
-rw-r--r--. 1 root root 16402 Aug 26  2019 public-yum-ol7.repo
-rw-r--r--. 1 root root   358 Jul 19  2020 redhat.repo
```
#### 创建Yum存储库
使用`yum config-manager`命令来创建Yum存储库。示例：
```
# yum config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/
docker-ce.reporepo saved to /etc/yum.repos.d/docker-ce.repo
```
&#8195;&#8195;修改此文件，以提供`GPG`密钥的自定义值和位置所示。密钥存储在远程存储库站点上的不同位置，如`http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8`。管理员应将该密钥下载到本地文件，而不是让`yum`从外部来源检索该密钥。例如：
```
[EPEL]
name=EPEL 8
baseurl=https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
```
#### 本地存储库的RPM配置软件包
&#8195;&#8195;一些存储库将一个配置文件和`GPG`公钥作为`RPM`软件包的一部分提供，该软件包可以使用`yum localinstall`命令来下载和安装。例如，`Extra Packages for Enterprise Linux (EPEL)`提供红帽不支持的、但与RHEL兼容的软件。以下命令将安装RHEL8 EPEL存储库软件包：
```
[root@redhat8 ~]# rpm --import \
> http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8
[root@redhat8 ~]# yum install \
> https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscr
iption-manager to register.Extra Packages for Enterprise Linux 7 - x86_64      3.0 kB/s | 7.6 kB     00:02    
Latest Unbreakable Enterprise Kernel Release 5 for  1.5 kB/s | 3.0 kB     00:01    
Oracle Linux 8 Latest (x86_64)                      1.5 kB/s | 3.6 kB     00:02    
epel-release-latest-8.noarch.rpm                    5.4 kB/s |  23 kB     00:04    
Dependencies resolved.
====================================================================================
 Package              Arch           Version             Repository            Size
====================================================================================
Upgrading:
 epel-release         noarch         8-15.el8            @commandline          23 k
Transaction Summary
====================================================================================
Upgrade  1 Package

Total size: 23 k
Is this ok [y/N]: Y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                            1/1 
  Running scriptlet: epel-release-8-15.el8.noarch                               1/1 
  Upgrading        : epel-release-8-15.el8.noarch                               1/2 
warning: /etc/yum.repos.d/epel.repo created as /etc/yum.repos.d/epel.repo.rpmnew
  Cleanup          : epel-release-7-14.noarch                                   2/2 
  Running scriptlet: epel-release-7-14.noarch                                   2/2 
  Verifying        : epel-release-8-15.el8.noarch                               1/2 
  Verifying        : epel-release-7-14.noarch                                   2/2 
Installed products updated.
Upgraded:
  epel-release-8-15.el8.noarch                                                      
Complete!
```
&#8195;&#8195;配置文件通常在一个文件中列举多个存储库引用。每一存储库引用的开头为包含在方括号里的单一词语名称。示例如下：
```
[root@redhat8 ~]# cat /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
baseurl=http://download.example/pub/epel/7/$basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch&infra
=$infra&content=$contentdirfailovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
...output omitted...
```
&#8195;&#8195;先安装`RPM GPG`密钥，再安装签名的软件包。这将验证软件包是否属于已经导入的密钥。否则，`yum`命令会因为缺少密钥而失败。可以通过`--nogpgcheck`选项忽略缺少的`GPG`密钥，但这可能会导致伪造或不安全的软件包被安装到系统上。
## 管理软件包模块流
### 应用流简介
&#8195;&#8195;RHEL8引入了应用流的概念。现在可同时提供发行版随附的多个版本的用户空间组件。它们可能比核心操作系统软件包更新得更频繁。这可以更灵活地自定义RHEL，而不会影响平台或特定部署的底层稳定性。    
&#8195;&#8195;从传统上看，管理应用软件包的备用版本及其相关软件包意味着为每个不同版本维护不同的存储库。如果开发人员想要最新版本的应用，而管理员希望获得该应用的最稳定版本，便会造成一种难以管理的繁琐局面。RHEL8中运用一种称为模块化的新技术简化了这个过程。模块化允许单个存储库承载应用软件包及其依赖项的多个版本。RHEL8内容通过两个主要的软件存储库进行分发，分别为`BaseOS`和`AppStream`(应用流)：
- `BaseOS`存储库以`RPM`软件包的形式为RHEL提供核心操作系统内容。BaseOS组件的生命周期与之前RHEL发行版中的内容相同
- 应用流存储库提供具有不同生命周期的内容，作为模块和传统软件包。应用流包含系统的必要部分，以及以前作为红帽软件集合的一部分以及其他产品和程序提供的各种应用。应用流存储库包含两种类型的内容：
    - 模块和传统的`RPM`软件包。模块描述了属于一个整体的一组`RPM`软件包。模块可以包含多个流，使多个版本的应用可供安装。启用模块流后，系统能够访问该模块流中的`RPM`软件包 

### 模块
&#8195;&#8195;模块是一组属于一个整体的、协调一致的 RPM 软件包。通常，这是围绕软件应用或编程语言的特定版本进行组织的。典型的模块可以包含应用的软件包、应用特定依赖库的软件包、应用文档的软件包，以及帮助器实用程序的软件包。
#### 模块流
&#8195;&#8195;每个模块可以具有一个或多个模块流，其包含不同版本的内容。每个流独立接收更新。模块流可以视为应用流物理存储库中的虚拟存储库。对于每个模块，只能启用其中一个流并提供它的软件包。
#### 模块配置文件
&#8195;&#8195;每个模块可以有一个或多个配置文件。配置文件是要为特定用例一起安装的某些软件包的列表，这些用例包括服务器、客户端、开发或最小安装等。安装特定的模块配置文件只是从模块流安装一组特定的软件包。可以随后正常安装或卸载软件包。如果未指定配置文件，模块将安装它的默认配置文件。
### 使用Yum管理模块
&#8195;&#8195;Yum版本4是RHEL8的增加了对应用流新模块化功能的支持。为处理模块化内容，添加了`yum module`命令。否则，`yum`很大程度上会像常规软件包一样处理模块。
#### 列出模块
使用`yum module list`显示可用模块的列表：
```
[root@redhat8 ~]# yum module list
Extra Packages for Enterprise Linux Modular 8 - x86_64
Name                 Stream           Profiles Summary                              
389-directory-server next             minimal, 389 Directory Server                 
                                       default 
389-directory-server stable           minimal, 389 Directory Server                 
                                       legacy, 
                                       default 
                                       [d]     
...output omitted...
```
列出特定模块的模块流并检索其状态：
```
[root@redhat8 ~]# yum module list nodejs
Extra Packages for Enterprise Linux Modular 8 - x86_64
Name         Stream       Profiles                           Summary                
nodejs       13           development, minimal, default      Javascript runtime     
nodejs       16-epel      development, minimal, default      Javascript runtime     

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
显示模块的详细信息：
```
[root@redhat8 ~]# yum module info nodejs
Name        : nodejs
Stream      : 13
Version     : 820200419230336
Context     : 9edba152
Profiles    : development, minimal, default
Repo        : epel-modular
Summary     : Javascript runtime
Description : Node.js is a platform built on Chrome''s JavaScript runtime for easily
 building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.            : This is the 13.x development stream and may not be suitible for produc
tion workloads.Artifacts   : c-ares-0:1.16.0-1.module_el8+8692+52300fb6.src
            : c-ares-0:1.16.0-1.module_el8+8692+52300fb6.x86_64
            : c-ares-debuginfo-0:1.16.0-1.module_el8+8692+52300fb6.x86_64
            ...output omitted...
...output omitted...
```
&#8195;&#8195;若不指定模块流，`yum module info`将显示使用默认流的模块的默认配置文件所安装的软件包列表。使用`module-name:stream`格式来查看特定的模块流。添加`--profile`选项可显示有关各个模块的配置文件所安装的软件包的信息。例如：
```
[root@redhat8 ~]# yum module info --profile perl:5.24
```
#### 启用模块流和安装模块
&#8195;&#8195;必须启用模块流才能安装其模块。为了简化此过程，在安装模块时，它将根据需要启用其模块流。可以使用`yum module enable`并提供模块流的名称来手动启用模块流。对于给定的模块，仅可启用一个模块流。启用其他模块流将禁用原始的模块流。

&#8195;&#8195;使用默认流和配置文件安装模块(运行`yum install @perl`效果一样。`@`表示法告知`yum`参数是模块名称而非软件包名称)：
```
[root@redhat8 ~]# yum module install zabbix
```
验证模块流和已安装配置文件的状态：
```
[root@redhat8 ~]# yum module list nginx
Extra Packages for Enterprise Linux Modular 8 - x86_64
Name              Stream              Profiles            Summary                   
nginx             1.20                common              nginx webserver           
nginx             mainline            common              nginx webserver           

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
#### 删除模块和禁用模块流
&#8195;&#8195;删除模块会删除当前启用的模块流的配置集所安装的所有软件包，以及依赖于这些软件包的任何其他软件包和模块。从此模块流安装的软件包如果未在其配置文件中列出，则会保持安装在系统上，可以手动删除。注意事项：
-  删除模块和切换模块流可能会有点棘手。切换为模块启用的流相当于重置当前流并启用新流。它不会自动更改任何已安装的软件包，必须手动来完成。
- 建议不要直接安装与当前所安装的模块流不同的模块流，因为升级脚本可能会在安装期间运行，从而破坏原始模块流。这可能会导致数据丢失或其他配置问题

要删除已安装的模块：
```
[root@redhat8 ~]# yum module remove nginx
```
&#8195;&#8195;删除模块后，其模块流仍然为启用状态。使用命令`yum module list`验证模块流是否仍处于启用状态。要禁用模块流： 
```
[root@redhat8 ~]# yum module disable nginx
Dependencies resolved.
====================================================================================
 Package            Arch              Version              Repository          Size
====================================================================================
Disabling module streams:
 nginx                                                                             
Transaction Summary
====================================================================================
Is this ok [y/N]: y
Complete!
[root@redhat8 ~]# yum module list nginx
Extra Packages for Enterprise Linux Modular 8 - x86_64
Name             Stream                 Profiles           Summary                  
nginx            1.20 [x]               common             nginx webserver          
nginx            mainline [x]           common             nginx webserver          

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
#### 切换模块流
&#8195;&#8195;切换模块流通常需要将内容升级或降级到不同版本。为确保顺利切换，应首先删除模块流提供的模块。这将删除模块的配置文件所安装的所有软件包，以及这些软件包依赖的任何模块和软件包。

为列出从模块安装的软件包，在下面的示例中安装了`postgresql:9.6`模块(官方示例)：
```
[user@host ~]$ sudo yum module info postgresql | grep module+el8 | \
sed 's/.*: //g;s/\n/ /g' | xargs yum list installed
Installed Packages
postgresql.x86_64          9.6.10-1.module+el8+2470+d1bafa0e   @rhel-8.0-for-x86_64-appstream-rpms
postgresql-server.x86_64   9.6.10-1.module+el8+2470+d1bafa0e   @rhel-8.0-for-x86_64-appstream-rpms
```
删除在上一个命令中列出的软件包。标记要卸载的模块配置文件(官方示例)：
```
[user@host ~]$ sudo yum module remove postgresql
...output omitted...
Is this ok [y/N]: y
...output omitted...
Removed:
  postgresql-server-9.6.10-1.module+el8+2470+d1bafa0e.x86_64   libpq-10.5-1.el8.x86_64  postgresql-9.6.10-1.module+el8+2470+d1bafa0e.x86_64
Complete
```
删除模块配置文件后，重置模块流。使用`yum module reset`命令重置模块流(官方示例)：
```
[user@host ~]$ sudo yum module reset postgresql
=================================================================
 Package       Arch             Version     Repository      Size
=================================================================
Resetting module streams:
postgresql                      9.6
Transaction Summary
=================================================================
Is this ok [y/N]: y
Complete!
```
要启用其他模块流并安装模块(官方示例)：
```
[user@host ~]$ sudo yum module install postgresql:10
```
&#8195;&#8195;将启用新的模块流，并禁用当前的流。可能需要更新或降级先前模块流中未在新配置文件中列出的软件包。必要时，可使用`yum distro-sync`来执行此任务。此外，也可能会有从先前模块流中保持安装的软件包，可通过`yum remove`删除它们。
## 练习