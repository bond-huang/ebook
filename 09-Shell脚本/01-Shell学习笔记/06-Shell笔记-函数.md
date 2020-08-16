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
