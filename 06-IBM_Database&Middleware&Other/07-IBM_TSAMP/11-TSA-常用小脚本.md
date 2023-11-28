# TSA-常用小脚本
## 巡检
### 日常检查
#### 简单数据收集脚本
脚本示例如下：
```sh
#!/bin/bash
for ip in `cat ip_tsa`
do	
		hostname=`ssh $ip "hostname"`
        ssh $ip "hostname;samversion;lssam; \
		lssamctrl;lssamctrl -V |grep -i master;\
		lsrpdomain; \
		lsrpnode; \
		lscomg; \
		lsrel -Ab; \
		lsrsrc -l IBM.Host; \
		lsrsrc -l IBM.PeerNode; \
		lsrsrc -l IBM.PeerDomain; \
		lsrsrc -l IBM.HostPublic; \
		lsrsrc -l IBM.ServiceIP; \
		lsrsrc -l IBM.NetworkInterface; \
		lsrsrc -l IBM.CommunicationGroup; \
		lsrsrc -l IBM.ResourceGroup; \
		lsrsrc -l IBM.Application" > "$hostname"_"$ip"_`date +%Y%m%d`
done
```
脚本说明：
- 脚本只是简单将运行命令输出到指定文件夹
- ip_tsa文件中都是运行TSA系统的IP，每个系统收集信息保存在以主机IP及时间命名的文件中

## 其他
### TSA探测
#### 检查是否运行TSA
脚本示例：
```sh
#!/bin/bash
for ip in `cat iplist3`
do
	echo '' > tmpout
	hostname=`ssh $ip "hostname"`
	ssh $ip "lssam" > tmpout
	tsastatus=`cat tmpout |grep IBM.ResourceGroup|gawk '{print $1}'|uniq|wc -l`
	if [ $tsastatus -eq 1 ]
	then
		echo "$hostname;$ip;Online" >> tsalist
	fi
done
```
脚本说明：
- 确保iplist3里面的IP地址都是免密可以访问的
- 只是通过判断`lssam`是否有输出，查找`ResourceGroup`有结果判断
- 脚本有问题，最终判断成功后，只能说明TSA有，不一定资源组是`Online`，但是默认输出定义为`Online`，后续进行优化

## 待补充