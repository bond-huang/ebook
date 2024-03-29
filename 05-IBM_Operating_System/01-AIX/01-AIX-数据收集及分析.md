# AIX-数据收集及查看
## 硬件类
&#8195;&#8195;gawk自定义变量可以是任意数目的字母、数字和下划线，但是不能以数字开头，并且区分大小写。
AIX系统安装在Power系列小型机上，硬件故障大多数也会记录在操作系统日志里面，一般情况下运行`snap -gc `收集可以了。如果没有HMC管理，AIX系统日志可以进行硬件故障判断，有些情况下需要二者结合。

硬件类型日志收集具体可以参考：[Power-小型机数据收集](https://bond-huang.github.io/huang/01-IBM_Power_System/02-Power_System/01-Power-%E5%B0%8F%E5%9E%8B%E6%9C%BA%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

## 系统日志
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

### 系统日志查看
使用`NormalizeSnap.sh`可以分析snap日志，暂时没有使用过，脚本代码地址：[NormalizeSnap.sh](https://adamssystems.nl/listings/NormalizeSnap.sh.html)。

## 多路径软件数据收集
连接外接存储并且使用多路径，当出现路径问题时候，需要收集多路径软件相关信息。
收集命令输出即可，常用AIX多路径软件有SDDPCM、SDD、MPIO和PowerPath。
### SDD收集命令
```shell
datapath query device
datapath query adapter
lsvpcfg
```
### MPIO收集命令
```shell
lspath
mpio_get_config -Av
```
### SDDPCM收集命令
```shell
pcmpath query device
pcmpath query adapter
lspcmcfg
```
### PowerPath（EMC存储）收集命令
```shell
powermt version
powermt display dev=all
```
## 日志查看
系统错误日志查看命令：
```
errpt
errpt -d H
errpt -d S
errpt -aj <event id>
```
系统错误日志目录：`/var/adm/ras/errlog`
系统启动日志查看目录：
```
/var/adm/ras/bootlog
/var/adm/ras/bosinstlog
/var/adm/ras/conslog
```
### alog命令
alog用于创建并维护创建字标准输入的固定大小的日志文件。常用参数介绍：
- `-C`:更改指定LogType的属性
- `-f <LogFile>`:指定日志文件的名称
- `-o`:列出日志文件的内容
- `-t <LogType>`:指定LogType
- `-H`:显示alog命令的用法
- `-s`:指定日志文件的大小限制（字节）

常用用法示例：

命令|功能
:---|:---
alog -t boot -o |查看系统启动日志
alog -t lvmcfg -o |查看LVM相关日志
alog -t boot -o &#124;grep data|查看系统最后重启时间
alog -f &#60;LogFile&#62; -o|列出日志文件的内容
date &#124; alog -f &#60;LogFile&#62;|记录日志文件中的当前日期和时间
alog -C -t boot -s 8192|修改boot日志大小为8192字节

更多用法介绍参考官方文档：[alog Command](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/a_commands/alog.html)

## 升级相关日志
install_all_updates: Log file is /var/adm/ras/install_all_updates.log

## 日志分析
### core dump分析
使用命令`snap -ac`收集dump，解压文件：
```
# ls
snap.pax.Z
# uncompress snap.pax.Z
# ls
snap.pax
# pax -r -f snap.pax
pax: Ready for volume 2
pax: Type "go" when ready to proceed (or "quit" to abort): go
pax: [offset 345m+441k+321]: Continuing
...
pax: Type "go" when ready to proceed (or "quit" to abort): go
pax: [offset 2g+24m+599k+902]: Continuing
pax: snap.pax : 0511-626 An invalid file header has been read.
pax: snap.pax :          Skipping to the next file...ls
```
解压报错了，并且没一会虚拟AIX把我电脑搞蓝屏，以后再示例。
## 数据收集脚本
### 常用命令输出收集
系统数量比较多，需要用到一个对所有系统免密的跳板机，检查是否免密的脚本：
```sh
#!/bin/bash
for ip in `cat jy_aix_all.list`
do
	ssh -o ConnectTimeout=3 -o BatchMode=yes $ip 'exit' > /dev/null 2>&1
	if [ $? -eq 0 ]
	then
		echo "$ip" >> aix.list
	else
		echo "$ip" >> aix_nopwd.list
	fi
done
```
收集数据脚本，例如名为aixck.sh：
```sh
#!/bin/ksh
echo "###### hostname ######"
hostname
echo "###### uname -Mu ######"
uname -Mu
echo "###### oslevel -s ######"
oslevel -s
echo "###### uptime ######"
uptime
echo "###### ntpq -p ######"
ntpq -p
echo "###### ifconfig -a ######"
ifconfig -a
echo "###### netstat -rn ######"
netstat -rn
echo "###### lspv ######"
lspv
echo "###### lsps -a ######"
lsps -a
echo "###### lspath ######"
lspath
echo "###### df -g ######"
df -g
echo "###### crontab -l ######"
crontab -l |sed -n '/^#/!p'
echo "###### cat /etc/hosts ######"
cat /etc/hosts |sed -n '/^#/!p'
echo "###### lsvg -p rootvg ######"
lsvg -p rootvg
echo "###### lsvg -l rootvg ######"
lsvg -l rootvg
echo "###### lsvg rootvg ######"
lsvg rootvg
for vg in `lsvg -o |grep -v rootvg`
do 
	echo "###### lsvg -p $vg ######"
	lsvg -p $vg
	echo "###### lsvg -l $vg ######"
	lsvg -l $vg
	echo "###### lsvg $vg ######"
	lsvg $vg
done
for fscsi in `lsdev |grep fscsi|awk '{print $1}'`
do 
	echo "###### lsattr -El $fscsi ######"
	lsattr -El $fscsi
done
for hdisk in `lspv |awk '{print $1}'`
do 
	echo "###### lsattr -El $hdisk ######"
	lsattr -El $hdisk
done
for dev in `lsdev -Cc adapter |grep -E "Virtual SCSI|Virtual Fibre|Virtual I/O"|awk '{print $1}'`
do 
	echo "###### lsattr -El $dev ######"
	lsattr -El $dev
done
echo "###### iostat ######"
iostat
echo "###### iostat -d 1 3 ######"
iostat -d 1 3 
echo "###### iostat -t 1 5 ######"
iostat -t 1 5
echo "###### iostat -atd 1 5 ######"
iostat -atd 1 5
echo "###### vmstat ######"
vmstat
echo "###### vmstat 1 5 ######"
vmstat 1 5 
echo "###### svmon -G ######"
svmon -G
echo "###### prtconf ######"
prtconf
echo "###### lscfg -vp ######"
lscfg -vp
echo "###### errpt ######"
errpt
```
使用跳板机将脚本传输到所有系统，脚本名为aixscp.sh:
```sh
#!/bin/bash
for ip in `cat aix.list`
do	
    scp aixck.sh $ip:/tmp/
done
```
数据收集脚本，脚本名为aixcheck.sh:
```sh
#!/bin/bash
for ip in `cat aix.list`
do	
	hostname=`ssh $ip "hostname"`
	ssh $ip "sh /tmp/aixck.sh " \
	> /root/ces/AIX/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
后台运行：
```sh
nohup sh -x aixcheck.sh &
```
传输和收集可以何在一起：
```sh
#!/bin/bash
for ip in `cat aix.list`
do	
    scp aixck.sh $ip:/tmp/
	hostname=`ssh $ip "hostname"`
	ssh $ip "sh /tmp/aixck.sh " \
	> /root/ces/AIX/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
### 批量数据分析
目前只做了抓取文件系统超过70%的脚本，脚本示例：
```sh
#!/bin/bash
for aixfile in `ls ./aixlist`
do
  echo "Check $aixfile filesystems"
  cat ./aixlist/$aixfile |sed -n '/###### df -g ######/,/###### crontab -l ######/{//!p}'|sed -n '/[0-9]%/p' > tempfile1
  while IFS= read -r line
  do
	pct=`echo $line |awk '{print $4}'|sed -n '/[0-9]%/p'|sed 's/[^0-9]//g'`
	if [[ $pct =~ ^[0-9]+$ ]]
	then 
		if [ $pct -ge 70 ]
		then 
			echo $line|awk '{print $7,$4}'
		fi
	fi
  done < tempfile1
done > aixresult
```
## 待补充
