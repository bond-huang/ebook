# Shell笔记-条件语句
主要内容是if条件判断语句和test命令的用法。
### if-then-else语句
格式如下：
```shell
if command
then
    command
else
    command
fi
```
当if语句中命令返回退出码状态为0时，then部分中的命令会被执行，当返回非零退出状态码时候，shell会执行else部分中的命令。else部分可以根据需求是否添加。脚本示例如下：
```shell
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
```shell
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
```shell
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
```shell
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
```shell
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
```shell
if test condition
then
    commands
fi
```
也可以用方括号替代：
```shell
if [ condition ]
then
    commands
fi
```
### 数值比较
示例如下：
```shell
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
- 方括号前后都要有空格，在RHEL8中会报错，上面示例中不加空格会提示：[10: command not found
- 对于浮点值的比较，bash shell是不能处理的。

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
