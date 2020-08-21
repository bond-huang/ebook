# Shell笔记-函数

### 格式
函数是一个脚本代码块，可以为其命名并在代码中任何位置重用。在bash shell中函数有两种格式，如下所示：
```sh
function name {
    commands
}
```
以及：
```sh
name (){
    commands
}
```
### 函数使用
函数示例如下：
```sh
#!/bin/bash
function heros {
    echo "Thor,Hulk,Wonder Woman,Batman,Captain America"
}
count=1
while [ $count -le 2 ]
do
    heros
    count=$[ $count +1 ]
done
echo "Super Hero:heros"
```
运行后示例如下：
```
[root@redhat8 function]# sh test1.sh
Thor,Hulk,Wonder Woman,Batman,Captain America
Thor,Hulk,Wonder Woman,Batman,Captain America
Thor,Hulk,Wonder Woman,Batman,Captain America
```
注意事项：
- 在函数定义之后才能使用该函数，定义之前调用会报错
- 函数名必须是唯一的，如果重复了，新定义函数会覆盖原来的函数

### 返回值
运行结束时会返回一个退出状态码，有三种不同方式来为函数生成退出状态码。
##### 默认退出状态码
&#8195;&#8195;默认情况下，函数的退出状态码是函数中最后一条命令返回的退出状态码，在函数执行结束后可以用标准变量`$?`来确定函数的退出状态码。使用默认退出状态码值是通过最后一条命令判断，但是无法知道函数中其它命令是否运行成功，示例如下：
```sh
#!/bin/bash
function1 () {
    ls -l badfile
    echo "Test bad command"
}
echo "Test the function:"
function1
echo "The exit status is $?"
```
运行后示例如下：
```
[root@redhat8 function]# sh test2.sh
Test the function:
ls: cannot access 'badfile': No such file or directory
Test bad command
The exit status is 0
```
##### 使用return命令
&#8195;&#8195;bash shell使用return命令来退出函数并返回特定的退出状态码。return命令允许指定一个数值来定义函数的退出状态码，脚本示例如下：
```sh
#!/bin/bash
function db1 {
    read -p "Enter a value: " value
    echo "Doubling the value"
    return $[ $value * 2 ]
}
db1
echo "The new value is $?"
```
运行后示例如下：
```
[root@redhat8 function]# sh test3.sh
Enter a value: 100
Doubling the value
The new value is 200
```
注意事项：
- 函数一结束就取返回值，如果中途运行了其它命令，函数的返回值就会丢失
- 退出状态码必须是0-255，任何大于256的值都会产生一个错误值

##### 使用函数输出
可以将函数的输出保存到变量中,例如将db1函数的输出赋值给result变量：result=&#96;db1&#96;,示例如下：
```sh
#!/bin/bash
db1 () {
    read -p "Enter a number: " value
    echo $[ $value * 2 ]
}
result=$(db1)
echo "The new number is $result!"
```
运行后示例如下：
```
[root@redhat8 function]# sh test4.sh
Enter a number: 1118
The new number is 2236!
```
说明:     
&#8195;&#8195;示例中使用了read取获取输入，read命令的输出bash shell并没将其作为STDOUT的一部分，如果使用echo语句生成这样的消息向用户查询，那么会与输出值一起被读进shell变量中。

### 在函数中使用变量

##### 向函数传递参数
&#8195;&#8195;函数可以使用标准的参数环境变量来表示命令行上传给函数的参数。例如，$0是函数名，函数命令行上的任何参数都会通过$1、$2等定义，特殊环境变量$#来判断传递给函数的参数数目。在脚本中指定函数时，必须将参数和函数放在同一行，示例：`function $value 1118`。示例如下：
```sh
#!/bin/bash
addnum () {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then 
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $[ $1 + $1 ]
    else
        echo $[ $1 + $2 ]
    fi
}
echo -n "Adding 18 and 19: "
value=$(addnum 18 19);echo $value
echo -n "Try adding just one number: "
value=$(addnum 18);echo $value
echo -n "Try adding no number: "
value=$(addnum);echo $value
echo -n "Try adding three numbers: "
value=$(addnum 18 19 20);echo $value
```
运行后示例如下：
```
[root@redhat8 function]# sh test5.sh
Adding 18 and 19: 37
Try adding just one number: 36
Try adding no number: -1
Try adding three numbers: -1
```
注意：    
&#8195;&#8195;由于函数使用的特殊参数环境变量作为字节的参数值，因此他无法直接获取脚本在命令行中的参数值，所以不能再脚本后面加参数去传入到函数中。如果需要用到脚本的参数，可以在调用函数的时候手动传入，例如：`value=$(addnum $1 $2)`。

##### 在函数中处理变量
函数使用的两种类型的变量：
- 全局变量：在shell脚本中任何地方都有效的变量；默认情况下脚本中定义的任何变量都是全局变量
- 局部变量：不需要在函数中使用全局变量时，函数内部的任何变量都可以被声明为局部变量，在变量声明前加上`local`关键字即可，也可以在变量赋值语句前面使用关键字：`local tmp=$[ $value + 1 ]`。

###### 全局变量
示例：
```sh
#!/bin/bash
function test1 {
    value=$[ $value * 3 ]
}
read -p "Please Enter a value: " value
test1
echo "The new value is: $value"
```
运行后示例如下：
```
[root@redhat8 function]# sh test6.sh
Please Enter a value: 168
The new value is: 504
```
说明：
- 变量$value在函数外定义并被赋值，当test1被调用时，该变量及其值在函数中都依然有效
- 如果变量在函数内被赋予了新值，那么在脚本中引用该变量时，新值也依然有效

###### 局部变量
&#8195;&#8195;如果一个函数调用了一个变量，并赋予了新值，接下来另外一个函数需要调用变量，并且是需要原来的值，此时调用却会是新的值，采用局部变量可以解决此问题。脚本示例如下：
```sh
#!/bin/bash
test1 () {
    local tmp=$[ $value +2]
    result=$[ $tmp * 2 ]
}
tmp=3;value=5;test1
echo "The result is $result"
if [ $tmp -gt $value ]
then
    echo "Tmp is larger!"
else
    echo "Tmp is smaller!"
fi
```
运行后示例如下：
```
[root@redhat8 function]# sh test7.sh
The result is 14
Tmp is smaller!
```
### 数组变量和函数
##### 向函数传递数组参数
&#8195;&#8195;如果直接将数组作为函数参数进行传递，那么函数只会读取数组变量的第一个值。必须将数组变量的值分解成单个的值，然后将这些值作为函数的参数使用，在函数内部，可以将所有的参数重新组合成一个新的变量。示例如下：
```sh
#!/bin/bash
function test2 {
    local newarray1
    newarray1=($(echo "$@"))
    echo "The new array value is: ${newarray1[*]}"
}
array1=(1 2 3 4)
echo "The original array is ${array1[*]}"
test2 ${array1[*]}
addarray () {
    local sum=0
    local newarray2
    newarray2=($(echo "$@"))
    for value in ${newarray2[*]}
    do
        sum=$[ $sum + $value ]
    done
    echo $sum
}
myarray=(4 3 2 1)
echo "The array is : ${myarray[*]}"
arg1=$( echo ${myarray[*]})
result=$(addarray $arg1)
echo "The result is $result"
```
运行后示例如下：
```
[root@redhat8 function]# sh test8.sh
The original array is 1 2 3 4
The new array value is: 1 2 3 4
The array is : 4 3 2 1
The result is 10
```
说明：在函数内部，数组仍然可以像其它数组一样使用，上面示例中就使用for循环遍历了数组。

##### 从函数返回数组
&#8195;&#8195;函数使用echo语句来按正确顺序输出单个数组值，然后脚本再将它们重新放进一个新的数组变量中，示例如下：
```sh
#!/bin/bash
arraydblr () {
    local origarray
    local newarray
    local elements
    local i
    origarray=($(echo "$@"))
    newarray=($(echo "$@"))
    elements=$[ $# - 1 ]
    for (( i =0; i <= $elements; i++ ))
    {
        newarray[$i]=$[ ${origarray[$i]} * 2 ]
    }
    echo ${newarray[*]}
}
myarray=(2 7 4 9 1)
echo "The original array is :${myarray[*]}"
arg1=$(echo ${myarray[*]})
result=($(arraydblr $arg1))
echo "The new array is ${result[*]}"
```
运行后输出示例如下：
```
[root@redhat8 function]# sh test9.sh
The original array is :2 7 4 9 1
The new array is 4 14 8 18 2
```
### 函数递归
&#8195;&#8195;局部函数变量的一个特性是自成体系，除了从脚本命令行获得的变量，自成体系的函数不需要使用任何外部资源。这个特性使得函数可以递归的调用，也就是函数可以调用自己来得到结果。           
&#8195;&#8195;通常递归函数都一个最终可迭代到的基准值。最经典例子就是计算阶乘，例如阶乘：`3！=1*2*3`,用递归方程可以写成：`x!=x * (x-1)!`,脚本示例如下：
```sh
#!/bin/bash
factorial () {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local tmp=$[ $1 - 1 ]
        local result=$(factorial $tmp)
        echo $[ $result * $1 ]
    fi
}
read -p "Please enter number: " num
result=$(factorial $num)
echo "The factorial of enter number is $result"
```
运行后输出示例如下：
```
Please enter number: 9
The factorial of enter number is 362880
```
### 创建库
&#8195;&#8195;当需要在多个脚本中使用同一段代码时候，可以创建一个函数库文件，在多个脚本中引用该库文件。首先是创建一个包含所需函数的公用库文件，例如testfuncs库文件，定义了几个简单的函数：
```sh
function addem {
    echo $[ $1 + $2 ]
}
multem () {
    echo $[ $1 * $2 ]
}
divem () {
    fi [ $2 -ne 0 ]
    then
        echo $[ $1 / $2 ]
    else
        echo -1
    fi
}
```
&#8195;&#8195;shell函数同样有作用域问题，和环境变量一样，运行testfuncs shell脚本，会创建一个新的shell并在其中运行这个脚本。当像普通脚本那样运行库文件，函数并不会出现在脚本中。使用函数库的关键在于source命令，source命令会在当前shell上下文中执行命令，而不是创建一个新的shell。source命令有个快捷的别名，称作点操作符（dot operator），调用刚才库文件的脚本示例如下：
```sh
#!/bin/bash
. ./testfuncs
num1=10
num2=5
result1=$(addem $num1 $num2)
result2=$(multem $num1 $num2)
result3=$(divem $num1 $num2)
echo "The result of adding them is:$result1"
echo "The result of adding them is:$result2"
echo "The result of adding them is:$result3"
```
运行后输出示例如下：
```
[root@redhat8 function]# sh test11.sh
The result of adding them is:90
The result of adding them is:176
The result of adding them is:44
```
### 命令行上使用函数
采用单号方式进行函数定义，示例如下：
```
[root@redhat8 function]# function multem  { echo $[ $1 * $2 ];}
[root@redhat8 function]# multem 9 9
81
[root@redhat8 function]# doubleit () { read -p "Enter number: " num;echo $[ $num * 2 ];}
[root@redhat8 function]# doubleit
Enter number: 9
18
```
注意事项：如果给函数起了一个内建命令或与另一命令相同的名字，函数会覆盖原来的命令。

### 在.bashrc文件中定义函数
&#8195;&#8195;命令行上定义的函数在退出shell时函数就消失了，可以将函数定义在一个特定的位置，每次启动一个新的shell时候，都会有shell重新载入，这个最佳位置就是.bashrc文件，不管是交互式shell还是从现有的shell中启动新的shell，都会在主目录下查找这个文件。          
&#8195;&#8195;在RHEL中，文件.bash在用户的home目录下，里面有预定义一些东西，不要删除，把个人写的函数放在文件末尾就行了，文件.bashrc的示例如下：
```
[huang@redhat8 ~]$ pwd
/home/huang
[huang@redhat8 ~]$ cat .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

# User specific environment
PATH="$HOME/.local/bin:$HOME/bin:$PATH"
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
```
&#8195;&#8195;在.bashrc文件中加入如下函数或者库文件，并且用source（点操作符）调用（注意会在下次重新启动bash shell时候生效）：
```sh
addtest () {
    echo $[ $1 + $2 ]
}
. /shell/function/testfuncs
```
在命令行调用示例如下：
```
[huang@redhat8 function]$ addtest 2 4
6
[huang@redhat8 function]$ addem 3 13
16
[huang@redhat8 function]$ divem 81 9
9
```
