# Shell笔记-条件语句
主要内容是if条件判断语句和test命令的用法。
### if-then-else语句
格式如下：
```sh
if command
then
    command
else
    command
fi
```
当if语句中命令返回退出码状态为0时，then部分中的命令会被执行，当返回非零退出状态码时候，shell会执行else部分中的命令。else部分可以根据需求是否添加。脚本示例如下：
```sh
#!/bin/bash
testuser=NoneUser
if grep $testuser /etc/passwd
then
    echo "Tha bash files for user $testuser are:"
    ls -al /home/$testuser/.b*
else
    echo "The user $testuser does not exist on the system!"
fi
```
执行后输出结果如下：
```
[root@redhat8 if-for]# ./test1.sh
The user NoneUser does not exist on the system!
```
### 嵌套if
有时候需要多种判断条件，可以使用嵌套的if-then语句。同样检查用户是否存在脚本示例，在Home目录下建立一个用户文件夹，但是不创建用户：`mkdir /home/NoneUser`。脚本示例如下:
```sh
#!/bin/bash
testuser=NoneUser
if grep $testuser /etc/passwd
then
    echo "Tha user $testuser exsits on the system!"
else
    echo "The user $testuser does not exist on the system!"
    if ls -d /home/$testuser/
    then
        echo "The user $testuer has a directory!"
    fi
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# ./test2.sh
The user NoneUser does not exist on the system!
/home/NoneUser/
The user  has a directory!
```
如同条件更多呢，可以使用else部分另外一种性质elif，上面示例修改如下：
```sh
#!/bin/bash
testuser=NoneUser
if grep $testuser /etc/passwd
then
    echo "Tha user $testuser exsits on the system!"
elif ls -d /home/$testuser/
then
    echo "The user $testuser does not exist on the system!"
    echo "The user $testuer has a directory!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# ./test2.sh
/home/NoneUser/
The user NoneUser does not exist on the system!
The user  has a directory!
```
再更近一步，在检查一项，elif后面加上else，示例代码如下：
```sh
#!/bin/bash
testuser=noneuser
if grep $testuser /etc/passwd
then
    echo "Tha user $testuser exsits on the system!"
elif ls -d /home/$testuser/
then
    echo "The user $testuser does not exist on the system,but has a directory!"
else
    echo "The user $testuser does not exist on the system,and does not have a directory!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# ./test2.sh
ls: cannot access '/home/noneuser/': No such file or directory
The user noneuser does not exist on the system,and does not have w directory!
```
可以继续将多个elif语句串起来，格式如下：
```sh
if command1
then
    command set 1
elif command2
then
    command set2
elif command3
then 
    command set3
fi
```
### test命令
在if-then语句中，不能测试命令退出状态码之外的条件。命令`test`提供了在if-then语句中不同条件的途径，如果命令`test`中条件成立，命令`test`就会退出并返回退出状态码0，如果不成立，返回非0的退出状态码，if-then语句就不会继续执行了。

test命令格式比较简单：`test condition`，condition是test命令要测试的一系列参数和值,可以判断的条件有三种：数值比较、字符串比较和文件比较，格式如下：
```sh
if test condition
then
    commands
fi
```
也可以用方括号替代：
```sh
if [ condition ]
then
    commands
fi
```
### 数值比较
示例如下：
```sh
#!/bin/bash
num1=10;num2=99
if [ $num1 -gt 9 ]
then
    echo "The number $num1 is greater then 9!"
fi
if [ $num2 -eq $num1 ]
then
    echo "The numbers are equal!"
else
    echo "The numbers are different!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# ./test3.sh
The number 10 is greater then 9!
The numbers are different!
```
注意事项：
- 方括号前后都要有空格，在RHEL8中会报错，上面示例中不加空格会提示：[10: command not found；或者：./test3.sh: line 3: [: missing `]'
- 对于浮点值的比较，bash shell是不能处理的

命令`test`的数值比较功能对照表：

比较方式|描述
:---|:---
n1 -eq n2|检查n1是否与n2相等
n1 -ge n2|检查n1是否大于或等于n2
n1 -gt n2|检查n1是否大于n2
n1 -le n2|检查n1是否小于或等于n2
n1 -lt n2|检查n1是否小于n2
n1 -ne n2|检查n1是否不能于n2

### 字符串比较
示例如下：
```sh
#!/bin/bash
testuser=tmpusr;string=""
hero1=Thor;hero2=Hulk
if [ $USER != $testuser ]
then
    echo "This is not $testuser! "
else
    echo "Welcome $testuser! "
fi
#大小比较
if [ $hero1 \> $hero2 ]
then
    echo "$hero1 is stronger than $hero2!"
else
    echo "$hero2 is weaker than $hero2!"
fi
#字符串大小
if [ -n $string ]
then
    echo "The string '$string' is empty!"
else
    echo "The string '$string' is not empty!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# ./test4.sh
This is not tmpusr! 
Thor is stronger than Hulk!
The string '' is empty!
```
示例中的知识点：
- 比较字符串相等性时候，会将所有的大小写以及标点都考虑在内
- 大于号和小于号必须要用转义字符转义，不然会认为是重定向符号
- 大小写的顺序和sort命令锁采用的不同，比较测试中，大写字母是小于小写字母，采用的标准的ASCII顺序，而sort命令恰好相反

字符串比较测试表：

比较方式|描述
:---|:---
str1 = str2|检查str1是否和str2相同
str1 != str2|检查str1是否和str2不同
str1 < str2|检查str1是否比str2小
str1 > str2|检查str1是否比str2大
-n str1|检查str1长度是否非0
-z str1|检查str1长度是否为0

### 文件比较
文件比较在shell编程中使用的比较多，允许测试Linux文件系统上文件和目录的状态，先直接上表：

比较方式|描述
:---|:---
-d file|检查file是否存在并是一个目录
-e file|检查file是否存在
-f file|检查file是否存在并是一个文件
-r file|检查file是否存在并可读
-s file|检查file是否存在并非空
-w file|检查file是否存在并可写
-x file|检查file是否存在并可执行
-O file|检查file是否存在并属当前用户所有
-G file|检查file是否存在并且默认组于当前用户相同
file1 -nt file2|检查file1是否比file2新
file1 -ot file2|检查file1是否比file2旧

代码示例如下：
```sh
#!/bin/bash
homedir=$HOME;file1="testfile"
if [ -e $homedir ]
then
    echo "The $homedir directory exists and now checking the $file1!"
    if [ -f $homedir/$file1 ]
    then
        echo "The $file is a file and exist!"
        ls -l > $homedir/$file1
        if [ $homedir/$file1 -nt $homedir/test ]
        then
            echo "The $homedir/$file1 newer then $homedir/test"
        else
            echo "The $homedir/$file1 older then $homedir/test"
        fi
    else
        echo "The $file1 does not exist!"
    fi
else
    echo "The $homedir directory is not exist!"
fi
```
执行后结果如下：
```
[huang@redhat8 if-for]$ ./test5.sh
The /home/huang directory exists and now checking the testfile!
The  is a file and exist!
The /home/huang/testfile newer then /home/huang/test
```
一般检查思路就是：检查所在的目录是否存在，然后检查是否是一个文件，然后再检查次文件是否空文件或者是否可读或者可写可执行等。

### 复合条件测试
if-then语句允许用户使用布尔逻辑来组合测试：
- [ condition1 ] && [ condition2 ] 
- [ condition1 ] || [ condition2 ]

第一种是and布尔运算组合条件，要让then部分执行，两个条件必须同时满足，第二种满足其中一个，then部分就会执行。示例如下:
```sh
#!/bin/bash
if [ -d $HOME ] && [ -w $HOME/testfile ]
then
    echo "The file exists and you can write!"
else
    echo "You cannot write the file!"
fi
```
执行后结果如下：
```
[huang@redhat8 if-for]$ ./test6.sh
The file exists and you can write!
```
### if-then的高级特性
bash 提供了良心可在if-then语句中使用的高级特性：
- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

##### 使用双括号
双括号允许用户再比较过程中使用高级数学表达式，常用命令符号如下

符号|描述
:---|:---
val++|后增
val--|后减
++val|先增
--val|先减
!|逻辑求反
~|位求反
**|幂运算
<<|左位移
\>>|右位移
&|位布尔和
&#124;|位布尔或
&&|逻辑和
&#124;&#124;|逻辑或

示例如下：
```sh
#!/bin/bash
num1=10
if (( $num1 ** 3 > 90 ))
then
    (( num2 = $num1 ** 2 ))
    echo "The square of the $num1 is $num2!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# sh test7.sh
The square of the 10 is 100!
```
##### 使用双方括号
双方括号里面的expression使用了test命令中采用的标准字符串比较，也提供了一个test命令没有的特征：匹配模式（pattern maching）。示例如下：
```sh
#!/bin/bash
if [[ $HOSTNAME == r* ]]
then
    echo "The hostname is $HOSTNAME!"
else
    echo "Unkonw hostname!"
fi
```
执行后结果如下：
```
[root@redhat8 if-for]# sh test7.sh
The hostname is redhat8!
```
注意：不是所有的shell都指定双方括号，RedHat8是支持的。
##### case命令
当在一组值中寻找特定的值的时候，可能会写出很长的if-then-else语句，有了case命令就很简单了，语法如下：
```sh
case varibale in
pattern1 | prttern2) commands1;;
pattern3) commands2;;
*) default commands3;;
easc
```
示例如下：
```sh
#!/bin/bash
case $USER in
huang | root)
    echo "Welcome,$USER,please enjoy your visit!";;
testuser)
    echo "Special testuser account";;
tmpusr)
    echo "Do not forget to log off when you donot operating the system!";;
*)
    echo "Sorry,you are not allowed here";;
esac
```
执行后结果如下：
```
[root@redhat8 if-for]# sh test8.sh
Welcome,root,please enjoy your visit!
```
我是root用户，将root从条件中改成其它用户选项，输出结果如下：
```
[root@redhat8 if-for]# sh test8.sh
Sorry,you are not allowed here
```
其实就是依次判断条件是否满足，满足就执行对于的命令。
