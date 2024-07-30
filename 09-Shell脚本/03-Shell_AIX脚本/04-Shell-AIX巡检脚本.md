# Shell-AIX巡检脚本
## 批量收集数据分析处理
### 批量数据收集
说明：执行下面脚本都是在一个RHEL系统上，对清单里面的所有AIX系统都可以免密访问。可以先检查是否免密：
系统数量比较多，需要用到一个对所有系统免密的跳板机，检查是否免密的脚本：
```sh
#!/bin/bash
for ip in `cat aix_allip.list`
do
	ssh -o ConnectTimeout=3 -o BatchMode=yes $ip 'exit' > /dev/null 2>&1
	if [ $? -eq 0 ]
	then
		echo "$ip" >> aixip.list
	else
		echo "$ip" >> aix_nopwd.list
	fi
done
```
进行批量数据收集的脚本aixck.sh内容示例：
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
echo "###### end ######"
```
将批量数据收集脚本aixck.sh传输到目标系统的tmp目录，脚本示例：
```sh
#!/bin/bash
for ip in `cat aixip.list`
do
    scp aixck.sh $ip:/tmp/
done
```
然后执行数据收集，收集后文件在result目录，脚本aixcollect.sh示例：
```sh
#!/bin/bash
for ip in `cat aixip.list`
do
        hostname=`ssh $ip "hostname"`
        ssh $ip "sh /tmp/aixck.sh " \
        > /root/ces/aix/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
量大可能会比较久，后台执行示例：
```sh
nohup sh -x aixcollect.sh &
```
传输和收集可以何在一起：
```sh
#!/bin/bash
for ip in `cat aixip.list`
do	
    scp aixck.sh $ip:/tmp/
	hostname=`ssh $ip "hostname"`
	ssh $ip "sh /tmp/aixck.sh " \
	> /root/ces/aix/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
### 批量分析
收集到的数据文件在result目录中，执行下面脚本即可进行分析：
```sh
#!/bin/bash
echo "hostname,IP,Machine SN,oslevel,rootvg disk status,rootvg stale lv,datavg disk status,\
datavg stale lv,disk path,fscsi dyntrk,fscsi fc_err_recov,fs over 70%,pagespace %,cpu entc %"
for aixfile in `ls ./result`
do
	### hostname
	hostname=`cat ./result/$aixfile |sed -n '/###### hostname ######/,/###### uname -Mu ######/{//!p}'`
	### IP Address
	ipaddr=`echo $aixfile |sed -n '/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/s//\n&\n/p'\
	|sed -n '/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/p'`
	### Machine Serial Number
	macsn=`cat ./result/$aixfile |sed -n '/Machine Serial Number:/p'|awk '{print $4}'`
	### oslevel
	oslevel=`cat ./result/$aixfile |sed -n '/###### oslevel -s ######/,/###### uptime ######/{//!p}'`
	### rootvg disk status
	rtvgdiskst=`cat ./result/$aixfile |sed -n '/###### lsvg -p rootvg ######/,/###### lsvg -l rootvg ######/{//!p}'\
	|sed -n '/^hdisk/p'|awk '{print $2}'|uniq`
	if [ "$rtvgdiskst" = active ]
	then
		rtvgdiskst="All active"
	else
		rtvgdiskst="Not all active"
	fi
	### rootvg stale lv
	rtvgstale=`cat ./result/$aixfile |sed -n '/###### lsvg -l rootvg ######/,/###### lsvg rootvg ######/{//!p}'\
	|sed -n '/stale/p'`
	if [ -z "$rootvgstale" ]
	then
		rtvgstale="No"
	else
		rtvgstale="Yes"
	fi
	### datavg disk status
	vgdiskst=`cat ./result/$aixfile |sed -n '/###### lsvg -p datavg ######/,/###### lsvg -l datavg ######/{//!p}'\
	|sed -n '/^hdisk/p'|awk '{print $2}'|uniq`
	if [ "$vgdiskst" = active ]
	then
		vgdiskst="All active"
	else
		vgdiskst="Not all active"
	fi
	### datavg stale lv
	datavgstale=`cat ./result/$aixfile |sed -n '/###### lsvg -l datavg ######/,/###### lsvg datavg ######/{//!p}'\
	|sed -n '/stale/p'`
	if [ -z "$rootvgstale" ]
	then
		datavgstale="No"
	else
		datavgstale="Yes"
	fi
	### disk path 
	diskpath=`cat ./result/$aixfile |sed -n '/###### lspath ######/,/###### df -g ######/{//!p}'\
	|awk '{print $1}'|uniq`
	if [ "$diskpath" = Enabled ]
	then
		diskpath="All Enabled"
	else
		diskpath="Not all Enabled"
	fi
	### fscsi configure check
	fscsidyntrk=`cat ./result/$aixfile |sed -n '/Dynamic Tracking of FC Devices/p'|awk '{print $2}'|uniq`
	if [ "$fscsidyntrk" = yes ]
	then
		fscsidyntrk="All yes"
	else
		fscsidyntrk="Not all yes"
	fi
	### fscsi fc_err_recov check
	fcerrrecov=`cat ./result/$aixfile |sed -n '/FC Fabric Event Error RECOVERY Policy/p'|awk '{print $2}'|uniq`
	if [ "$fcerrrecov" = fast_fail ]
	then
		fcerrrecov="All fast_fail"
	else
		fcerrrecov="Not all fast_fail"
	fi
	### filesystems check
	cat ./result/$aixfile |sed -n '/###### df -g ######/,/###### crontab -l ######/{//!p}'|sed -n '/[0-9]%/p' > tempfile1
	cat /dev/null > tempfile2
	while IFS= read -r line
	do
		pct=`echo $line |awk '{print $4}'|sed -n '/[0-9]%/p'|sed 's/[^0-9]//g'`
		if [[ $pct =~ ^[0-9]+$ ]]
		then 
			if [ $pct -ge 70 ]
			then 
				echo $line|awk '{print $7,$4}' >> tempfile2
			fi
		fi
	done < tempfile1
	fsover=`cat tempfile2|sed ':a;N;$!ba;s/\n/;/g'`
	cat /dev/null > tempfile2
	### pagespace check
	pgck=`cat ./result/$aixfile|sed -n '/###### lsps -a ######/,/###### lspath ######/{//!p}'\
	|sed '/Page Space/d'|awk '{print $1,$5"%"}'|sed ':a;N;$!ba;s/\n/;/g'`
	### cpu ent check
	cpuentc=`cat ./result/$aixfile |sed -n '/###### iostat ######/,/###### iostat -d 1 3 ######/{//!p}'\
	|sed -n '/entc/{n;p}'|awk '{print $8"%"}'`
	echo $hostname,$ipaddr,$macsn,$oslevel,$rtvgdiskst,$rtvgstale,$vgdiskst,$datavgstale,$diskpath,\
	$fscsidyntrk,$fcerrrecov,$fsover,$pgck,$cpuentc
done
```
执行将数据导入到csv文件，方便做成excel，例如：
```sh
sh aixcheck.sh > aixckresult.csv
```
单独抓取文件系统超过70%的脚本，脚本示例：
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