# Shell笔记-数据呈现
### 标准文件描述符
Linux系统将每个对象当作文件处理，这包括输入和输出进程。Linux用文件描述符（file descriptor）来标识每个文件对象，文件描述符是一个非负整数，每个进程一次最多有九个文件描述符，出于特殊目的，bash shell保留了前三个文件描述符，详细见下表：

文件描述符|缩写|描述
:---:|:---:|:---:
0|STDIN|标准输入
1|STDOUT|标准输出
2|STDERR|标准错误

##### STDIN
SIDIN文件描述符代表shell的标准输入。对终端界面来说，标准输入是键盘。cat命令处理SIDIN示例：
```
[root@redhat8 ~]# cat
I am Iron Man!
I am Iron Man!
[root@redhat8 input]# cat < test
Yes,I do!
Let's paddle together!
```
说明：由键盘输入，也可以用重定向符号强制cat命令接收来自非SIDIN文件的输入。

##### STDOUT
SIDIN文件描述符代表shell的标准输出。对终端界面来说，标准输入就是终端显示器。输出导向STDOUT文件描述符示例：
```
[root@redhat8 output]# ls -l > tes
[root@redhat8 output]# pwd >> test
[root@redhat8 output]# cat test
total 0
-rw-r--r--. 1 root root 0 Aug  3 09:16 test
-rw-r--r--. 1 root root 0 Aug  3 09:16 test1.sh
/shell/output
```
##### STDERR
Shell通过特殊的STDERR文件描述符来处理错误消息。

只重定向错误消息（用`2>`，注意是紧贴着）示例如下：
```
[root@redhat8 output]# ls -l test3 > test4
ls: cannot access 'test3': No such file or directory
[root@redhat8 output]# ls -l test3 2> test4
[root@redhat8 output]# cat test4
ls: cannot access 'test3': No such file or directory
```
重定向错误和数据示例如下：
```
[root@redhat8 output]# ls -l test2 test3 2> test5 1> test6
[root@redhat8 output]# cat test5
ls: cannot access 'test3': No such file or directory
[root@redhat8 output]# cat test6
-rw-r--r--. 1 root root 0 Aug  3 09:29 test2
[root@redhat8 output]# ls -l test2 test3 &>test7
[root@redhat8 output]# cat test7
ls: cannot access 'test3': No such file or directory
-rw-r--r--. 1 root root 0 Aug  3 09:29 test2
```
### 在脚本种重定向输出
##### 临时重定向
如果有意在脚本种生成错误消息，可以将单独得一行输出重定向到STDERR。示例如下：
```sh
#!/bin/bash
echo "This is a error message!" >&2
echo "This is a normal output!"
```
运行后示例如下：
```
[root@redhat8 output]# sh test1.sh
This is a error message!
This is a normal output!
[root@redhat8 output]# sh test1.sh 2> test8
This is a normal output!
[root@redhat8 output]# cat test8
This is a error message!
```
##### 永久重定向
如果脚本种有大量数据需要重定向，可以使用exec命令告诉shell在脚本执行期间重定向某个特定文件描述符。脚本示例：
```sh
#!/bin/bash
exec 2>testerror
echo "This is the start of hte script!"
echo "Redirecting all output to another location"
exec 1>testout
echo "This output should go to the testout file"
echo "But this should go to the testerror file" >&2
```
运行后示例如下：
```
[root@redhat8 output]# sh test2.sh
This is the start of hte script!
Redirecting all output to another location
[root@redhat8 output]# cat testerror
But this should go to the testerror file
[root@redhat8 output]# cat testout
This output should go to the testout file
```
说明如下：
- 脚本用exec命令来讲发给STDERR的输出重定向到testerror
- 然后用echo想STDOUT显示了几行文本
- 随后再次使用exec命令在讲STDOUT重定向到testout文件
- 虽然STDOUT被重定向了，仍然可以讲echo的语句输出发给STDERR

### 在脚本种重定向输入
exec命令允将STDIN重定向到Linux系统上的文件种：`exex 0< testfile`,示例如下：
```sh
#!/bin/bash
exec 0< testfile
count=1
whilie read line
do
    echo "Line $count : $line"
    count=$[ $count + 1 ]
done
```
运行后输出如下：
```
[root@redhat8 output]# sh test3.sh
Line 1 : Yes,I do!
Line 2 : Let's paddle together!
Line 3 : 
```
### 创建自定义的重定向
在shell中，最对个可以有9个打开的文件描述符，其它3-8的文件描述符均可作为输入或输出重定向。可以将这些文件描述符中的人一个分配给文件，然后再脚本中使用。
### 创建输出文件描述符
脚本示例如下：
```sh
#!/bin/bash
exec 3>test9
echo "This should display on the monitor!"
echo "And this should be stored in the file" >&3
echo "Then this should be back on the monitor!"
```
运行后示例如下：
```
[root@redhat8 output]# sh test4.sh
This should display on the monitor!
Then this should be back on the monitor!
[root@redhat8 output]# cat test9
And this should be stored in the file
```
说明：如果想输入到现有的文件中，可以使用：`exec 3>>test9`。

### 重定向文件描述符
可以将STDOUT的原来位置重定向到另一个文件描述符，然后再利用该文件描述符的重定向回STDOUT，示例如下：
```sh
#!/bin/bash
exec 3>&1
exec 1>test10
echo "This should store in the output file!"
echo "Along with this line!"
exec 1>&3
echo "Now things should be back to normal!"
```
运行后示例如下：
```
[root@redhat8 output]# sh test5.sh
Now things should be back to normal!
[root@redhat8 output]# cat test10
This should store in the output file!
Along with this line!
```
说明：
- 脚本将文件描述符3重定向刀文件描述符1的当前位置，也就是STDOUT
- 第二个exec命令将STDOUT重定向到文件，shell会将发送给STDOUT的输出直接重定向到输出文件中
- 但是文件描述符3仍然指向STDOUT原来的位置，也就是显示器，此时将输出数据发送给文件描述符3，它仍然回出现在显示器上，尽管STDOUT已经被重定向了
- 在向STDOUT（现在指向一个文件）发送一些输出后，脚本将STDOUT重定向到文件描述符3当前的位置（现在仍然事显示器），这意味着STDOUT又指向了它原来的位置：显示器

### 创建输入文件描述符
