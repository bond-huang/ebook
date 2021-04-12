# AIX-常见问题
记录一些常见的AIX系统问题。
## 系统升级问题
### 升级时空间报错
升级版本过程中报错：
```
AIX 6.1 Not enough disk space for sni_lv.
```
问题说明：
```
sni_lv is an lv that is temporarily created during
the install but needs 256M of space to be created. Encouraged the 
customer to reduce the size of a fs to have at least 256 MB free in the 
rootvg, then retry the installation. 
```
解决方法：       
保证rootvg里面至少有256M的空余空间，再升级过程中会创建一个临时文件需要需要空间。

### 升级bosboot报错
升级AIX系统过程中报错：
```
0503-409 installp: bosboot verification starting... 
0503-497 installp: An error occurred during bosboot verification 
processing. 
```
查看boot信息重建bosboot命令都可以执行：
```
bootlist -m normal -o
bootlist -v
bosboot -ad /dev/hdisk0
bosboot -ad /dev/hdisk1
bootlist -m normal hdisk0 hdisk1
```
再次尝试升级还是同样报错，解决方法：

如果hdisk0是引导设备，那么下面命令输出应该是这样的：
```
#ls -l /dev/rhdisk0 /dev/ipldevice 
crw------- 2 root system 20, 0 Apr 07 2004 /dev/ipldevice 
crw------- 2 root system 20, 0 Apr 07 2004 /dev/rhdisk0 
```
如果`/dev/ipdevice`丢失或者不正确, 重建链接即可，示例：
```
#ln -f /dev/rhdisk0 /dev/ipldevice
```
再次尝试升级成功。

### 系统升级提示差JAVA包
AIX7232升级到AIX7236最后报了一个包安装不上，提示需要JAVA包：
```
  Requisite Failures
    ------------------
    SELECTED FILESETS: The following is a list of filesets that you asked to
    install. They cannot be installed until all of their requisite filesets
    are also installed. See subsequent lists for details of requisites.

    sysmgt.cfgassist 7.2.3.16                # Configuration Assistant

    MISSING REQUISITES: The following filesets are required by one or more of the selected filesets listed above. They are not currently installed and could not be found on the installation media.

    Java8_64.sdk                             # Base Level Fileset
```
下载Java8_64.jre和Java8_64.sdk后安装，先安装Java8_64.jre，示例：
```
# gunzip Java8_64.jre.tar.gz
# tar -xvf Java8_64.jre.tar
# inutoc .
# smit installp
```
下载地址：[IBM Java for AIX Reference: Service Information and Download Guide](https://www.ibm.com/support/pages/node/681265)

### 系统升级后SSH用不了
AIX7232升级到AIX7236后，使用ssh或者scp时候都会报错：
```
/etc/ssh/sshd_config line 40: Bad SSH2 cipher spec \
'aes128-cbc,aes192-cbc,aes256-cbc,blowfish-cbc,arcfour,\
aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128'.
```
不止cipher，还有MAC也有此样报错，用下面命令逐一检查cipher和MAC：
```
# sshd -t -o Ciphers=aes128-cbc
# sshd -t -o MACs=hmac-ripemd160
```
有报错提示bad的，从ssh_config文件中完全删除密码行。验证新密码列表：
```
# ssh -Q cipher
# ssh -Q MAC
``` 
官方说明：[IBM AIX: 'Bad SSH2 cipher spec' Error Upon Starting sshd](https://www.ibm.com/support/pages/node/6406544)

运行上面命令同时，还会发现如下报错：
```
Deprecated option RhostsAuthentication 
```
启动sshd服务时候也会报，这个是新系统已经废弃选项，在文件sshd_config中注释掉即可。
### 提示用户组非已知
报错示例如下：
```
sysck: 3001-038 The name mail is not a known group for entry /usr/bin/bellmail.
sysck: 3001-003 A value must be specified for group for entry /usr/bin/bellmail.
...
sysck: 3001-038 The name printq is not a known group for entry /usr/bin/chquedev.
sysck: 3001-003 A value must be specified for group for entry /usr/bin/chquedev.
sysck: 3001-017 Errors were detected validating the files
        for package bos.rte.printers.
        
0503-464 installp:  The installation has FAILED for the "usr" part
        of the following filesets:
        bos.rte.printers 7.2.3.17
installp:  Cleaning up software for:
        bos.rte.printers 7.2.3.17
```
创建提示的用户组再执行升级即可：
- mail用户组：name=mail,id=6,admin=true
- printq用户组：name=mail,id=6,admin=true,users=lp

## 命令反应慢
### 系统资源问题
现象：
- ping此机器ip会有比较慢，有几毫秒
- ssh过去时候等半天才弹出输密码界面
- 登录进去后敲任何命令都比较慢
- 系统CPU资源没爆但是使用比较高
- rootvg有磁盘使用率100%

原因：有大文件在持续向rootvg里面磁盘写入，并且只是写入其中一块，导致其100%，所以系统慢。

### 系统资源空闲
&#8195;&#8195;系统正常运行，但是登录特别慢，登录成功后输入命令反应也特别慢，系统资源也不是很紧张，文件系统也没爆。刚开始怀疑DNS问题，检查没发现问题，再检查日志有大量以下报错：
```
3A30359F   1030230010 T S init           SOFTWARE PROGRAM ERROR 
```
报错详细内容：
```
Detail Data 
SOFTWARE ERROR CODE 
Command is respawning too rapidly. Check for possible errors. 
COMMAND 
id: audit "/usr/sbin/audit start # System Auditing" 
```
&#8195;&#8195;检查`/etc/inittab`文件，关于此命令的条目`Aciton`项设置是`respawn`,此设置表示如果进程不存在，启动进程；进程终止后重新启动。结合报错信息:Command is respawning too rapidly,说明执行过于频繁导致其它命令反应比较慢，检查命令是否有问题。

### 待补充
