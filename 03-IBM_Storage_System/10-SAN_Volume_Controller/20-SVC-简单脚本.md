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
## 待补充

