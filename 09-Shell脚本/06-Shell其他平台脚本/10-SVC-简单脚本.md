# SVC-简单脚本
记录一些简单实用的脚本。
## host相关
### 找出node logged in count为单数的host
&#8195;&#8195;SVC中可能会有很多hosts，并且如果是PowerVC自动化部署的hosts，状态都会在SVC上显示降级，如果hosts非常多，就很难找出哪些hosts的node_logged_in_count为单数，通过图形界面需要一个个查看属性，命令行中7.8微码版本也没找到比较直观列出来的命令。

脚本说明：
- 测试SVC版本是V7.8版本，此版本中没有awk或gawk命令，主要通过sed实现
- 直接把脚本贴在命令行运行即可，输出的内容复制到一个csv文件，然后通过excel格式优化即可
- 脚本只用到了lshost命令，不需要superuser等高权限用户，命令参考：[V7000 7.8 lshost](https://www.ibm.com/docs/en/ST3FR7_7.8.1/com.ibm.storwize.v7000.781.doc/svc_lshost_21pdxx.html)
- 脚本中第一列是host name，第二列是WWPN，第三列是此WWPN的node_logged_in_count，后面是第二列第三列的循环，根据自己的一个host WWPN多少去自定义多少，或者手动在csv里面加入也行
- 脚本中过滤掉了node_logged_in_count为0的WWPN，只列出了单数的，例如1，3，只要有单数的，host就会被列出来，具体哪个WWPN是单数也很直观看出来

脚本内容：
```sh
echo 'Host Name,WWPN,node_logged_in_count,WWPN,node_logged_in_count,\
    WWPN,node_logged_in_count,WWPN,node_logged_in_count,'
for id in `lshost -nohdr -delim : -filtervalue status=degraded |\
    sed 's/:.*//g'`
do
  for count in `lshost $id |\
    sed -n 's/node_logged_in_count //p' |sed -n '/0/!p'|uniq`
  do 
    if [ $(($count%2)) -ne 0 ]
    then 
      lshost -delim , $id |\
      sed -n '/^name\|^WWPN\|^node_/p'|\
      sed '/^WWPN/{N;s/\n/,/}'|\
      sed -n '/node_logged_in_count,0/!p'|\
      sed 's/name,//;s/WWPN,//;s/node_logged_in_count,//'|\
      tr '\n' ','
      echo
    fi
  done
done
```
输出示例：
```
Host Name,WWPN,node_logged_in_count,WWPN,node_logged_in_count,WWPN,node_logged_in_count,WWPN,node_logged_in_count,
DB-c99d1-63771951,C050760872FB00CD,1,C050760871FC00AC,1,C050760872EC00DA,1,C050760872FA00A1,1,
DB-6350d-18064883,C0507608CDF4002A,1,C0507608CDE4001B,1,C0507608CDB1008A,1,C0507608CDD10076,1,
```
## 卷相关
### vdisk中镜像在同一个池的卷
&#8195;&#8195;如果vdisk的镜像分布在同一个池，那么意义就不大，一般建议在不同的池，池的外部存储器也不是同一个。

脚本说明：
- 测试SVC版本是V7.8版本，此版本中没有awk或gawk命令，主要通过sed实现
- 直接把脚本贴在命令行运行即可，输出的内容复制到一个csv文件，然后通过excel格式优化即可
- 脚本只用到了lsvdisk命令，不需要superuser等高权限用户

首选筛选了`copy_count`为2的卷ID:
```sh
lsvdisk -nohdr -delim : -filtervalue copy_count=2
lsvdisk -nohdr -delim : -filtervalue copy_count=2 |sed 's/:.*//g'
```
通过查找匹配，如果去重后删除第一行为空，说明镜像在同一个池：
```sh
lsvdisk -delim , 2 |sed -n '/^mdisk_grp_id/p'
lsvdisk -delim , 2 |sed -n '/^mdisk_grp_id/p'|sed '1d'
lsvdisk -delim , 2 |sed -n '/^mdisk_grp_id/p'|sed '1d'|uniq
lsvdisk -delim , 2 |sed -n '/^mdisk_grp_id/p'|sed '1d'|uniq|sed '1d'
```
判断卷name是否一致的脚本：
```sh
for id in `lsvdisk -nohdr -delim : -filtervalue copy_count=2 |\
sed 's/:.*//g'`
do
  result=`lsvdisk -delim , $id |sed -n '/^mdisk_grp_id/p'|\
  sed '1d'|uniq|sed '1d'`
  if  [ -z $result ]
  then
    lsvdisk -delim , $id|\
    sed -n '/^name\|^capacity\|^mdisk_grp_name/p'
    fi
done
```
或者判断`mdisk_grp_name,many`这项的值是不是many：
```sh
for id in `lsvdisk -nohdr -delim : -filtervalue copy_count=2 |\
sed 's/:.*//g'`
do
  result=`lsvdisk -delim , $id |\
  sed -n '/^mdisk_grp_name,many/p'`
  if  [ -z $result ]
  then
    lsvdisk -delim , $id|\
    sed -n '/^name\|^capacity\|^mdisk_grp_name/p'
    fi	
done
```
### 查找池中主镜像在此池的卷
例如池ID为8，需要找出此池中卷的主镜像在此池的卷，下面三个卷类型说明：
```sh
### 主镜像在其他池：
lsvdisk -delim : 1925
### 主镜像在其本池：
lsvdisk -delim : 154
### 没有镜像：
lsvdisk -delim : 297
```
下面命令显示primary:yes和parent_mdisk_grp_id:8同时匹配的数据，以空行为数据分隔：
```sh
lsvdisk -delim : 154 |sed -n '/^$/ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p;x;s/.*//}; /./{H}; $ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p}'
lsvdisk -delim : 1925 |sed -n '/^$/ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p;x;s/.*//}; /./{H}; $ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p}'
lsvdisk -delim : 154 |sed -n '/^$/ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p;x;s/.*//}; /./{H}; $ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p}'
```
脚本示例：
```sh
for vdiskid in `lsvdisk -nohdr -delim :|cut -d: -f1`
do
	result=`lsvdisk -delim : $vdiskid |\
	sed -n '/^$/ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;\
	s/^\n//p;x;s/.*//}; /./{H}; \
	$ {x;/primary:yes/!d;/parent_mdisk_grp_id:8/!d;s/^\n//p}'`
	if [ -n "$result" ]
	then
		lsvdisk -nohdr -filtervalue id=$vdiskid
	fi
done
```
运行后示例：
```
154 Vmware_mbank_mysqldatamigrate_Pool_2 1 io_grp1 online many many 2.03TB many     600507680C8181600000000000000C9A 0 2 not_empty 2 no 0 many many no no 154 Vmware_mbank_mysqldatamigrate_Pool_2  
```
### 查找未映射给主机的卷
首先列所有的卷ID:
```SH
lsvdisk -nohdr -delim :|sed 's/:.*//g'
```
判断条件看下面命令是否有输出：
```sh
result=`lsvdiskhostmap 2`
```
脚本如下：
```sh
for id in `lsvdisk -nohdr -delim :|sed 's/:.*//g'`
do
  result=`lsvdiskhostmap $id`
  if  [ -z "$result" ]
  then
    lsvdisk -nohdr -delim , -filtervalue volume_id=$id
  fi
done
```
## 待补充

