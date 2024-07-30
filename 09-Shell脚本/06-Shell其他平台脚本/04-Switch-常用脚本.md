# Switch-常用脚本
交换机管理或配置梳理常用小脚本。
## SAN B-type
### 查询wwpn是否在此交换机上配置
&#8195;&#8195;有时候有一批需要查询一个主机端口不知道具体连的那些交换机，特别是NPIV的，通过switch不能直观看到，如果比较多也很难找。脚本具体说明：
- 交换机上直接操作不方便，把配置用`cfgshow`命令取下来，例如文件名叫cfgshow
- 把需要查询的WWN放在一个单列列表中，例如文件名叫wwnlist
- 在AIX或者Linux系统下运行即可
- 输出第一列是需要，第二列是zone名字，第三列是查询的WWN
- 可以将输出重定向到csv文件用excel打开编辑

脚本如下：
```sh
count=0
for wwn in `cat wwnlist`
do
    result=`cat cfgshow |grep -i $wwn|awk 'NR==1'`
    if [ -z "$result"]
    then
        count=$((count+1))
        echo "$count,$result,$wwn"
    fi
done
```
### 待补充
