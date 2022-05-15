# Oracle-安装与部署
官方软件下载链接：
- [甲骨文中国 Oracle软件下载](https://www.oracle.com/cn/downloads/)
- [Oracle yum](https://yum.oracle.com/index.html)
- [Oracle public-yum](https://public-yum.oracle.com/)

## Oracle数据库安装
官方安装guide：[Oracle Database 19c Install and Upgrade](https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html)
### 配置YUM源
为安装依赖包，配置Oracle的YUM源：
```
[root@redhat8 yum.repos.d]# wget http://public-yum.oracle.com/public-yum-ol7.repo
--2022-05-14 13:18:55--  http://public-yum.oracle.com/public-yum-ol7.repo
Resolving public-yum.oracle.com (public-yum.oracle.com)... 23.195.249.95, 2600:1409:3000:38b::2a7d, 2600:1409:3000:382::2
a7dConnecting to public-yum.oracle.com (public-yum.oracle.com)|23.195.249.95|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://public-yum.oracle.com/public-yum-ol7.repo [following]
--2022-05-14 13:18:55--  https://public-yum.oracle.com/public-yum-ol7.repo
Connecting to public-yum.oracle.com (public-yum.oracle.com)|23.195.249.95|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16402 (16K) [text/plain]
Saving to: ‘public-yum-ol7.repo’
public-yum-ol7.repo            100%[=================================================>]  16.02K  --.-KB/s    in 0s      
2022-05-14 13:18:57 (128 MB/s) - ‘public-yum-ol7.repo’ saved [16402/16402]
```
### Linux系统安装RPM
Oracle官网下载的两个RPM：
- oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
- oracle-database-ee-19c-1.0-1.x86_64.rpm

第一个RPM下载地址：[Latest packages for Oracle Linux 7 (x86_64)](https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/index.html)

#### 预安装RPM
首先安装`oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm`，提示：
```
[root@redhat8 Downloads]# rpm -ivh oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
warning: oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID ec551f03: NOKEY
error: Failed dependencies:
	compat-libcap1 is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	compat-libstdc++-33 is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	ksh is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	libaio-devel is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	libstdc++-devel is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	sysstat is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
	xorg-x11-utils is needed by oracle-database-preinstall-19c-1.0-1.el7.x86_64
```
根据提示安装依赖包(一条命令解决发现报错，一个一个安装)：
```
[root@redhat8 ~]# yum install compat-libcap1
[root@redhat8 ~]# yum install compat-libstdc++-33
[root@redhat8 ~]# yum install ksh
[root@redhat8 ~]# yum install libaio-devel
[root@redhat8 ~]# yum install libstdc++-devel
[root@redhat8 ~]# yum install sysstat
```
Oracle数据库预安装RPM自动配置Linux，例如用户等：
- 自动下载和安装安装`Oracle Grid Infrastructure`和Oracle数据库所需的任何其他RPM包，并解决任何依赖关系
- 创建一个`oracle`用户，并为该用户创建`oraInventory`(oinstall)和`OSDBA`(dba)组
- 根据需要，将`sysctl.conf`设置、系统启动参数和驱动程序参数设置为基于`Oracle数据库预安装RPM程序建议的值
- 设置硬资源和软资源限制
- 设置其他推荐参数，具体取决于安装系统环境的内核版本
- 在Linux x86_64机器的内核中设置`numa=off`

安装示例如下：
```
[root@redhat8 Downloads]# rpm -ivh oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:oracle-database-preinstall-19c-1.################################# [100%]
```
#### RPM包安装Oracle
安装示例：
```
[root@redhat8 ~]# rpm -ivh --nodigest --nofiledigest /tmp/oracle-database-ee-19c-1.0-1.x86_64.rpm
```
安装报错，研究中......
### 安装中出现的问题
#### 依赖包安装问题1
安装依赖包时候提示错误：
```
Curl error (37): Couldn't read a file:// file for file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle [Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle]
```
解决方法：
```
[root@redhat8 rpm-gpg]# wget http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol7 -O /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
--2022-05-14 13:29:05--  http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol7
Resolving public-yum.oracle.com (public-yum.oracle.com)... 23.195.249.95, 2600:1409:3000:38b::2a7d, 2600:1409:3000:382::2
a7dConnecting to public-yum.oracle.com (public-yum.oracle.com)|23.195.249.95|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol7 [following]
--2022-05-14 13:29:06--  https://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol7
Connecting to public-yum.oracle.com (public-yum.oracle.com)|23.195.249.95|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1011 [text/plain]
Saving to: ‘/etc/pki/rpm-gpg/RPM-GPG-KEY-oracle’
/etc/pki/rpm-gpg/RPM-GPG-KEY-o 100%[=================================================>]    1011  --.-KB/s    in 0s      
2022-05-14 13:29:07 (13.5 MB/s) - ‘/etc/pki/rpm-gpg/RPM-GPG-KEY-oracle’ saved [1011/1011]
```
再次安装即可。问题解决参考链接：[Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle error while installing Oracle Pre-install RPM](https://www.funoracleapps.com/2020/03/couldnt-open-file-etcpkirpm-gpgrpm-gpg.html)。

#### 依赖包安装问题2
除了`libstdc++-devel`其他都安装成功，报错：
```
[root@redhat8 ~]# yum install libstdc++-devel
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:05:26 ago on Sat 14 May 2022 01:33:49 PM EDT.
Error: 
 Problem: cannot install both libstdc++-4.8.5-44.0.3.el7.x86_64 and libstdc++-8.2.1-3.5.el8.x86_64
  - package libstdc++-devel-4.8.5-44.0.3.el7.x86_64 requires libstdc++(x86-64) = 4.8.5-44.0.3.el7, but none of the provid
ers can be installed  - package aspell-12:0.60.6.1-21.el8.x86_64 requires libstdc++.so.6(CXXABI_1.3.9)(64bit), but none of the providers can 
be installed  - cannot install the best candidate for the job
  - problem with installed package aspell-12:0.60.6.1-21.el8.x86_64
(try to add '--allowerasing' to command line to replace conflicting packages or '--skip-broken' to skip uninstallable pac
kages or '--nobest' to use not only best candidate packages)
```
检查是系统中以及有一个版本：
```
[root@redhat8 Downloads]# rpm -qa |grep libstdc++-
compat-libstdc++-33-3.2.3-72.el7.x86_64
libstdc++-8.2.1-3.5.el8.x86_64
```
使用YUM安装怎么弄都不行，手动安装：
```
[root@redhat8 Downloads]# rpm -ivh libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm 
warning: libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 13d0a55d: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:libstdc++-devel-8.2.1-3.5.el8    ################################# [100%]
[root@redhat8 Downloads]# rpm -qa |grep libstdc++-
compat-libstdc++-33-3.2.3-72.el7.x86_64
libstdc++-8.2.1-3.5.el8.x86_64
libstdc++-devel-8.2.1-3.5.el8.x86_64
```
`libstdc++-devel`下载地址：[Libstdc++-devel Download for Linux (rpm, xbps)](https://pkgs.org/download/libstdc++-devel)
#### 安装Oracle报错1
报错如下：
```
[root@redhat8 Downloads]# rpm -ivh oracle-database-ee-19c-1.0-1.x86_64.rpm
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
	package oracle-database-ee-19c-1.0-1.x86_64 does not verify: no digest
```
使用`--nodigest`和`--nofiledigest`解决：
```
[root@redhat8 ~]# rpm -ivh --nodigest --nofiledigest /tmp/oracle-database-ee-19c-1.0-1.x86_64.rpm
```
官方参考链接：[rpm error "does not verify: no digest"](https://access.redhat.com/solutions/4460971)
#### 安装RPM报错1
报错示例如下：
```
'AttachHome' failed.
Exception in thread "main" java.lang.NullPointerException
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.main_helper(OiicBaseInventoryApp.java:706)
	at oracle.sysman.oii.oiic.OiicAttachHome.main(OiicAttachHome.java:696)
[SEVERE] An error occurred while registering the Oracle home. Verify logs in /var/log/oracle-database-ee-19c/results/oraInstall.log and /opt/oracle/oraInventory for more details and try again.warning: %post(oracle-database-ee-19c-1.0-1.x86_64) scriptlet failed, exit status 1
```

### 临时输出
```
[root@redhat8 ~]# rpm -ivh --nodigest --nofiledigest /tmp/oracle-database-ee-19c-1.0-1.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:oracle-database-ee-19c-1.0-1     ################################# [100%]
Exception java.lang.UnsatisfiedLinkError: /opt/oracle/product/19c/dbhome_1/oui/lib/linux64/liboraInstaller.so: libnsl.so.
1: cannot open shared object file: No such file or directory occurred..java.lang.UnsatisfiedLinkError: /opt/oracle/product/19c/dbhome_1/oui/lib/linux64/liboraInstaller.so: libnsl.so.1: cannot open shared object file: No such file or directory	at java.lang.ClassLoader$NativeLibrary.load(Native Method)
	at java.lang.ClassLoader.loadLibrary0(ClassLoader.java:1941)
	at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1857)
	at java.lang.Runtime.loadLibrary0(Runtime.java:870)
	at java.lang.System.loadLibrary(System.java:1122)
	at oracle.sysman.oii.oiip.osd.unix.OiipuUnixOps.loadNativeLib(OiipuUnixOps.java:388)
	at oracle.sysman.oii.oiip.osd.unix.OiipuUnixOps.<clinit>(OiipuUnixOps.java:130)
	at oracle.sysman.oii.oiip.oiipg.OiipgEnvironment.getEnv(OiipgEnvironment.java:201)
	at oracle.sysman.oii.oiix.OiixIniPair.instantiateEnvVars(OiixIniPair.java:299)
	at oracle.sysman.oii.oiix.OiixIniPair.updateValue(OiixIniPair.java:230)
	at oracle.sysman.oii.oiix.OiixIniPair.<init>(OiixIniPair.java:148)
	at oracle.sysman.oii.oiix.OiixIniFile.readFile(OiixIniFile.java:809)
	at oracle.sysman.oii.oiix.OiixIniFile.readIniFile(OiixIniFile.java:978)
	at oracle.sysman.oii.oiix.OiixIniFile.getProfileString(OiixIniFile.java:385)
	at oracle.sysman.oii.oiix.OiixOraparam.getOraparamProfileString(OiixOraparam.java:338)
	at oracle.sysman.oii.oiix.OiixOraparam.getOraparamProfileString(OiixOraparam.java:296)
	at oracle.sysman.oii.oiix.OiixOraparam.usePrereqChecker(OiixOraparam.java:416)
	at oracle.sysman.oii.oiic.OiicSessionContext.setVariables(OiicSessionContext.java:1325)
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.execute(OiicBaseInventoryApp.java:771)
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.main_helper(OiicBaseInventoryApp.java:690)
	at oracle.sysman.oii.oiic.OiicDetachHome.main(OiicDetachHome.java:420)
'DetachHome' failed.
Exception in thread "main" java.lang.NullPointerException
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.main_helper(OiicBaseInventoryApp.java:706)
	at oracle.sysman.oii.oiic.OiicDetachHome.main(OiicDetachHome.java:420)
Exception java.lang.NoClassDefFoundError: Could not initialize class oracle.sysman.oii.oiip.osd.unix.OiipuUnixOps occurre
d..java.lang.NoClassDefFoundError: Could not initialize class oracle.sysman.oii.oiip.osd.unix.OiipuUnixOps
	at oracle.sysman.oii.oiip.oiipg.OiipgEnvironment.getEnv(OiipgEnvironment.java:201)
	at oracle.sysman.oii.oiix.OiixIniPair.instantiateEnvVars(OiixIniPair.java:299)
	at oracle.sysman.oii.oiix.OiixIniPair.updateValue(OiixIniPair.java:230)
	at oracle.sysman.oii.oiix.OiixIniPair.<init>(OiixIniPair.java:148)
	at oracle.sysman.oii.oiix.OiixIniFile.readFile(OiixIniFile.java:809)
	at oracle.sysman.oii.oiix.OiixIniFile.readIniFile(OiixIniFile.java:978)
	at oracle.sysman.oii.oiix.OiixIniFile.getProfileString(OiixIniFile.java:385)
	at oracle.sysman.oii.oiix.OiixOraparam.getOraparamProfileString(OiixOraparam.java:338)
	at oracle.sysman.oii.oiix.OiixOraparam.getOraparamProfileString(OiixOraparam.java:296)
	at oracle.sysman.oii.oiix.OiixOraparam.usePrereqChecker(OiixOraparam.java:416)
	at oracle.sysman.oii.oiic.OiicSessionContext.setVariables(OiicSessionContext.java:1325)
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.execute(OiicBaseInventoryApp.java:771)
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.main_helper(OiicBaseInventoryApp.java:690)
	at oracle.sysman.oii.oiic.OiicAttachHome.main(OiicAttachHome.java:696)
'AttachHome' failed.
Exception in thread "main" java.lang.NullPointerException
	at oracle.sysman.oii.oiic.OiicBaseInventoryApp.main_helper(OiicBaseInventoryApp.java:706)
	at oracle.sysman.oii.oiic.OiicAttachHome.main(OiicAttachHome.java:696)
[SEVERE] An error occurred while registering the Oracle home. Verify logs in /var/log/oracle-database-ee-19c/results/oraI
nstall.log and /opt/oracle/oraInventory for more details and try again.warning: %post(oracle-database-ee-19c-1.0-1.x86_64) scriptlet failed, exit status 1



[root@redhat8 tmp]# rpm -ivh libnsl-2.28-164.el8.x86_64.rpm
warning: libnsl-2.28-164.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
error: Failed dependencies:
	glibc(x86-64) = 2.28-164.el8 is needed by libnsl-2.28-164.el8.x86_64


[root@redhat8 tmp]# rpm -ivh glibc-2.28-164.el8.x86_64.rpm
warning: glibc-2.28-164.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
error: Failed dependencies:
	glibc-common = 2.28-164.el8 is needed by glibc-2.28-164.el8.x86_64
	glibc-langpack = 2.28-164.el8 is needed by glibc-2.28-164.el8.x86_64

[root@redhat8 tmp]# rpm -qa |grep glibc
glibc-devel-2.28-42.el8.x86_64
glibc-2.28-42.el8.x86_64
glibc-headers-2.28-42.el8.x86_64
glibc-common-2.28-42.el8.x86_64
glibc-langpack-en-2.28-42.el8.x86_64


[root@redhat8 tmp]# rpm -qa |grep libnsl
libnsl2-1.2.0-2.20180605git4a062cf.el8.x86_64
```