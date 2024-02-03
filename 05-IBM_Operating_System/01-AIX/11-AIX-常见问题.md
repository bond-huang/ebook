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

## 磁盘问题
### rootvg磁盘missing
&#8195;&#8195;rootvg有两块磁盘，都是通过vscsi过来的，vios端路径全部丢失，但是已经恢复了，vioc系统中还是现实missing昨天，部分lv处于stale状态。

如果丢失磁盘上有lg_dumplv，需要先修改dump：
```
# sysdumpdev -P -p /dev/sysdumpnull
```
确认磁盘路径是恢复的，激活一下VG：
```
# varyonvg rootvg
```
磁盘状态恢复正常，镜像自动同步，等待完成即可，改回dump配置：
```
# sysdumpdev -P -p /dev/lg_dumplv
```
如果直接修改磁盘状态是不成功的(即使修改了dump配置)：
- smit chpv
- Change Characteristics of a Physical Volume
- Physical volum STATE from not active to active

### Migratepv命令失败
参考链接：[IBM Support Migratepv command fail](https://www.ibm.com/support/pages/migratepv-command-fail)

### rootvg被锁
执行命令时候提示rootvg被锁：
```
# lsvg -l rootvg
0516-1201 lsvg: Warning: Volume group rootvg is locked. This command
        will continue retries until lock is free.  If lock is inadvertent
        and needs to be removed, execute 'chvg -u rootvg'.
```
执行解锁命令发现卡住半天没反应：
```
# chvg -u rootvg

```
查看进程发现很多lsvg进程卡住了：
```
# ps -ef |grep vg
    root  7077894 10485878   0   Mar 08      -  1:35 /usr/sbin/lsvg -p rootvg
    ......
    root 41746566 41091104   0   Jun 04      -  1:09 /usr/sbin/lsvg -p rootvg
    root 43122796 41287680   0 17:04:24  pts/5  0:00 /bin/ksh /etc/chvg -u rootvg
```
kill 掉除chvg的所有
```
# kill -9 41746566
# ps -ef |grep vg
    root 38863020 22413336   0 17:12:46  pts/6  0:00 grep vg
    root 43122796 41287680   0 17:04:24  pts/5  0:00 /bin/ksh /etc/chvg -u rootvg
```
但是chvg命令还是没反应，继续kill掉chvg：
```
# kill -9 43122796
```
可以看到被干掉了：
```
# chvg -u rootvg
^CKilled
```
再去执行lsvg相关命令就可以使用了。
## 引导问题
### savebase failed
当解除rootvg镜像时候报错：
```
0516-1734 rmlvcopy:Warning,savebase failed.Please manually run 'savebase' berfore rebooting.
```
可能原因引导只在一块盘中，而解除镜像的盘正好是引导所在的盘，把引导添加进去即可：
```
# bosboot -ad /dev/hdiskX
# bootlist -m normal hdiskX
# savebase
```
### 删除磁盘报错
当`rmdev`删除某个磁盘时候报错:
```
rmdev 0514-508 Connot save the base customized infomation on /dev/ipldevice
```
原因应该也是hd5里面的boot信息没有了，处理方法：
```
# bosboot -ad /dev/hdiskX
# bootlist -m normal hdiskX
# synclvodm -P -v rootvg
```
### hd5丢失
执行某些命令时候可能有报错如下：
```
0514-508 Cannot save the base customized information on /dev/ipldevice.
0514-609 Unable to save the base customized information on /dev/ipldevice.
```
检查LV，可能是hd5引导lv被删掉了，重建lv命令：
```
# mklv -y hd5 -t boot -a e rootvg 1
```
如果创建失败，先删掉odm库信息：
```
# odmdelete -q name=hd5 -o CuDv
# odmdelete -q name=hd5 -o CuAt
```
创建成功后同步lv：
```
# synclvodm -P -v rootvg
```
然后查看lv信息：
```
# lslv -m hd5
```
查看引导盘信息：
```
# bootinfo -b 
```
里面会现实hd5所在在盘hdiskX，往里面写入引导信息：
```
# bosboot -ad /dev/hdiskX
```
如果写入失败，加入lv名称：
```
# bosboot -ad hdiskX -l hd5  
```
如果rootvg是双盘镜像，镜像lv到另外一个块盘，写入引导信息：
```
# bosboot -ad hdiskXX -l hd5  
```
最后添加引导列表信息：
```
# bootlist -m normal hdiskX hdiskXX
```
## 删文件报错
### rm命令报错
当使用rm命令删除大量文件时候，会有如下报错：
```
# rm testfile* 
ksh: /usr/bin/rm: 0403-027 The parameter list is too long. 
```
#### 报错说明
&#8195;&#8195;此错误表示已达到ARG_MAX值。 ARG_MAX的值为24,576 字节，`在/usr/include/sys/limits.h`文件中设置。此值无法更改，因为它已编译到列出的命令中。 这是传递给正在运行的命令的字节数，在此实例中表示正在执行的文件的名称。

#### 解决方法一
先查找然后递归方式删除，示例如下：
```
# find . -xdev -exec rm -r {} \;
```
#### 解决方法二
或者修改参数：
```
# smitty chgsys 
ARG/ENV list size in 4K byte blocks  [6] 
change ARG/ENV list size in 6K byte blocks to 1024 
```
或者使用命令：
```
chdev -l sys0 -a ncargs=64
```
删除完成后，修改回来即可
## 文件系统问题
官方参考链接：[Troubleshooting file systems](https://www.ibm.com/docs/en/aix/7.3?topic=systems-troubleshooting-file)
### Superblock问题
#### superblock corrupted
报错示例：
```
0506-945 The /dev/hd10opt JFS2 filesystem superblock is corrupted.
0507-545 Connot read superblock on device /dev/hd10opt
```
可能重启即可修复，官方参考修复方法链接：
- [Fixing a corrupted magic number in the file system superblock](https://www.ibm.com/docs/en/aix/7.2?topic=tfs-fixing-corrupted-magic-number-in-file-system-superblock)
- [Repairing Corrupt File Systems or File System Log Devices](https://www.ibm.com/support/pages/repairing-corrupt-file-systems-or-file-system-log-devices)

相关APAR：
- [IY60010: FSCK ERROR ON > 1TB FILESYSTEM](https://www.ibm.com/support/pages/apar/IY60010?mhsrc=ibmsearch_a&mhq=superblock%20%20corrupted)
- [IV69453: FSCK DOES NOT REPORT CORRUPTION TO A SPECIFIC SUPERBLOCK FIELD](https://www.ibm.com/support/pages/apar/IV69453?mhsrc=ibmsearch_a&mhq=superblock%20%20corrupted)
- [IV54476: J2 FILESYSTEM GETS CORRUPTED AFTER ROLLBACK OF EXTERNAL SNAPSHOT APPLIES TO AIX 6100-09](https://www.ibm.com/support/pages/apar/IV54476?mhsrc=ibmsearch_a&mhq=superblock%20%20corrupted)
- [IV57152: J2 FILESYSTEM GETS CORRUPTED AFTER ROLLBACK OF EXTERNAL SNAPSHOT APPLIES TO AIX 7100-02](https://www.ibm.com/support/pages/apar/IV57152?mhsrc=ibmsearch_a&mhq=superblock%20%20corrupted)

## 网络问题
### IP配置问题
&#8195;&#8195;配置的IP重启后没有了，可能是ifconfig命令配置的，不会写入ODM库里面，当然也有可能是其他原因。使用`mktcpip`命令配置的IP才会在`lsattr`里面查看到，才会写入ODM库。例如`mktcpip`命令配置en0查看属性可以看到IP信息：
```sh
lsattr -El en0
```
查询ODM库信息：
```sh
odmget -q name=en0 CuAt
```
直接筛选出IP地址：
```sh
lsattr -E -l en0 -F "value" -a netaddr
```
直接筛选出掩码信息：
```sh
lsattr -E -l en0 -F "value" -a netmask
```
如果是`mktcpip`命令重启后依旧没有排查思路：
- 配置后使用上述命令查看信息是否正常写入，如果没有，可能是bug或系统问题或网络设备问题导致
- 检查`/etc/rc.net.serial`文件，里面会查询网卡配置，获取IP掩码的信息启动时候进行配置，如果上面命令都查看不到信息，这个文件里面的命令也不会获取IP信息
- 检查/`etc/inittab`文件，是否有自定义的启动项影响IP配置

## 其它问题
### 错误代码0403-059
执行某些命令时候会收到`0403-059`报错，示例如下：
```
/etc/reducevg[436]: Mb_pending:0403-059 There cannot be more than 9 levels of recursion
```
&#8195;&#8195;这不是AIX问题，是如何获取`.profile`的问题，如果是要到root用户，使用`su root`而不是使用`su - root`可以解决问题。官方参考链接：[Receive 0403-059 There cannot be more than 9 levels of recursion during Informix Install.](https://www.ibm.com/support/pages/receive-0403-059-there-cannot-be-more-9-levels-recursion-during-informix-install)
## 待补充