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
### 使用Yum管理软件包