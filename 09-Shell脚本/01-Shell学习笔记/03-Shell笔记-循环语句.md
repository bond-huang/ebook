# Shell笔记-循环语句

### for循环
简单的一般直接在shell中写入，格式：`for var in list;do`,shell会提示输入命令，然后输入`done`就开始执行。例如我现在就在shell下创建几个测试脚本文件：
```
[root@redhat8 for]# for i in 1 2 3 4 5
> do
> touch test$i.sh
> done
```
标准格式如下：
```sh
for var in list 
do
    commands
done
```
##### 读取列表中的值
直接代码示例：
```sh
#!/bin/bash
for hero in "Captain America" Thor "Iron Man" 
do
    echo "$hero is a SuerpHero!"
done
list="Batman WonderWoman"
list=$list" SuperMan"
for hero in $list
do
    echo "$hero is a SuerpHero!"
done
```
说明：
- 列表中的内容用空格来划分，如果一个值本身有空格，用双引号括起来，并且shell不会把双引号当成内容的一部分
- 列表中的内容本身有引号，例如：I'am Iron man,同样可以用双引号括起来，也可以用转义字符(反斜杠)来转义
- 从变量中读取列表也可以，示例中也演示了向列表中添加元素的方法

运行后输出结果如下：
```
[root@redhat8 for]# sh test1.sh
Captain America is a SuerpHero!
Thor is a SuerpHero!
Iron Man is a SuerpHero!
Batman is a SuerpHero!
WonderWoman is a SuerpHero!
SuperMan is a SuerpHero!
```
##### 读取命令中的值
shell中可以用命令替换来执行任何能产生输出的命令，然后可以在for循环中使用命令的输出。   
新建一个文件SuperHero，查看内容：
```
[root@redhat8 for]# cat SuperHero
Thor
Captain America
Hulk
Iron Man
```
然后用代码来遍历：
```sh
#!/bin/bash
file="SuperHero"
IFS=$'\n'
for hero in $(cat $file)
do
    echo "The superhero is $hero!"
done
```
执行后输出如下：
```
[root@redhat8 for]# sh test2.sh
The superhero is Thor!
The superhero is Captain America!
The superhero is Hulk!
The superhero is Iron Man!
```
可以看到for将文本中的换行符作为字段分隔符，并不是之前列表示例中的空格，也没额外加上引号。这跟`IFS=$'\n'`这条命令有关。    
     
这里我们用到了特殊环境变量IFS(Iinternal Field Separator),IFS定义了一系列bash shell中用作字段分隔符，默认情况下，bash shell 的字段分隔符有：空格、制表符和换行符可以在脚本中临时改变一下IFS的值，例如只识别换行符，忽略空格和制表符：`IFS=$'\n'`。    
其它用法示例：
- 如果代码中多次用到IFS，后面的想用默认的，方法：
    - IFS.OLD=$IFS
    - IFS=$'\n' :代码中使用新的IFS
    - IFS=$IFS.OLD:后续的操作就使用默认的IFS
- 遍历冒号分隔的值（/etc/passwd），可以将IFS设置成冒号：IFS=:
- 需要指定多个字符的情况，串起来即可,例如换行符、冒号、分号和双引号一起：IFS=$'\n':;"    

##### 用通配符读取目录
代码示例如下：
```sh
#!/bin/bash
for file in /shell/for/hero/* /shell/for/hero/test
do
    if [ -f "$file" ]
    then
        echo "The $file is a file!"
    elif [ -d "$file" ]
    then
        echo "The $file is a directory!"
    else 
        echo "The $file is not exist!"
    fi
done
```
运行后输出如下：
```
[root@redhat8 for]# sh test3.sh
The /shell/for/hero/herodir is a directory!
The /shell/for/hero/SuperHero is a file!
The /shell/for/hero/test is not exist!
```
注意事项：
- 目录和文件名中有空格也是合法的，所以用分号将变量括起来："$file"
- 用到了之前的test命令中方括号方法，注意方括号前后要加空格，不然会报错

##### C语言风格的for
我试了下AIX中ksh不支持此用法，安装个bash就支持了。bash shell中示例如下：
```sh
#!/bin/bash
for ((i=1; i<=10;i++))
do 
    echo "The number is $i"
done
```
使用多个变量时候循环会单独处理每个变量，但是只能定义一个条件，示例：
```sh
#!/bin/bash
for ((i=1,j=3;i<=3;i++,j--))
do
    echo "$i + $j"
done
```
执行后结果如下：
```
bash-5.0# ./test1.sh
1 + 3
2 + 2
3 + 1
```
### while循环
while某种意义上是if-then和for循环的混合。定义一个要测试的命令，然后循环执行一组命令，只要定义的测试命令返回的退出码状态是0，返回非0时候，while会终止执行那组命令。基本格式如下：
```sh
while test command
do
    other command
done
```
多个测试命令的示例如下：
```sh
#!/bin/bash
num1=3
while echo $num1
        [ $num1 -ge 0 ]
do
    echo "This is a test loop!"
    num1=$[ $num1 - 1 ]
done
```
运行后输出如下：
```
[root@redhat8 for]# sh test4.sh
3
This is a test loop!
2
This is a test loop!
1
This is a test loop!
0
This is a test loop!
-1
```
注意：
- 示例中定义了两个测试命令，两个都要满足才会继续，
- 每次迭代中所有的测试命令都会执行，例如最后值是-1了，执行了`echo $num1`,第二个也执行了，只是返回为非零，条件不满足，while循环就终止了，这种用法要注意。

### until命令
until命令和while的工作方式完全相反。until要求指定一个通常返回非零退出码的测试命令，只有测试命令退出码不为零时，才会继续执行循环中列出的命令，直到测试命令返回的退出状态码为0，循环终止。
### 嵌套循环
被嵌套循环也称为内部循环(inner loop),被嵌套的循环会在外部循环的每次迭代中遍历一次它所有的值。     
##### while和for混用
示例代码：
```sh
#!/bin/bash
num1=3
while [ $num1 -ge 0 ]
do
    echo "Outer loop:$num1"
    for ((num2=1;num2<3;num2++))
    do
        num3=$[ $num1 + $num2 ]
        echo "  Inner loop:$num1+$num2 = $num3"
    done
    num1=$[ $num1 - 1 ]
done
```
echo命令中首行空两格可以在输出结果中缩进，运行后输出如下:
```
[root@redhat8 for]# sh test5.sh
Outer loop:3
  Inner loop:3+1 = 4
  Inner loop:3+2 = 5
Outer loop:2
  Inner loop:2+1 = 3
  Inner loop:2+2 = 4
Outer loop:1
  Inner loop:1+1 = 2
  Inner loop:1+2 = 3
Outer loop:0
  Inner loop:0+1 = 1
  Inner loop:0+2 = 2
```
##### while和until混用
示例代码：
```sh
#!/bin/bash
num1=2
until [ $num1 -eq 0 ]
do
    echo "Outer loop:$num1"
    num2=1
    while [ $num2 -lt 3 ]
    do
        num3=$(echo "scale=4; $num1 / $num2" | bc)
        echo "  Inner loop: $num1 / $num2 = $num3"
        num2=$[ $num2 + 1 ]
    done
    num1=$[ $num1 - 1 ]
done
```
运行后输出结果如下：
```
[root@redhat8 for]# sh test6.sh
Outer loop:2
  Inner loop: 2 / 1 = 2.0000
  Inner loop: 2 / 2 = 1.0000
Outer loop:1
  Inner loop: 1 / 1 = 1.0000
  Inner loop: 1 / 2 = .5000
```
### 循环处理文件数据
通常遍历文件中的数据，通常需要用到嵌套循环和IFS环境变量。比较典型的例子就是处理/etc/passwd里面的数据，示例代码如下：
```sh
#!/bin/bash
IFS=$'\n'
for user in $(cat /etc/passwd)
do
    echo "User information:$user -"
    IFS=:
    for value in $user
    do
        echo "  $value"
    done
done
```
执行后输入如下（太多了，只显示其中一个用户的）：
```
User information:huang:x:1000:1000:huang:/home/huang:/bin/bash -
  huang
  x
  1000
  1000
  huang
  /home/huang
  /bin/bash
```
### 循环控制
控制循环命令：break命令和continue命令
##### break命令
当shell执行break命令时，会尝试跳出当前正在执行的循环，代码示例如下：
```sh
#!/bin/bash
for i in {1..5}
do
    if [ $i -eq 3 ]
    then
        break
    fi
    echo "Iteration number:$i"
done
echo "The loop is end!"
```
运行后输出结果如下：
```
[root@redhat8 for]# sh test8.sh
Iteration number:1
Iteration number:2
The loop is end!
```
知识点：
- 这种方法同样适用于while和until循环。  
- 跳出内部循环的时候，把`break`写在内部循环的if语句中即可；   
- 如果在内部循环中要停止外部循环，break命令接受单个命令行参数值：`break n`。n指定了跳出循环的层次，默认情况下，n的值为1，表示跳出的是当前的循环，值为2的时候，break命令就会停止下一级的外部循环。

##### continue命令
continue命令可以提前中止某次循环中的命令，但是不会终止整个循环，示例如下：
```sh
#!/bin/bash
for ((i = 1; i <=8; i++ ))
do
    if [ $i -gt 3 ] && [ $i -lt 6 ]
    then
        continue
    fi
    echo "The number is $i !"
done 
```
运行后输出结果如下：
```
[root@redhat8 for]# sh test9.sh
The number is 1 !
The number is 2 !
The number is 3 !
The number is 6 !
The number is 7 !
The number is 8 !
```
可以看到大于3和小于6的项目被跳过了，if-then里面的条件满足，会跳过循环中剩余的命令，不满足时候，循环继续。    
同样适用于while和until循环，使用要小心，但是如果在某个条件里面对测试的条件变量进行增值，就会出现问题，会变成死循环，不停执行，示例代码如下：
```sh
#!/bin/bash
i=0
while echo "while interation:$i"
        [ $i -lt 10 ]
do
    if [ $i -gt 3 ] && [ $i -lt 6 ]
    then
        continue
    fi
    echo "  The number is $i !"
    i=$[ $i + 1 ]
done 
```
运行后最后会一直不停输出，无穷无尽：while interation:4

同样`continue`命令接受单个命令行参数值：`continue n`，n定义了要继续的循环的层级。

### 处理循环输出
代码示例如下：
```sh
#!/bin/bash
for file in /shell/for/*
do
    if [ -d "$file" ]
    then
        echo "$file is a directory!"
    else
        echo "$file is a file!"
    fi
done > output.txt
```
执行脚本后，不会显示在屏幕上，会将输出写过重定向到output.txt文件中，查看文件内容如下：
```
[root@redhat8 for]# cat output.txt
/shell/for/hero is a directory!
/shell/for/output.txt is a file!
/shell/for/SuperHero is a file!
/shell/for/test10.sh is a file!
/shell/for/test11.sh is a file!
/shell/for/test1.sh is a file!
```
如果在`done`后面加入`echo`的语句，shell会在`for`完成后在屏幕显示`echo`的内容。也可以使用管道符，例如在`done`后面加上`| sort`，就会对for循环的输出进行排序。
