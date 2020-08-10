# Shell笔记-数据呈现
### 标准文件描述符
&#8195;&#8195;Linux系统将每个对象当作文件处理，这包括输入和输出进程。Linux用文件描述符（file descriptor）来标识每个文件对象，文件描述符是一个非负整数，每个进程一次最多有九个文件描述符，出于特殊目的，bash shell保留了前三个文件描述符，详细见下表：

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
&#8195;&#8195;在shell中，最对个可以有9个打开的文件描述符，其它3-8的文件描述符均可作为输入或输出重定向。可以将这些文件描述符中的人一个分配给文件，然后再脚本中使用。
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
可以用和重定向输出文件描述符同样的方法重定向输入文件描述符。示例如下：
```sh
#!/bin/bash
exec 6<&0
exec 0< testfile
count=1
while read line
do
    echo "Line #$count : $line"
    count=$[ $count +1 ]
done
exec 0<&6
read -p "Will you marry me?[Y/N] " answer
case $answer in 
Y | y)  echo
        echo "Yes,I do!" ;;
N | n)  echo
        echo "Sorry,you are a good man,goodbye!"
        exit ;;
esac
```
运行后示例如下：
```
[root@redhat8 output]# sh test6.sh
Line #1 : Yes,I do!
Line #2 : Let's paddle together!
Line #3 : 
Will you marry me?[Y/N] y

Yes,I do!
```
说明：
- 文件描述符6用来保存STDIN的位置，然后脚本将STDIN重定向到一个文件
- read命令的所有输入都来自重定向后的STDIN（也就是输入文件）
- 在读取所有行文件后，脚本会将STDIN重定向到文件描述符6，从而将STDIN的位置回复到源线的位置
- 改脚本用了另外一个read命令来测试STDIN是否恢复正常了，并且等待键盘的输入

### 创建读写文件描述符
使用需要注意，会覆盖原有的数据示例如下：
```sh
#!/bin/bash
exec 3<> testfile
read line <&3
echo "Read:$line"
echo "This is a test line " >&3
```
运行示例如下：
```
[root@redhat8 output]# cat testfile
Yes,I do!
Let's paddle together!
This is the third line!
[root@redhat8 output]# sh test7.sh
Read:Yes,I do!
[root@redhat8 output]# cat testfile
Yes,I do!
This is a test line 
!
This is the third line!
```
说明：
- 用exec将文件描述符3分配给文件testfile以进行文件读写，然后通过分配好的文件描述符，使用read命令读取文件中的第一行，然后将这一行显示在STDOUT上，最后用echo语句将一行数据写入由同一个文件描述符打开的文件中
- 脚本开始时候运行正常，输出内容表面脚本读取了testfile文件中的第一行，脚本运行完毕后，查看testfile文件内容的话，会发现写入文件中的数据覆盖了已有的数据
- 当脚本向文件中写入数据时，他会从文件指针所处的位置开始。read命令读取了第一行数据，所以它使得文件指针指向了第二行数据的第一个字符，在echo语句将数据输出到文件时，他会将数据放在文件指针的当前位置，覆盖了该位置已有的数据

### 关闭文件描述符
&#8195;&#8195;shell会在脚本退出时自动关闭创建的文件描述符，有些情况下，需要在脚本结束前手动关闭。关闭文件描述符，将它重定向到特殊符号`&-`,例如：`exec 3>&-`,脚本示例如下：
```sh
#!/bin/bash
exec 3> test11
echo "This is a test line of data" >&3
exec 3>&-
cat test11
exec 3> test11
echo "This will be bad" >&3
```
脚本运行后示例：
```
[root@redhat8 output]# sh test8.sh
This is a test line of data
[root@redhat8 output]# cat test11
This will be bad
```
说明：
- 一旦关闭了文件描述符，就能在脚本中向它写入任何数据，否则会报错
- 但是，随意在脚本中打开了同一个文件输出，shell会用一个新的文件来替换已有文件，也就是输出的新数据会覆盖已有文件

### 列出打开的文件描述符
&#8195;&#8195;lsof命令会列出整个Linux系统打开的所有文件描述符（普通用户运行可能要全路径：/user/sbin/lsof），输出会比较多，可以使用命令行选项和参数过滤输出，例如`-p`:指定进程ID（PID），`-d`：允许指定要显示的文件描述符编号。示例如下：
```
[root@redhat8 output]# lsof -a -p $$ -d 0,1,2
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    2546 root    0u   CHR  136,0      0t0    3 /dev/pts/0
bash    2546 root    1u   CHR  136,0      0t0    3 /dev/pts/0
bash    2546 root    2u   CHR  136,0      0t0    3 /dev/pts/0
```
说明：
- 想知道进程的当前PID，可以用特殊环境变量`$$`(shell会将他设为当前PID)
- 选项`-a`用来对其它两个选择的结果执行布尔and运算
- 上面示例显示了当前进程（bash shell）的默认文件描述符（0、1和2）

lsof的默认输出有7列信息，描述如下：

列|描述
:---|:---
COMMAND|正在运行的命令的前9个字符
PID|进程的PID
USER|进程属主的登录名
FD|文件描述符号以及访问类型（r代表读，w代表写，u代表读写）
TYPE|文件类型（CHR字符型，BLK块型，DIR目录，REG常规文件）
DEVICE|设备的设备号（主设备号和从设备号）
SIZE|如果有的话，表示文件的大小
NODE|本地文件的节点号
NAME|文件名称

&#8195;&#8195;与STDIN、STDOUT和STDERR关联的文件类型是字符型，三种文件描述符都指向终端，所有输出的文件名称就是终端的设备名，三种都支持读和写，脚本示例如下：
```sh
#!/bin/bash
exec 3> test12
exec 6> test13
exec 7< testfile
lsof -a -p $$ -d 0,1,2,3,6,7
```
脚本运行后示例：
```
[root@redhat8 output]# sh test9.sh
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
sh      2935 root    0u   CHR  136,0      0t0      3 /dev/pts/0
sh      2935 root    1u   CHR  136,0      0t0      3 /dev/pts/0
sh      2935 root    2u   CHR  136,0      0t0      3 /dev/pts/0
sh      2935 root    3w   REG  253,0        0 782380 /shell/output/test12
sh      2935 root    6w   REG  253,0        0 789185 /shell/output/test13
sh      2935 root    7r   REG  253,0       57 782397 /shell/output/testfile
```
### 阻止命令输出
&#8195;&#8195;有时候不想显示脚本的输出，如果允许在后台的脚本出现错误消息，可以发送邮件给属主，但是有时候会很麻烦，可以将STDERR重定向到一个叫做null文件的特殊文件，shell输出到null文件的任何数据都补i保持，全部都被丢掉了，在Linux中标准位置是/dev/null，重定向到此的任何数据都会被丢掉。示例如下：
```
[root@redhat8 output]# cat test8
This is a error message!
[root@redhat8 output]# cat /dev/null > test8
[root@redhat8 output]# cat test8
[root@redhat8 output]# date > /dev/null
[root@redhat8 output]# cat /dev/null
[root@redhat8 output]# ls -al badfile test9 2> /dev/null
-rw-r--r--. 1 root root 38 Aug  3 11:00 test9
```
说明：
- 可用于快速清除文件，例如示例中的test8，也是日常中清除日志文件的常用方法
- 这是避免出现错误消息，也无需保存他们的一个常用方法，示例中最后一个示例

### 创建临时文件
&#8195;&#8195;在Linux中，可以使用mktemp命令在/tmp目录中创建一个唯一的临时文件，但不会使用默认的umask值，它会将文件的读和写权限分配给文件的数字，并将你设置长文件的属主，一旦创建了，在脚本中就有完整的读写权限，其它非root用户无法访问。
##### 创建本地临时文件
&#8195;&#8195;默认情况下，mktemp会在本地目录中创建一个文件，用mktemp命令时候，指定一个文件名模板就行了，模板可以包含任意文本文件名，在文件名末尾加上6个X就可以，mktemp会用6个字符替代6个X，确保文件名的唯一性，示例如下：
```
[root@redhat8 output]# mktemp test.XXXXXX
test.O7Y4vI
```
在脚本中示例如下：
```sh
#!/bin/bash
tmpfile=$(mktemp test.XXXXXX)
exec 3>$tmpfile
echo "This script writes to temp file $tmpfile"
echo "This is the first line!" >&3
echo "This is the second line!" >&3
echo "This is the last line!" >&3
exec 3>&-
echo "Done creating temp file.The contents are: "
cat $tmpfile
rm -f $tmpfile 2> /dev/null
```
运行后示例如下：
```
[root@redhat8 output]# sh test10.sh
This script writes to temp file test.hBLWII
Done creating temp file.The contents are: 
This is the first line!
This is the second line!
This is the last line!
[root@redhat8 output]# ls -al test.hBLWII
ls: cannot access 'test.hBLWII': No such file or directory
```
说明：
- 示例脚本用mktemp命令来创建临时文件并将文件名赋给$tmpfile变量
- 接着将这个临时文件作为文件描述符3的输出重定向文件
- 在将临时文件名显示在STDOUT之后，向零食文件中写入了几行文本，然后关闭了该文件描述符
- 最后，显示出临时文件的内容，并用rm命令将其删除

##### 在/tmp目录篡改就临时文件
&#8195;&#8195;mktemp命令的`-t`选项会在系统的临时目录来创建该文件，在用这个特性时，会返回用来创建临时文件的全路径，而不是只有文件名，示例如下：
```
[root@redhat8 output]# mktemp -t test.XXXXXX
/tmp/test.pbRqLV
```
说明：由于返回的是全路径名，可以在系统上任何目录中银行用该临时文件，不管临时文件在哪里。

##### 创建临时目录
&#8195;&#8195;mktemp命令的`-d`选项可以创建一个临时目录而不是临时文件，这样就能够用该目录进行任何需要的操作，比如创建其它临时文件，示例如下：
```
[root@redhat8 output]# mktemp -d dir.XXXXXX
dir.EMEN9F
[root@redhat8 output]# ls -l
total 84
drwx------. 2 root root   6 Aug 10 11:55 dir.EMEN9F
```
### 记录消息
&#8195;&#8195;将输出同时发送到显示器和日志文件，不需要输出重定向两次，用tee命令就行。tee命令相当于一个管道的T型接头，它将STDIN过来的数据同时发往两处，一处是STDOUT,另一处是tee命行所指定的文件名：`tee filename`。示例如下：
```
[root@redhat8 output]# date | tee test14
Mon Aug 10 12:01:42 EDT 2010
[root@redhat8 output]# cat test14
Mon Aug 10 12:01:42 EDT 2010
huang    pts/0        2010-08-10 09:39 (192.168.18.1)
[root@redhat8 output]# cat test14
huang    pts/0        2010-08-10 09:39 (192.168.18.1)
[root@redhat8 output]# date | tee -a test14
Mon Aug 10 12:05:07 EDT 2010
[root@redhat8 output]# cat test14
huang    pts/0        2010-08-10 09:39 (192.168.18.1)
Mon Aug 10 12:05:07 EDT 2010
```
说明：
- 由于tes会重定向来自STDIN的数据，可以配合管道命令来重定向命令输出
- tee命令会在每次使用时覆盖文件内容
- 使用`-a`选项可以将数据追加到文件中

利用此方法，可以将数据保存在文件中，同时也能将数据显示在屏幕上，示例：
```sh
#!/bin/bash
tmpfile=test15
echo "This is the first line!" | tee $tmpfile
echo "This is the second line!" | tee -a $tmpfile
echo "This is the last line!" | tee -a $tmpfile
```
运行后示例如下：
```
[root@redhat8 output]# sh test11.sh
This is the first line!
This is the second line!
This is the last line!
[root@redhat8 output]# cat test15
This is the first line!
This is the second line!
This is the last line!
```
### 实例
实例收录在以下章节，方便查阅：[Shell-bash脚本实例](https://bond-huang.github.io/huang/09-Shell%E8%84%9A%E6%9C%AC/02-Shell%E8%84%9A%E6%9C%AC%E5%BF%AB%E9%80%9F%E6%8C%87%E5%8D%97/01-Shell-bash%E8%84%9A%E6%9C%AC%E5%AE%9E%E4%BE%8B.html)

