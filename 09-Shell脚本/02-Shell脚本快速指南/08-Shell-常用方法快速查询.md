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

### 其他行数据操作
#### 删除恐慌
删除空行
```sh
sed '/^$/d'
```
#### 数据合并成一行
数据合并成一行，方法如下：
```
sed ':a;N;$!ba;s/\n/ /g'
```
示例：
```
[root@centos82 testnew]# cat line
/dev/hd10opt 5.91 0.92 85% 298539 59% /opt
/dev/fslv00 10.00 3.02 70% 14444 2% /opt/IBM/ITM
10.24.128.155:/ptfs 580.00 89.51 85% 127168 1% /mnt
[root@centos82 testnew]# cat line |sed ':a;N;$!ba;s/\n/;/g'
/dev/hd10opt 5.91 0.92 85% 298539 59% /opt;/dev/fslv00 10.00 3.02 70% 14444 2% /opt/IBM/ITM;10.24.128.155:/ptfs 580.00 89.51 85% 127168 1% /mnt
```
#### 显示匹配到行的下一行
显示匹配到行的下一行方法：
```sh
sed -n '/pattern/ {n; p}' filename
```
## 字符串处理方法
### 只显示匹配到的字符
使用grep方法，示例：
```
[root@centos82 ~]# echo "I want a work!" |grep -o '\bwork\b'
work
```
使用sed方式示例：
```
[root@centos82 ~]# echo "I want a work!" |sed -n 's/.*\b\(work\)\b.*/\1/p'
work
```
&#8195;&#8195;下面方法也行，就是复杂点（用上面方法加下面正则表达式去匹配IP不行），可以将匹配到的字符和之外的字符分开显示，例如：
```
[root@centos82 ces]# echo ddfepatterndfe |sed -n '/pattern/s//\n&\n/p'
ddfe
pattern
dfe
```
这样就进行了换行拆分，通过此方法，可以从字符串中截取IP地址，示例：
```
[root@centos82 ces]# echo TEST1_10.0.82.78_20240723 |sed -n '/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/s//\n&\n/p'
TEST1_
10.0.82.78
_20240723
[root@centos82 ces]# echo TEST1_10.0.82.78_20240723 |sed -n '/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/s//\n&\n/p' \
|sed -n '/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/p'
10.0.82.78
```
### 只取数字字符
只取数据中的数字示例：
```sh
sed 's/[^0-9]//g'
```
#### 条件语句
## 条件语句
### 条件判断方法
判断变量是否是数字：
```sh
if [[ $value =~ ^[0-9]+$ ]]
```
### 格式注意
&#8195;&#8195;在使用if条件语句进行判断时候，`=`和`==`效果一样，但是要注意格式，最近发现写错了不会报错，但是完全得不到想要的结果，以下是错误写法：
```sh
if [ "$test"==test ]
if [ "$test"=test ]
```
正确写法：
```sh
if [ "$test" == test ]
if [ "$test" = test ]
```
## 待补充