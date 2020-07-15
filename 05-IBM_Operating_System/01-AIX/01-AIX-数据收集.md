# AIX-数据收集
### 硬件类
AIX系统安装在Power系列小型机上，硬件故障大多数也会记录在操作系统日志里面，一般情况下运行`snap -gc `收集可以了。如果没有HMC管理，AIX系统日志可以进行硬件故障判断，有些情况下需要二者结合。

硬件类型日志收集具体可以参考：[Power-小型机数据收集](https://bond-huang.github.io/huang/01-IBM_Power_System/02-Power_System/01-Power-%E5%B0%8F%E5%9E%8B%E6%9C%BA%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

### 系统日志收集
命令`snap`是AIX系统中收集系统信息和配置最常用的命令。
最常用的就是`snap -gc`，其中参数`-c`是打包压缩成.Z文件，
收集需要条件：
- 需要root用户执行
- 需要ftp服务器传输出日志
- 确保文件系统/tmp有足够的空间（`snap -ac`可能会很大，dump经常几个G）

收集方法以`snap -gc`为例如下：
- root用户登录到系统
- 运行`snap -r`清除之前收集的日志
- 运行`snap -gc`进行收集
- 在/tmp/ibmsupt目录下将snap.pax.Z文件拷贝出来

常用其它收集参数如下：
- `-a`：收集全部信息并压缩打包（包括dump，但是不包括PowerHA信息）
- `-D`：收集DUMP信息
- `-f`：收集文件系统信息
- `-e`：收集PowerHA信息
- `-k`：收集内核信息
- `-n`：收集NFS信息
- `-L`：收集LVM信息
- `-t`：收集网络相关信息

命令`snap`更多用法可以参照官方介绍：[AIX snap命令介绍](https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_72/s_commands/snap.html)

### 多路径软件数据收集
连接外接存储并且使用多路径，当出现路径问题时候，需要收集多路径软件相关信息。
收集命令输出即可，常用AIX多路径软件有SDDPCM、SDD、MPIO和PowerPath。
##### SDD收集命令
```shell
datapath query device
datapath query adapter
lsvpcfg
```
##### MPIO收集命令
```shell
lspath
mpio_get_config -Av
```
##### SDDPCM收集命令
```shell
pcmpath query device
pcmpath query adapter
lspcmcfg
```
##### PowerPath（EMC存储）收集命令
```shell
powermt version
powermt display dev=all
```
