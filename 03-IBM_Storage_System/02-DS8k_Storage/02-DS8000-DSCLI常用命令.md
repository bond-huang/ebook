# DS8000-DSCLI常用命令
&#8195;&#8195;DSCLI全称DS command-line interface，是管理DS8000系列存储的常用工具，DS8000的MC系统上有集成，也可以下载到个人终端上，在IBM fixcentral中有下载。DSCLI命令官方说明链接：
- [DS8880 8.5.4 CLI commands](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=interface-cli-commands)
- [DS8880 Command-line interface](https://www.ibm.com/docs/en/ds8880/8.5.4?topic=reference-command-line-interface)

## 登录使用
### HMC上使用
在HMC上的空白处右键，既有打开DSCLI选项，点击打开输入用户密码即可管理此DS8000存储。
### PC终端
&#8195;&#8195;从IBM fixcentral中下载DSCLI安装后，运行程序，输入`dscli`命令后根据提示输入IP地址、用户名及密码即可连上对于的DS8000存储，登录成功后会有存储名称提示，注意核对序列号避免出错。
## 常用命令
### 查看类命令
命令`lsrank`，示例如下：
```
dscli> lsrank
ID Group State datastate Array RAIDtype extpoolID stgtype
===========================================================
R0 0 Normal Normal A0 5 P0 fb
R1 0 Normal Normal A1 5 P0 fb
R2 0 Normal Normal A2 5 P0 fb
```
命令`lsarraysite`，示例如下：
```
dscli> lsarraysite
arsite DA Pair dkcap (10^9B) State Array
===========================================
S1 0 300.0 Assigned A32
S2 0 300.0 Assigned A15
S3 0 300.0 Assigned A30
S4 0 300.0 Assigned A13
```
命令`lsarray`，示例如下：
```
dscli> lsarray
Array State Data RAIDtype arsite Rank DA Pair DDMcap (10^9B)
==================================================================
A0 Assigned Normal 5 (7+P) S34 R0 18 400.0
A1 Assigned Normal 5 (6+P+S) S32 R1 18 400.0
A2 Assigned Normal 5 (7+P) S23 R2 5 300.0
A3 Assigned Normal 5 (6+P+S) S28 R3 7 300.0
```
命令`lsextpool`，示例如下：
```
dscli> lsextpool
Name ID stgtype rankgrp status availstor (2^30B) %allocated available reserved numvols
======================================================================================
Pool P0 fb 0 below 5160 82 330227 3072 527
Pool P1 fb 1 below 5160 82 330235 3072 527
```
命令`lsfbvol`，示例如下：
```
dscli> lsfbvol
Name ID accstate datastate configstate deviceMTM datatype extpool cap (2^30B) cap (10^9B) cap (blocks)
======================================================================================================
E980PRD_IBASE 1000 Online Normal Normal 2107-099 FB 520PV P0 70.0 75.2 146800640
E980PRD_IBASE 1001 Online Normal Normal 2107-099 FB 520PV P0 70.0 75.2 146800640
...
E980PRD_IASP 1200 Online Normal Normal 2107-A04 FB 520P P0 65.7 70.6 137822208
E980PRD_IASP 1201 Online Normal Normal 2107-A04 FB 520P P0 65.7 70.6 137822208
```
查看指定的volgrp的卷信息：
```
dscli> lsfbvol -volgrp v3
```
命令`lsddm`，示例如下：
```
dscli> lsddm
ID DA Pair dkcap (10^9B) dkuse arsite State
===============================================================================
IBM.2107-D02-04XXB/R1-P1-D1 0 800.0 array member S6 Normal
IBM.2107-D02-04XXB/R1-P1-D2 0 800.0 array member S1 Normal
IBM.2107-D02-04XXB/R1-P1-D3 0 800.0 array member S5 Normal
IBM.2107-D02-04XXB/R1-P1-D4 0 800.0 array member S2 Normal
IBM.2107-D02-04XXB/R1-P1-D5 0 800.0 spare required S3 Normal
```
命令`lsioport`，示例如下：
```
dscli> lsioport
ID WWPN State Type topo portgrp
==============================================================================
I0000 50050763080011XD Communication established Fibre Channel-SW SCSI-FCP 0
I0001 50050763080051XD Communication established Fibre Channel-SW SCSI-FCP 0
I0002 50050763080091XD Communication established Fibre Channel-SW SCSI-FCP 0
I0003 500507630800D1XD Communication established Fibre Channel-SW SCSI-FCP 0
```
命令`lshostconnect`，示例如下：
```
dscli> lshostconnect
Name ID WWPN HostType Profile portgrp volgrpID ESSIOport
================================================================================
E980PRD_IBASE_1A 0000 10000090FAF6D5A4 iSeries IBM iSeries - OS/400 0 V0 I0000
E980PRD_IBASE_1B 0001 10000090FAF5E642 iSeries IBM iSeries - OS/400 0 V0 I0330
...
E980PRD_IASP_1A 0004 100000109B0E0A40 iSeries IBM iSeries - OS/400 0 V2 I0001
E980PRD_IASP_1B 0005 10000090FAF5EC2A iSeries IBM iSeries - OS/400 0 V2 I0300
```
命令`lsvolgrp`，示例如下：
```
dscli> lsvolgrp
Name ID Type
=======================================
E980PRD_IBASE V0 OS400 Mask
E980PRD_IASP V2 OS400 Mask
E980PRD_IASP V3 OS400 Mask
...
All CKD V10 FICON/ESCON All
...
All Fixed Block-512 V20 SCSI All
All Fixed Block-520 V30 OS400 All
```
命令`showvolgrp`可以查看volgrp里面的卷ID：
```
dscli> showvolgrp v3
```
#### showfbvol命令
`showfbvol`命令可以查看fbvol的详细信息，包括分布的ranks情况，命令输出示例：
```
Date/Time: June 27, 2024 9:24:44 PM CST IBM DSCLI Version: 7.8.51.148 DS: IBM.2107-75HFA66
Name                       E870TEST_IASP
ID                         12A5
accstate                   Online
datastate                  Normal
configstate                Normal
deviceMTM                  2107-A04
datatype                   FB 520P
addrgrp                    1
extpool                    P0
exts                       4206
cap (MiB)                  67296
captype                    iSeries
cap (2^30B)                65.7
cap (10^9B)                70.6
cap (blocks)               137822208
volgrp                     V15
ranks                      21
dbexts                     -
sam                        Standard
repcapalloc                -
eam                        managed
reqcap (blocks)            137822208
realextents                4206
virtualextents             8
realcap (MiB)              67296
migrating                  0
migratingcap (MiB)         0
perfgrp                    PG0
migratingfrom              -
resgrp                     RG0
tierassignstatus           -
tierassignerror            -
tierassignorder            -
tierassigntarget           -
%tierassigned              0
etmonpauseremain           -
etmonitorreset             unknown
GUID                       6005076308FFD1F300000000000013A6
safeguardedcap (2^30B)     -
safeguardedloc             -
usedsafeguardedcap (2^30B) -
safeguarded                no
SGC Recovered              no
```
&#8195;&#8195;如果需要统计几百个vol的ranks情况，可以将`showfbvol 12A5`这种命令写在文本里面，然后在GUI里面嵌入的DSCLI里导入脚本执行，执行完成后将结果粘贴到文本，然后在Linux下运行下面脚本显示结果，示例：
```sh
sed -n '/^ID/p; /ranks/p' showfbvol.txt |awk '{print $2}' |sed '{N;s/\n/,/}'
```
命令官方参考链接：[DS8870 showfbvol](https://www.ibm.com/docs/en/ds8870/7.5.0?topic=commands-showfbvol)
#### showsi命令
查看存储Signature等信息使用`showsi`命令，输出示例：
```
Name             sherwood
Desc             -
ID               IBM.2107-75NA901
Storage Unit     IBM.2107-75NA900
Model            996
WWNN             5005076408FEC795
Signature        9719-5d08-1865-1091
State            Online
ESSNet           Enabled
Volume Group     V0
os400Serial      795
NVS Memory       127.5 GB
Cache Memory     1867.3 GB
Processor Memory 1912.9 GB
MTS              IBM.5631-75NA910
numegsupported   16
ETAutoMode       all
ETMonitor        all
IOPMmode         Disabled
ETCCMode         -
ETHMTMode        Enabled
ETSRMode         Enabled
ETTierOrder      High utilization
ETAutoModeAccel  Enabled
```
### 修改类命令

### 用户管理
#### 密码相关命令
`showpass`命令可以现实当前系统用户的密码策略，示例：
```
dscli> showpass
Date/Time: June 26, 2024 7:21:54 PM CST IBM DSCLI Version: 7.9.33.112 DS: -
Password Expiration   90 days
Failed Logins Allowed 5
Password Age          1 days
Minimum Length        8
Password History      8
```
`chpass`命令更改密码策略中过期策略、寿命以及history示例：
```
dscli> chpass -expire 0 -age 0 -history 2
Date/Time: June 26, 2024 7:25:22 PM CST IBM DSCLI Version: 7.9.33.112 DS: -
CMUC00195I chpass: Security properties successfully set.
dscli> showpass
Date/Time: June 26, 2024 7:25:28 PM CST IBM DSCLI Version: 7.9.33.112 DS: -
Password Expiration   0 days
Failed Logins Allowed 5
Password Age          0 days
Minimum Length        8
Password History      2
```
#### 更改用户密码
使用`chuser`命令修改qlpar用户密码示例：
```
chuser -pw passw1rd qlpar
```
解锁用户密码示例：
```
chuser -unlock qlpar
```
官方命令参考文档：[DS8900 chuser](https://www.ibm.com/docs/en/ds8900/9.3.1?topic=commands-chuser)
### Copy Services命令
#### FlashCopy命令
FlashCopy命令：

命令|说明
:---|:---
[commitflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clicommitflash_1vd000.html)|用于灾难恢复，完成部分形成的全局镜像一致性组
[resyncflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_cliincflash_1vd004.html)|使用`-record`和`-persist`参数建立的现有FlashCopy对的时间点副本
[lsflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clilsflash_1kz82u.html)|显示FlashCopy关系列表以及列表中每个FlashCopy关系的状态信息
[mkflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_climkflash_1kz8nz.html)|启动从源卷到目标卷的时间点复制
[reverseflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clireverseflash_1kz8m0.html)|反转FlashCopy关系
[revertflash](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clirevertflash_1kz8m1.html)|用户灾难恢复，从当前正在形成的全局镜像一致性组中恢复以前的全局镜像一致性组
[rmflash ](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clirmflash_1kz7xe.html)|删除FlashCopy关系
[unfreezeflash ](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clirmflashconsistency_1vd00b.html)|重置一个 FlashCopy 一致性组，该组以前在发出`mkflash`或`resyncflash`命令时使用`-freeze`参数建立
[setflashrevertible](https://www.ibm.com/docs/en/ST5GLJ_8.5.4/com.ibm.storage.ssic.help.doc/f2c_clisetflashrevertible_1kz8m2.html)|作为FlashCopy关系一部分的FlashCopy卷对修改为`revertible`