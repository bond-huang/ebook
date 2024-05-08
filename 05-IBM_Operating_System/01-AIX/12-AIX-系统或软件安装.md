# AIX-系统或软件安装
记录系统安装及一些常见的软件安装方法。
## 系统安装
### 系统差异
PDF文档链接：
- [IBM AIX Version 6.1 Differences Guide](https://www.redbooks.ibm.com/abstracts/sg247559.html?Open)
- [IBM AIX Version 7.1 Differences Guide](https://www.redbooks.ibm.com/abstracts/sg247910.html#:~:text=AIX%20Version%207.1%20introduces%20many%20new%20features%2C%20including%3A,you%20can%20explore%20them%20all%20in%20this%20publication.)

## 软件安装
## DNF配置
AIX使用DNF官方参考文档：
- [DNF is now available on AIX Toolbox](https://community.ibm.com/community/user/power/blogs/sangamesh-mallayya1/2021/05/28/dnf-is-now-available-on-aix-toolbox)
- [Creating local repo with DNF and AIX Toolbox Media Image](https://community.ibm.com/community/user/power/blogs/sangamesh-mallayya1/2022/02/09/creating-local-repo-with-dnf-and-aix-toolbox-media?CommunityKey=10c1d831-47ee-4d92-a138-b03f7896f7c9)

### 安装OpenSSL和OpenSSH
官方参考链接：[Downloading and Installing or Upgrading OpenSSL and OpenSSH](https://www.ibm.com/support/pages/downloading-and-installing-or-upgrading-openssl-and-openssh?mhsrc=ibmsearch_a&mhq=OPENSSH)
#### 下载安装包
需要下的安装包:
- OpenSSH_8.1.102.2104.tar.Z
- openssl-1.1.1.1200.tar.Z

下载链接：
- OpenSSH: [https://www.ibm.com/resources/mrs/assets?source=aixbp&S_PKG=openssh](https://www.ibm.com/resources/mrs/assets?source=aixbp&S_PKG=openssh)
- OpenSSL: [https://www.ibm.com/resources/mrs/assets?source=aixbp&S_PKG=openssl](https://www.ibm.com/resources/mrs/assets?source=aixbp&S_PKG=openssl)
- All available packages: [https://www.ibm.com/resources/mrs/assets?source=aixbp](https://www.ibm.com/resources/mrs/assets?source=aixbp)

#### 升级安装步骤
创建临时目录，上传安装包：
```
# mkdir /tmp/newOpenSSL
# mkdir /tmp/newOpenSSH
```
备份SSH配置文件夹：
```
# cp -pr /etc/ssh /etc/ssh_backup
```
解压OpenSSL：
```
# cd /tmp/newOpenSSL
# uncompress openssl-1.1.1.1200.tar.Z 
# tar -xvf openssl-1.1.1.1200.tar
# cd <newly created OpenSSL directory if one was created>
```
安装OpenSSL：
- smitty install
- 选择Install and Update Software->Install Software
- INPUT device / directory for software  \[.] 输入当前目录.
- SOFTWARE to install \[_all_latest]  按F4、/然后查找ssl匹配项，选中回车
- "ACCEPT new license agreements?"  \[yes] (Tab键)
- 安装完成后的sshd命令位于`/usr/sbin`目录，配置文件位于`/etc/ssh`目录

解压OpenSSH：
```
# cd /tmp/newOpenSSH
# uncompress OpenSSH_8.1.102.2104.tar.Z
# tar -xvf OpenSSH_8.1.102.2104.tar
# cd <newly created OpenSSH directory if one was created>
```
安装OpenSSH：
- smitty install
- 选择Install and Update Software->Install Software
- INPUT device / directory for software  \[.] 输入当前目录.
- SOFTWARE to install \[_all_latest]   按F4、/然后查找SSH匹配项，选中回车
- "ACCEPT new license agreements?"  \[yes]  (Tab键)

检查服务状态：
```
# lssrc -g sshd
```
备份新的版本配置：
```
# cp -p ssh_config ssh_config.orig_<today's_date>
# cp -p sshd_config sshd_config.orig_<today's_date>
```
恢复ssh host keys：
```
# cd /etc/ssh_backup
# cp -pr cp ssh_host_*_key*  /etc/ssh
```
重启SSHD服务：
```
# startsrc -s sshd   
# stopsrc -s sshd
```
### 安装bash
AIX的bash官方下载地址：[Bash 5.2.15 for aix](https://public.dhe.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/bash/bash-5.2.15-1.aix7.1.ppc.rpm)
### 待补充