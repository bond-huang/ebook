# PowerVM-数据收集

## 硬件故障
&#8195;&#8195;PowerVM是Power小型机平台上产品，VIOS也是一个系统，出现硬件故障就是Power小型机硬件故障，在非操作系统层面需要收集的日志可以参照：[Power-小型机数据收集](https://bond-huang.github.io/huang/01-IBM_Power_System/02-Power_System/01-Power-%E5%B0%8F%E5%9E%8B%E6%9C%BA%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

## PowerVM问题
如果虚拟I/O的问题，一般是PowerVM配置或者设备参数问题，需要结合VIOS和VIOC来判断问题。
### VIOS上收集
#### 收集snap
&#8195;&#8195;用padmin用户登录VIOS，输入命令`snap`收集snap，不需要加参数，在/home/padmin目录下生成的snap.pax.Z文件，拷贝出来即可。
#### 收集映射关系
收集vscsi、NPIV和网络映射关系，依次运行如下命令，通过终端日志功能记录下来：
```
$lsmap -all
$lsmap -all -npiv
$lsmap -all -net
```
将终端记录的日志拷贝下来。
### VIOC上收集
用root用户登录到VIOC，依次运行：
```
snap -r
snap -ac
```
在/tmp/ibmsupr目录下生成的snap.pax.Z文件，拷贝出来即可。
### HMC上收集
在HMC上同样也可以收集vscsi、NPIV和网络映射关系，用hscroot用户登录到HMC，依次运行如下命令，通过终端日志功能记录下来：
```
# lshwres -r virtualio -m <managed system> --rsubtype scsi --level lpar
# lshwres -r virtualio -m <managed system> --rsubtype fc --level lpar
# lshwres -r virtualio -m <managed system> --rsubtype eth --level lpar
```
## 数据收集脚本
### 常用命令输出收集
收集脚本，名字例如为viosck.sh：
```sh
#!/bin/ksh
ioscli="/usr/ios/cli/ioscli"
echo "###### hostname ######"
hostname
echo "###### uname -Mu ######"
uname -Mu
echo "###### ioslevel ######"
$ioscli ioslevel
echo "###### uptime ######"
uptime
echo "###### ifconfig -a ######"
ifconfig -a
echo "###### lspv ######"
lspv
echo "###### lsps -a ######"
lsps -a
echo "###### lspath ######"
lspath
echo "###### lsvg -p rootvg ######"
lsvg -p rootvg
echo "###### lsvg -l rootvg ######"
lsvg -l rootvg
echo "###### lsvg rootvg ######"
lsvg rootvg
echo "###### df -g ######"
df -g
echo "###### lsmap -all -net ######"
$ioscli lsmap -all -net;
echo "###### lsmap -all ######"
$ioscli lsmap -all; 
echo "###### lsmap -all -npiv ######"
$ioscli lsmap -all -npiv;
echo "###### lsmap -all -suspend ######"
$ioscli lsmap -all -suspend;
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
for sea in `$ioscli lsmap -all -net |grep SEA |grep ent |awk '{print $2}'`
do 
	echo "###### entstat -all $sea ######"
	$ioscli entstat -all $sea|grep Active
done
for dev in `lsdev -Cc adapter |grep -E "EtherChannel|Shared|Virtual I/O|FC Adapter"|awk '{print $1}'`
do 
	echo "###### lsattr -El $dev ######"
	lsattr -El $dev
done
for vdev in `lsdev -Cc adapter |grep -E "Virtual FC|Virtual SCSI"|awk '{print $1}'`
do 
	echo "###### lsdev -dev $vdev -attr ######"
	$ioscli lsdev -dev $vdev -attr
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
有多个vios系统，使用对所有系统免密的跳板机传输脚本，传输脚本名为viosscp.sh：
```sh
#!/bin/bash
for ip in `cat vioslist240118`
do	
    scp viosck.sh $ip:/tmp/
done
```
批量检查脚本并生成日志文件，脚本名为vioscheck.sh：
```sh
#!/bin/bash
for ip in `cat vioslist240118`
do	
	hostname=`ssh $ip "hostname"`
	ssh $ip "sh /tmp/viosck.sh " \
	> /root/ces/vios/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
两个传输和收集可以合在一起：
```sh
#!/bin/bash
for ip in `cat vioslist240118`
do	
    scp viosck.sh $ip:/tmp/
	hostname=`ssh $ip "hostname"`
	ssh $ip "sh /tmp/viosck.sh " \
	> /root/ces/vios/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
### 批量数据分析
## 待补充
