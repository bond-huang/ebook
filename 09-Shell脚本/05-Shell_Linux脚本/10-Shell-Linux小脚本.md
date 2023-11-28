# Shell-Linux小脚本
## 用户相关
### 从其他系统同步用户
&#8195;&#8195;通常可以通过`rsync`或`scp`命令对文件`/etc/passwd`、`/etc/shadow`和`/etc/group`进行同步或者拷贝以达到从其他系统同步用户的目的，有时候需求不需要全部的，只需要部分，直接覆盖不行，但是又比较多的情况下，用脚本实现比较方便。例如redhat8系统中有如下用户要在redhat9中创建：
```sh
christ:x:1003:1003::/home/christ:/bin/bash
christb:x:1004:1003::/home/christb:/bin/bash
harry:x:1005:1005::/home/harry:/bin/bash
sarah:x:1006:1006::/home/sarah:/sbin/nologin
testuser2:x:1007:1007::/home/testuser2:/sbin/nologin
testusr3:x:1014:1014::/home/testusr3:/bin/bash
```
&#8195;&#8195;将上面需要copy的用户的内容拷贝到redhat9分区的文件中passwd.txt，对应`/etc/shadow`中的内容拷贝到shadow.txt中，对应group信息拷贝在group.txt文件夹中，脚本示例如：
```sh
#!/bin/bash
for groupinfo in `cat group.txt`
do
	group=`echo $groupinfo|gawk 'BEGIN{FS=":"}{print $1}'`
	groupid=`echo $groupinfo|gawk 'BEGIN{FS=":"}{print $3}'`
	groupadd $group -g $groupid
done
for usrinfo in `cat passwd.txt`
do 
    usr=`echo $usrinfo|gawk 'BEGIN{FS=":"}{print $1}'`
    usrid=`echo $usrinfo|gawk 'BEGIN{FS=":"}{print $3}'`
    usrgid=`echo $usrinfo|gawk 'BEGIN{FS=":"}{print $4}'`
    usrdir=`echo $usrinfo|gawk 'BEGIN{FS=":"}{print $6}'`
    usrsh=`echo $usrinfo|gawk 'BEGIN{FS=":"}{print $7}'`
    usrpw=`cat shadow.txt |grep -w $usr |gawk 'BEGIN{FS=":"}{print $2}'`
    useradd $usr -u $usrid -g $usrgid -d $usrdir -s $usrsh -p $usrpw
done
```
脚本执行示例：
```
[root@redhat9 ~]# sh cpusr.sh
```
redhat9分区创建用户信息：
```sh
christ:x:1003:1003::/home/christ:/bin/bash
christb:x:1004:1003::/home/christb:/bin/bash
harry:x:1005:1005::/home/harry:/bin/bash
sarah:x:1006:1006::/home/sarah:/sbin/nologin
testuser2:x:1007:1007::/home/testuser2:/sbin/nologin
testusr3:x:1014:1014::/home/testusr3:/bin/bash
```
脚本说明及注意事项：
- 注意在脚本中，`grep`后面要加上`-w`进行完全匹配，否则christ和christb这种用户`grep`后结果是两个
- 脚本没有考虑用户目录的权限，创建后是默认权限，可能与原分区权限不一样，需要注意

### 用户访问问题
#### 检查是否免密
批量检查iplist1文件中IP是否免密登录：
```sh
#!/bin/bash
while IFS= read -r ip
do
	ssh -o ConnectTimeout=3 -o BatchMode=yes $ip 'exit' > /dev/null 2>&1
	if [ $? -eq 0 ]
	then
		echo "$ip" >> iplist3
	else
		echo "$ip" >> iplist4
	fi
done < iplist1
```
脚本说明：
- iplist1文件中每一行是一个主机系统的IP地址
- 免密的IP和不免密的分别存放在当前目录的不同文件中

## 网络相关
### 网络端口
#### 检查端口是否通
脚本示例如下：
```sh
#!/bin/bash
cat ~/.ssh/known_hosts |gawk '{print $1}'|uniq > hostlist
while IFS= read -r ip
do
	nc -z -w 1 "$ip" 22 &> /dev/null
	if [ $? -eq 0 ]
	then
		echo "$ip" >> iplist1
	else
		echo "$ip" >> iplist2
	fi
done < hostlist
```
脚本说明：
- 脚本检查是known_hosts文件中的所有IP是否22端口能通，改成一个ip list也行
- 通的IP和不通的分别存放在当前目录的不同文件中

## 待补充