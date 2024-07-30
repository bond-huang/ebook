# Shell-PowerHA巡检脚本
## 批量收集数据分析处理
### 批量数据收集
说明：执行下面脚本都是在一个RHEL系统上，对清单里面的所有AIX系统都可以免密访问。

在一批AIX系统中筛选出是否是HA（可以排除TSAMP），脚本如下：
```sh
#!/bin/ksh
for ip in `cat aix.list`
do
    ha=`ssh $ip "/usr/sbin/lscluster -c |sed -n '/Cluster Name:/p'"`
	if [ -n "$ha" ]
	then
		echo $ip >> haip.list
	fi
done
```
然后进行批量数据收集，收集脚本hack.sh内容示例：
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
echo "###### df -g ######"
df -g
echo "###### clustername ######"
/usr/sbin/lscluster -c |sed -n '/Cluster Name:/p'| awk '{print $3}'
echo "###### cluster version ######"
lslpp -l |grep cluster.es.server.rte|uniq|awk '{print $2}'
echo "###### rsct install version ######"
lslpp -l |grep rsct.basic.rte|uniq|awk '{print $2}'
echo "###### rsct act version ######"
lsrpdomain |tail -n 1|awk '{print $3}'
echo "###### cluster config ######"
/usr/sbin/lscluster -c
echo "###### cluster node config ######"
/usr/sbin/lscluster -m
echo "###### cluster interface ######"
/usr/sbin/lscluster -i
echo "###### cluster storage ######"
/usr/sbin/lscluster -d
echo "###### cltopinfo ######"
/usr/es/sbin/cluster/utilities/cltopinfo
echo "###### IP Label ######"
/usr/es/sbin/cluster/utilities/cltopinfo -i
echo "###### cluster src ######"
lssrc -g cluster
echo "###### caa src ######"
lssrc -g caa
echo "###### rsct src ######"
lssrc -g rsct_rm
echo "###### rpnode ######"
lsrpnode
echo "###### rpdomain ######"
lsrpdomain
echo "###### Current state ######"
lssrc -ls clstrmgrES
echo "###### netmon ######"
cat /usr/es/sbin/cluster/netmon.cf
echo "###### rhosts ######"
cat /etc/cluster/rhosts
cat /usr/es/sbin/cluster/etc/rhosts
echo "###### hosts ######"
cat /etc/hosts |sed '/^#/d'
echo "###### caa_config ######"
caasta=`lssrc -g caa |grep clcomd|awk '{print $4}'`
if [ "$caasta" == active ]
then
	rpdomain=`lsrpdomain |tail -n 1|awk '{print $1}'`
	/usr/sbin/rsct/bin/caa_config -s  $rpdomain
fi
echo "###### Resource Group state ######"
/usr/es/sbin/cluster/utilities/clRGinfo -p
echo "###### end ######"
```
将批量数据收集脚本hack.sh传输到目标系统的tmp目录，脚本示例：
```sh
#!/bin/bash
for ip in `cat haip.list`
do
    scp hack.sh $ip:/tmp/
done
```
然后执行数据收集，收集后文件在result目录，脚本hacollect.sh示例：
```sh
#!/bin/bash
for ip in `cat haip.list`
do
        hostname=`ssh $ip "hostname"`
        ssh $ip "sh /tmp/hack.sh " \
        > /root/ces/powerha/result/"$hostname"_"$ip"_`date +%Y%m%d`
done
```
量大可能会比较久，后台执行示例：
```sh
nohup sh -x hacollect.sh &
```
### 批量分析
收集到的数据文件在result目录中，执行下面脚本即可进行分析：
```sh
#!/bin/bash
echo "cluster name,node count,hostname,node ip,HA version,rsct version,cluster server,caa server,node ha status,\
RG Primary State,RG Secondary State,priority 1 site,netmon,rhosts,node state,caavg_private disk,interfaces"
for hafile in `ls ./result`
do
	### clustername
	clustername=`cat ./result/$hafile |sed -n '/###### cluster config ######/,/###### cluster interface ######/{//!p}'\
	|grep "Cluster Name:"|awk '{print $3}'`
	### nodecount
	nodecount=`cat ./result/$hafile |sed -n '/###### cluster config ######/,/###### cluster interface ######/{//!p}'\
	|grep "Number of nodes in cluster"|awk '{print $7}'`
	### hostname
	hostname=`cat ./result/$hafile |sed -n '/###### hostname ######/,/###### uname -Mu ######/{//!p}'`
	### nodeip
	nodeip=`cat ./result/$hafile |sed -n '/###### cltopinfo ######/,/###### IP Label ######/{//!p}'\
	|sed -n '/'${hostname}'.[[:digit:]]/p'|awk '{print $2}'`
	### HA version
	haversion=`cat ./result/$hafile |sed -n '/###### cluster version ######/,/###### rsct install version ######/{//!p}'`
	### rsct version
	rsctversion=`cat ./result/$hafile |sed -n '/###### rsct act version ######/,/###### cluster config ######/{//!p}'`
	### clustersrc
	clustersrc=`cat ./result/$hafile |sed -n '/###### cluster src ######/,/###### caa src ######/{//!p}'\
	|grep "cluster"|awk '{print $4}'|uniq`
	if [ "$clustersrc"==active ]
	then
		clssrcst="All active"
	else
		clssrcst="Not all active"
	fi
	### caasrc
	caasrc=`cat ./result/$hafile |sed -n '/###### caa src ######/,/###### rsct src ######/{//!p}'\
	|grep "caa"|awk '{print $4}'|uniq`
	if [ "$caasrc"==active ]
	then
		caasrcst="All active"
	else
		caasrcst="Not all active"
	fi
	### node status
	nodehast=`cat ./result/$hafile |sed -n '/###### Current state ######/,/###### netmon ######/{//!p}'\
	|grep "Current state:"|awk '{print $3}'`
	### RG status
	rgprist=`cat ./result/$hafile |sed -n '/###### Resource Group state ######/,/###### end ######/{//!p}'\
	|grep "$hostname"|awk '{print $2}'`
	rgsecst=`cat ./result/$hafile |sed -n '/###### Resource Group state ######/,/###### end ######/{//!p}'\
	|grep "$hostname"|awk '{print $3 $4}'`
	### site_priority
	sitepri=`cat ./result/$hafile |sed -n '/###### caa_config ######/,/###### Resource Group state ######/{//!p}'\
	|grep -n "site_priority(1)"|awk '{print $3}'|sed -n 's/site_name\(.\+\)$/\1/p'`
	### netmon status
	netmon=`cat ./result/$hafile |sed -n '/###### netmon ######/,/###### rhosts ######/{//!p}'`
	if [ -n "$netmon" ]
	then
		netmonst="Not empty"
	else
		netmonst="Empty"
	fi
	### rhosts status
	rhosts=`cat ./result/$hafile |sed -n '/###### rhosts ######/,/###### hosts ######/{//!p}'`
	if [ -n "$rhosts" ]
	then
		rhostsst="Not empty"
	else
		rhostsst="Empty"
	fi
	### node state
	nodestate=`cat ./result/$hafile |sed -n '/###### cluster node config ######/,/###### cluster interface ######/{//!p}'\
	|grep "State of node:"|awk '{print $4}'|uniq`
	if [ "$nodestate"==UP ]
	then
		nodestate="All UP"
	else
		nodestate="Noet all UP"
	fi
	### caavg_private 
	diskstatus=`cat ./result/$hafile |sed -n '/###### cluster storage ######/,/###### cltopinfo ######/{//!p}'\
	|grep "State :"|awk '{print $3}'|uniq`
	if [ "$diskstatus"==UP ]
	then
		diskstatus="All UP"
	else
		diskstatus="Noet all UP"
	fi
	### interface state
	interfacest=`cat ./result/$hafile |sed -n '/###### cluster interface ######/,/###### cluster storage ######/{//!p}'\
	|grep "Interface state ="|awk '{print $4}'|uniq`
	if [ "$interfacest"==UP ]
	then
		interfacest="All UP"
	else
		interfacest="Noet all UP"
	fi
	echo $clustername,$nodecount,$hostname,$nodeip,$haversion,$rsctversion,$clssrcst,$caasrcst,$nodehast,\
	$rgprist,$rgsecst,$sitepri,$netmonst,$rhostsst,$nodestate,$diskstatus,$interfacest
done
```
执行将数据导入到csv文件，方便做成excel，例如：
```sh
sh hacheck.sh > hackresult.csv
```
## 待补充
