# Shell笔记-处理用户输入
### 命令行参数
bash shell会将一些称为位置参数(positional parameter)的特殊变量分配给输入到命令中的所有参数。位置参数变量是标准的数字：$0是程序名，$1是第一个参数，$2是第二个参数，依次类推，直到第九个参数$9，如果有更多的，需要加上大括号，例如：${10}、${11}。
##### 读取参数
示例脚本如下：
```sh
#!/bin/bash
total=$[ $1 * $2 ]
echo $1 \* $2 = $total
echo The name of $3 is $4!
echo "I am ${10}!"
```
将参数写入命令行中，并执行脚本：
```
[root@redhat8 input]# ./test1.sh 3 6 "Captain America" Steve 1 3 4 5 6 "Iron Man"
3 * 6 = 18
The name of Captain America is Steve!
I am Iron Man!
```
知识点：
- shell会自动将命令行参数的值分配给变量，每个参数间必须用空格隔开
- 文本和字符串都可以作为命令行参数
- 当一个参数中有空格时候，必须用引号（单引号或双引号均可）
- 当参数超过九个的时候，需要加上花括号，例如：${10}
- 示例中用到了星号，原本只是想输出乘号，但是输出的不是，说明冲突了，需要加上转义字符

##### 读取脚本名
可以用$0参数获取shell在命令行启动的脚本名，脚本示例如下：
```sh
#!/bin/bash
echo "The running script is:$0"
```
三种方式运行示例：
```
[root@redhat8 input]# bash test2.sh
The running script is:test2.sh
[root@redhat8 input]# ./test2.sh
The running script is:./test2.sh
[root@redhat8 input]# bash /shell/input/test2.sh
The running script is:/shell/input/test2.sh
```
可以看到把路径也加进去了，要剥离掉路径，可以采用`basename`命令，示例如下：
```sh
#!/bin/bash
echo "The running script is:$(basename $0)"
```
完整路径执行后如下所示：
```
[root@redhat8 input]# bash /shell/input/test2.sh
The running script is:test2.sh
```
##### 测试参数
脚本中使用了命令行参数，而脚本不加参数运行，脚本运行就会报错。可以使用`-n`来检查命令行参数$1中是否有数据，示例如下：
```sh
#!/bin/bash
if [ -n "$1" ]
then
    echo "Hello $1,glad to meet you!"
else
    echo "Sorry,you did not indentify yourself!"
fi
```
注意变量要加上引号："$1"，运行示例如下：
```
[root@redhat8 input]# sh test3.sh
Sorry,you did not indentify yourself!
[root@redhat8 input]# sh test3.sh Bond
Hello Bond,glad to meet you!
```
### 特殊参数变量
在bash shell中有些特殊变量，它们会记录命令行参数。
##### 参数统计
特殊变量`$#`含有脚本运行时携带的命令行参数的个数，示例如下：
```sh
#!/bin/bash
if [ $# -lt 2 ]
then
    echo "Please usage this format:test4.sh a b"
else
    echo $1 \* $2 = $((  $1 * $2  ))
fi
params=$#
echo The numbers of parameters are $params!
echo The last parameter is ${!#}
```
加参数运行后结果：
```
[root@redhat8 input]# sh test4.sh 3 4 5 2 4
3 * 4 = 12
The numbers of parameters are 5!
The last parameter is
```
注意事项：
- 可以对参数数量进行测试，如果参数数量不对，会显示错误消息提醒格式
- 书中用`${!#}`可以获取到最后一个参数，但是我跑脚本不行，不知道是不是RHEL原因

##### 抓取所有数据
有时候需要抓取命令行上提供的所有参数。变量`$*`和变量`$@`可以用来访问所有参数：
- 变量`$*`会将命令行上提供的所有参数当作一个元素保存，并将这些参数视为一个整体
- 变量`$@`会将命令行上提供的所有参数当作同意字符串的多个独立的元素；可以通过for来遍历所有的参数值去得到每个参数

代码示例如下：
```sh
#!/bin/bash
count=1
for param in "$*"
do
    echo Parameter $count = $param
    count=$[ $count + 1 ]
done
#
count=1
for param in "$@"
do
    echo "  Parameter $count = $param"
    count=$[ $count + 1 ]
done
```
加参数执行后如下所示：
```
[root@redhat8 input]# sh tst5.sh Hulk Thor "Captain America"
Parameter 1 = Hulk Thor Captain America
   Parameter 1 = Hulk
   Parameter 2 = Thor
   Parameter 3 = Captain America
```
### 移动变量
bash shell中`shift`命令可以用来操作命令参数，根据它们的相对位置来移动命令行参数。在使用`shift`命令时候，默认会将每个参数移动到前一个位置，$3的值会移动到$2中，到最后$1的值会被删除，$0不会改变。所以可以用来遍历命令行参数，在不知道参数多少时候，可以操作$1，然后移动参数，继续操作$1。脚本示例：
```sh
#!/bin/bash
shift 2
echo "The new first parameter is $1"
count=1
while [ -n "$1" ]
do
    echo Parameter $count = $1
    count=$[ $count + 1 ]
    shift
done
```
加参数运行脚本后示例：
```
[root@redhat8 input]# sh test6.sh 3 2 Thor Hulk Batman
The new first parameter is Thor
Parameter 1 = Thor
Parameter 2 = Hulk
Parameter 3 = Batman
```
### 处理选项
选项是跟在单破折线后的单个字母，它能改变命令的行为。
##### 查找选项
脚本示例：
```sh
#!/bin/bash
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option";;
    -b) echo "Found the -b option";;
     *) echo "$1 is not an option";;
    esac
    shift
done
```
运行脚本示例如下：
```
[root@redhat8 input]# sh test7.sh -a -b -c -d
Found the -a option
Found the -b option
-c is not an option
-d is not an option
```
##### 分离参数和选项
在Linux中用双破折号`--`来将命令行上的选项和参数划分开来。脚本示例：
```sh
#!/bin/bash
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option" ;;
    -b) echo "Found the -b option" ;;
    --) shift
        break ;;
     *) echo "$1 is not an option";;
    esac
    shift
done
count=1
for param in "$@"
do
    echo Parameter $count = $param
    count=$[ $count + 1 ]
done
```
加入参数和选项运行脚本示例如下：
```
[root@redhat8 input]# sh test8.sh -a -b -c -- Thor Hulk
Found the -a option
Found the -b option
-c is not an option
Parameter 1 = Thor
Parameter 2 = Hulk
```
##### 处理带值得选项
有些选项会带一个额外得参数值，脚本示例如下：
```sh
#!/bin/bash
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param="$2"
        echo "Found the -b option with parameter value $param"
        shift ;;
    --) shift
        break ;;
     *) echo "$1 is not an option";;
    esac
    shift
done
count=1
for param in "$@"
do
    echo Parameter $count = $param
    count=$[ $count + 1 ]
done
```
加入参数和选项运行脚本示例如下：
```
[root@redhat8 input]# sh test9.sh -a -b Hulk -c
Found the -a option
Found the -b option with parameter value Hulk
-c is not an option
[root@redhat8 input]# sh test9.sh -c -b Hulk -a
-c is not an option
Found the -b option with parameter value Hulk
Found the -a option
```
##### 使用getopt命令（AIX中适用）
getopt命令是在处理命令行选项和参数时非常方便的工具,格式：`getopt optstring parameters`,举例如下：
```
[root@redhat8 input]# getopt ab:cd -a -b Thor -cd Hulk Batman
 -a -b Thor -c -d -- Hulk Batman
[root@redhat8 input]# getopt -q ab:cd -a -b Thor -cd Hulk Batman -e 
 -a -b 'Thor' -c -d -- 'Hulk' 'Batman'
```
解释如下：
- optstring定义了四个有效字母，a,b,c和d，在b后面又冒号，因为b选项需要一个参数
- 当getopt运行时候会检查提供的参数列表(-a -b Thor -cd Hulk Batman),基于提供的optstring进行解析
- 注意，会自动将`-cd`选项分成两个单独的选项，并插入`--`来分割行中的额外参数
- 如果有不在optstring中的选项就会报错，在命令后面加入`-q`选项即可（AIX中不会报错加`-q`也没用）

脚本示例：
```sh
#!/bin/bash
set -- $(getopt -q ab:cd "$@")
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param="$2"
        echo "Found the -b option with parameter value $param"
        shift ;;
    --) shift
        break ;;
     *) echo "$1 is not an option";;
    esac
    shift
done
count=1
for param in "$@"
do
    echo Parameter $count = $param
    count=$[ $count + 1 ]
done
```
加入参数和选项运行脚本示例如下：
```
[root@redhat8 input]# sh test10.sh -a -b Thor -cd Hulk Batman
Found the -a option
Found the -b option with parameter value 'Thor'
-c is not an option
-d is not an option
Parameter 1 = 'Hulk'
Parameter 2 = 'Batman'
```
说明：
- set命令的选项`--`会将命令行参数替换成set命令的命令行值
- 第一行代码set行方式是将原始脚本的命令行参数传递给getopt，只后再将getopt命令的输出传给set命令，用格式化后的命令行参数来替换环视的命令行参数
- getopt命令不能处理带空格和引号的参数值，它会将空格单子参数分隔符

##### 使用getopts命令（AIX中适用）
getopts命令与getopt不通，getopts调用的时候只处理命令行上检测到的一个参数，处理完所有参数偶，它会退出并返回一个大于0的退出状态码,格式：`getopts optstring variable`,举例如下：
```sh
#!/bin/bash
while getopts :ab:c opt
do
    case "$opt" in 
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option with parameter value $OPTARG" ;;
        c) echo "Found the -c option" ;;
        *) echo "Unknown option: $opt" ;;
    esac
done
```
加入参数和选项运行脚本示例如下：
```
[root@redhat8 input]# sh test11.sh -ab "Captain America" -c -e
Found the -a option
Found the -b option with parameter value Captain America
Found the -c option
Unknown option: ?
```
说明：
- while语句定义了getopts命令，指明需要查找的命令行选项，以及每次迭代中存储它们的变量名“opt"
- getopts解析命令选项是会移开`-`，所有在case定义在不需要`-`
- getopts可以在参数值中包含空格和引号
- getopts能将命令行上找到的所有未定义的选项统一输出成问号

能在所有shell脚本中使用的全功能命令行选项和参数的处理工具：
```sh
#!/bin/bash
while getopts :ab:cd opt
do
    case "$opt" in 
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option with parameter value $OPTARG" ;;
        c) echo "Found the -c option" ;;
        d) echo "Found the -c option" ;;
        *) echo "Unknown option: $opt" ;;
    esac
done
shift $[ $OPTIND - 1 ]
count=1
for param in "$@"
do
    echo Parameter $count = $param
    count=$[ $count + 1 ]
done
```
加入参数和选项运行脚本示例如下：
```
[root@redhat8 input]# sh test12.sh -ab "Iron Man" -c Thor Hukl
Found the -a option
Found the -b option with parameter value Iron Man
Found the -c option
Parameter 1 = Thor
Parameter 2 = Hukl
```
说明：
- 在getopts处理每个选项时，会将OPTIND环境变量值增一
- 在getopts处理完成时，可以使用shift命令和OPTIND值来移动参数

### 选项标准化
选项可以自定义，但是有些字母选项在Linux中已经拥有了某种程度的标准含义，推荐使用统一标准的含义，方便他人阅读。    
常用Linxu命令选项：  

选项|描述
:---:|:---
-a|显示所有对象
-c|生成一个计数
-d|指定一个目录
-e|扩展一个对象
-f|指定读入数据的文件
-h|显示命令的帮助信息
-i|忽略文本大小
-l|产生输出的长格式版本
-n|使用非交互模式（批处理）
-o|将所有输出重定向到指定的输出文件
-q|以安静模式运行
-r|递归地处理目录和玩家
-s|以安静模式运行
-v|生成详细输出
-x|排除某个对象
-y|对所有问题回答yes

### 获得用户输入
bash shell中提供了read命令，增强交互性。
##### 基本的读取
read命令从标准输入（键盘）或另一个文件描述符中接受输入。在接收到输入后，read命令会将数据放进一个变量。脚本示例：
```sh
#!/bin/bash
echo -n "Please enter the hero's name: "
read hero
echo "YES,$hero is a superhero!"
#
read -p "Please enter the hero's home and ideal: " home ideal
echo "The superhero come from $home,and will $ideal!"
```
运行后示例如下：
```
[root@redhat8 input]# sh test13.sh
Please enter the hero's name: Batman     
YES,Batman is a superhero!
Please enter the hero's home and ideal: earth save the world 
The superhero come from earth,and will save the world!
```
知识点：
- echo命令使用了`-n`选项，该选项不会再字符串末尾输出换行符，用户可以在后面输入数据，然后用read接受了输入的数据，并且赋值给指定变量
- 可以在`read`中定义多个变量去接受输入数据，输入中以空格作为分隔符，如果变量不够，剩下的数据就会全部分配给最后一个变量，试过用冒号不行
- 如果在`read`中不定义变量，`read`会将接收到的所有数据放进特殊环境变量`REPLY`中，用`$REPLY`调用即可
- 有时候不想在屏幕上显示输入信息，比如密码，可以在`read`命令中使用`-s`选项

##### 超时
脚本可能一直等待用户输入，可以设定一个计时器，用`-t`选项，该选项指定了read命令等待输入的秒数，当设定时间到了后，read命令会返回一个非零退出状态码。脚本示例如下：
```sh
#!/bin/bash
if read -t 5 -p "Please enter the hero's name: " name
then
    echo " The superhero name is $name!"
else
    echo
    echo "Sorry,your enter is too slow!"
fi
```
运行后一直不输入情况下示例结果：
```
[root@redhat8 input]# sh test14.sh
Please enter the hero's name: 
Sorry,your enter is too slow!
```
也可以不设置时间，当用户输入达到一定的字符后，就自动突出并将输入的数据赋值给变量，示例：
```sh
#!/bin/bash
read -n1 -p "Will you marry me?[Y/N] " answer
case $answer in
Y | y)  echo
        echo "Yes,I do!" ;;
N | n)  echo
        echo "Sorry,you are a good man,goodbye!"
        exit ;;
esac
echo "Let's paddle together!"
```
运行示例如下：
```
[root@redhat8 input]# sh test15.sh
Will you marry me?[Y/N] Y
Yes,I do!
Let's paddle together!
[root@redhat8 input]# sh test15.sh
Will you marry me?[Y/N] n
Sorry,you are a good man,goodbye!
```
##### 从文件中读取
read可以从文件中读取内容，每次调用read命令，它都会从文件中读取一行文本，文本中没有内容时候就返回非零退出状态码。代码示例如下：
```sh
#!/bin/bash
count=1
cat test | while read line
do
    echo "Line $count:$line"
    count=$[ $count + 1 ]
done
```
运行后示例：
```
[root@redhat8 input]# sh test16.sh
Line 1:Yes,I do!
Line 2:Let's paddle together!
```
