# Shell-常用方法快速查询
一些简单，但是常用的方法。
## 数据行处理方法
获取行数据：
```shell
# 取第一行
sed '1q'
head -n 1
awk 'FNR=1 {print}'
# 取前十行
sed '10q'
head -n 10
awk 'FNR>=1&&FNR<=10 {print}'
```
删除行：
```shell
# 删除第一行
sed '1d'
# 删除第2至5行
sed '2,5d'
```
### 获取两行间的数据行
例如数据如下：
```
###hostname###
VIOS1
###ioslevel###
2.2.4.22
###uptime###
08:55PM   up 646 days,  16:36,  0 users,  load average: 0.54, 0.58, 0.70
###lspath###
Enabled hdisk0 sas0
Enabled hdisk1 sas0
```
包含匹配行示例：
```
[root@centos82 ces]# cat test |sed -n '/###ioslevel###/,/###uptime###/p'
###ioslevel###
2.2.4.22
###uptime###
```
不包含匹配行示例：
```
[root@centos82 ces]# cat test |sed -n '/###ioslevel###/,/###uptime###/{//!p}'
2.2.4.22
```
注意，如果匹配的行有重复的，不会出现预期结果。


只取数字：
```sh
sed 's/[^0-9]//g'
```
删除空行
```sh
sed '/^$/d'
```
## 条件语句
判断变量是否是数字：
```sh
if [[ $value =~ ^[0-9]+$ ]]
```
## 待补充