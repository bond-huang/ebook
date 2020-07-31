# Shell笔记-基础脚本
shell脚本基础只是学习笔记
### 输入和显示
直接上脚本进行示例：
```sh
#!/bin/bash
#This script it a test script!
echo "who's logged into the system:"
who am i
echo -n "The date is:"
date
echo This is a test script!
```
脚本中知识点：
- 第一行指定使用的shell类型
- 第二行是注释，一般用"#"号(第一行例外)
- 直接写入命令就可以，执行脚本时候会输出命令输出结果
- 两个命令可用分号";"隔开写在同一行
- `echo`输出用户自定义内容，不需要引号，如果定义内容里有引号，需要加入引号
- 如果想让`echo`输出内容和后面命令输出内容显示在同一行，用`-n`参数

执行后输出结果如下：
```
[root@redhat8 basis]# ./basis_1.sh
who's logged into the system:
huang    pts/1        2014-02-21 07:55 (192.168.18.1)
The date is:Sat Feb 21 08:26:27 EDT 2014
This is a test script!
```
### 使用变量
脚本示例如下：
```sh
#!/bin/bash
#Environment variable usage
echo The user home directory is $HMOE
echo "The system hostname is $HOSTNAME"
echo "The cost of the phone is \$199"
time1=20
time2=$time1
name1="Bond"
echo "$name1 will get off work in $time2 minutes."
```
脚本中知识点：
- 调用系统环境变量可用直接调用，放在有双引号字符串中也可以
- 调用变量使用符合"$",当作为它本身时候需要加上转义字符反斜杠"\"
- 可以自定义变量，shell脚本自动决定变量类型，在脚本结束时候定义的变量被删除
- 可以将变量赋值给变量，一定要用"$"

执行后输出结果如下：
```
[root@redhat8 basis]# ./basis_2.sh
The user home directory is
The system hostname is redhat8
The cost of the phone is $199
Bond will get off work in 20 minutes.
```
### 命令替换
shell脚本可以从命令输出中提取信息，有两种方法：
- 反引号字符：`
- $()格式

示例如下：
```sh
#!/bin/bash
#Get the command output
syst1=`date`
syst2=$(date)
echo "The system date is："$syst1
echo The system date is：$syst2
today=$(date +%y%m%d)
cat /etc/filesystems > filesystems.bak.$today
ls -l
```
脚本中知识点：
- 示例了两种获取系统命令输出的方法，并且回顾了之前echo的用法
- 后面示例了实际中常用的调用方法，备份文件时候在输出文件名中加入自定义时间格式
- 命令替换会创建一个子shell运行命令，子shell是不会使用脚本中创建的命令

执行后输出结果如下：
```
[root@redhat8 basis]# sh basis_3.sh
The system date is：Sat Feb 21 08:26:27 EDT 2014
The system date is：Sat Feb 21 08:26:27 EDT 2014
total 8
-rw-r--r--. 1 root root 206 Feb 21 02:12 basis_3.sh
-rw-r--r--. 1 root root  66 Feb 21 02:12 filesystems.bak.140221
```
### 重定向输入和输出
在刚才的示例中已经用到了重定向输出的用法，继续详细学习。
##### 重定向输出
示例如下：
```sh
ls -l > test1
cat test1
date >> test1
cat test1
```
示例中知识点：
- 使用符合">"将输出重定向到指定文件中，系统没有此文件会自动创建一个
- 如果不想覆盖原文件内容，可以使用符号">>"来追加数据

执行后输出结果如下：
```
[root@redhat8 basis]# ls -l > test1
[root@redhat8 basis]# cat test1
total 8
-rw-r--r--. 1 root root   0 Feb 21 02:26 test1
[root@redhat8 basis]# date >> test1
[root@redhat8 basis]# cat test1
total 9
-rw-r--r--. 1 root root   0 Feb 21 02:26 test1
Sat Feb 21 08:26:27 EDT 2014
```
##### 重定向输入
输入重定向采用符号"<",示例如下：
```sh
wc < test1
```
执行后输出结果如下：
```
[root@redhat8 basis]# wc < test1
  7  53 305
```
wc命令会对数据中的文本进行计数，默认情况下输出3个值：文件的行数、词数和字节数。

还有一种输入重定向方法，叫内联输入重定向（inline input redirection，采用符号"<<"。
内联输入重定向必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可以，单在数据的开始和结尾的文本标记必须一致。内联输入重定向使用方法`command << marker`。示例如下：
```
[root@redhat8 basis]# wc << END
> This is a test string
> also is a test string
> END
 2 10 44
```
输入命令后显示次提示符"<"，以获取更火的输入数据，知道输入了作为标记的文本，然后`wc`命令会对输入的数据进行处理。

### 管道
经常需要将一个命令的输出作为另一个命令的输入，可以通过重定向来实现，但是相对比较麻烦。使用管道方法就很方便，日常中经常用到，直接示例：
```
[root@redhat8 basis]# rpm -qa | sort | grep python3 | more > python3.rpmlist
libpeas-loader-python3-1.22.0-6.el8.x86_64
python36-3.6.8-1.module+el8+2710+846623d6.x86_64
python3-asn1crypto-0.24.0-3.el8.noarch
```
输出命令`rpm -qa`结果，用`sort`排序，然后用`grep`查找`python3`，然后用文本分页命令`more`将输出按屏显示,最后将结果重定向到文件"python3.rpmlist"中。

### 数学运算
shell脚本中有两种途径来进行数学相关运算。
##### expr命令
关于expr命令操作符，后面章节采用专业页面收录，方便查阅。
直接在shell命令行中操作：
```
[root@redhat8 basis]# expr 10+10
10+10
[root@redhat8 basis]# expr 10 \* 10
100
```
注意，对于一些字符在shell有特殊含义的，需要使用转移字符"\"来让符号表示原有的含义。

脚本中示例如下:
```sh
#!/bin/bash
#use the expr conmmand
num1=100;num2=10
num3=$(expr $num1 / $num2)
echo The number is $num3
```
执行后输出结果如下：
```
[root@redhat8 basis]# sh basis_4.sh
The number is 10
```
##### 使用方括号
使用方括号比expr命令方便多了，命令行中示例如下：
```
[root@redhat8 basis]# nums=$[10+10];echo $nums
20
```
脚本中示例如下：
```sh
#!/bin/bash
num1=100;num2=10;num3=20
num4=$[$num2 * ($num1-$num3)]
echo The number is $num4
```
执行后输出结果如下：
```
[root@redhat8 basis]# sh basis_4.sh
The number is 800
```
注意事项：
- 在使用方括号的时候，不用担心乘号或其它符号代表非运算符的情况，所以不需要转义字符；
- 支支持整数运算，若要进行任意实际数字的计算，需要采用其它方法。

##### 浮点解决方案
有几种方法可以在bash中客服整数运算的限制，常见的方法就是内建的bash计算机，简称"bc"。bash计算机能够识别的内容：
- 数字（整数和浮点数）
- 变量（简单变量和数组）
- 表达式和函数
- 注释（以#或C语言中/* */开始的行）
- 编程语句（例如if-then）

直接运行命令`bc`就可以访问bash计算器，示例如下：
```
[root@redhat8 basis]# bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundatio
n, Inc.This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
scale=4
100*10+100/20-66.66/37
1003.1984
num1=10
num2=20
num3=num2 / num1
print num3
2.0000
```
示例中知识点：
- 使用scale指定输出的小数位数，scale变量默认值是0
- bash计算机还支持变量，引用就直接引用，用print去打印出来

脚本中使用示例如下：
```sh
#!/bin/bash
num1=10
num2=3.1415926
num3=$(echo "scale=3; 20 * 10 / 5 - $num2" | bc)
num4=$(echo "scale=2; $num3 * $num2" | bc)
echo The number is $num4
num5=$(bc << End
scale=4
num6=($num4 + $num3)
num7=($num1 * 10)
num6 + num7
End
)
echo The answer fot this mess is $num5
```
执行后输出结果如下：
```
[root@redhat8 basis]# sh basis_4.sh
The number is 115.7940999
The answer fot this mess is 252.6525073
```
脚本中的知识点：
- scale指定浮点运算小数位，应该是乘除不尽时候保留小数位
- 使用管道来调用bash计算器进行运算
- 可以使用内联输入重定向来可以针对较复杂的运算表达式，逐行显示，清晰明了

### 退出脚本
在shell中，每个命令都是以退出码状态（exit status）反馈给shell它已经运行完毕。退出状态码是一个0~255的整数，在命令结束时候由命令转给shell。有时候需要捕捉这个状态值在脚本中使用。

##### 查看状态退出码
在Linux中，可以使用变量"$?"来保存上个已执行命令的退出码状态，示例如下：
```
[root@redhat8 basis]# ls
basis_1.sh  basis_2.sh  basis_3.sh  basis_4.sh  filesystems.bak.140221  test1
[root@redhat8 basis]# echo $?
0
[root@redhat8 basis]# lsfeage
bash: lsfeage: command not found...
[root@redhat8 basis]# echo $?
127
```
Linux中退出状态码：

状态码|描述
:---|:---
0|命令成功结束
1|一般性未知错误
2|不适合的shell命令
126|命令不可执行
127|没找到命令
128|无效的退出参数
128+x|与Linux信号x相关的严重作为
130|通过Ctrl+C终止的命令
255|正常范围之外的退出状态码

##### exit命令
命令`exit`运行用户在脚本结束时指定一个自定义的退出码状态，同样最大值是255
例如在刚才basis_4.sh脚本中加入`exit 168`，示例：
```
[root@redhat8 basis]# sh basis_4.sh
The number is 115.7940999
The answer fot this mess is 252.6525073
[root@redhat8 basis]# echo $?
168
```
也可以调用变量的值进行返回，如果返回数值大于255，shell会通过模运算，最终结果是指定的值减去256得到的余数。
