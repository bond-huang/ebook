# Shell笔记-gawk程序
## 使用变量
gawk编程语言支持两种不同类型的变量：内建变量和自定义变量。
### 内建变量
gawk程序使用内建变量来引用程序数据的一些特殊功能。
#### 字段和记录分隔符变量
&#8195;&#8195;之前学习了内建变量中的数据字段变量，可以使用美元符号（$）和字段在该记录中的位置值来引用记录对应的字段。默认情况下，字段分隔符是一个空白字符，空格或者制表符，可以使用命令行参数`-F`或者在gawk程序中使用特殊内建变量FS来更改字段分隔符。gawk数字字段和记录变量：

变量|描述
:---|:---
FIELDWIDTHS|由空格分隔的一列数字，定义了每个数据字段确切宽度
FS|输入字段分隔符
RS|输入记录分隔符
OFS|输出字段分隔符
ORS|输出记录分隔符

&#8195;&#8195;变量FS和OFS定义了gawk如何处理数据流中的数据字段，FS来定义记录中的字段分隔符，OFS功能一致只是用在print输出上，示例如下：
```
[root@redhat8 gawk]# cat test1
Thor,Hulk,Captain America,Iron man
Batman,Wonder Woman,Super Man
[root@redhat8 gawk]# gawk 'BEGIN{FS=","} {print $1,$2,$3}' test1
Thor Hulk Captain America
Batman Wonder Woman Super Man
[root@redhat8 gawk]# gawk 'BEGIN{FS=",";OFS="<->"}{print $1,$2,$3}' test1
Thor<->Hulk<->Captain America
Batman<->Wonder Woman<->Super Man
```
说明：
- 默认情况下，gawk会将OFS设置成一个空格
- 通过OFS变量，可以在输出中使用任意字符串来分隔字段

&#8195;&#8195;FIELDWIDTHS变量允许不依靠字段分隔符来读取记录，一些应用程序中，数据没有使用字段分隔符，而是放置在记录中的特定列，需要设定FIELDWIDTHS变量来匹配数据在记录中的位置，示例如下：
```
[root@redhat8 gawk]# cat test2
1103.1418866.66
911-5.328499.00
02711.1133100.9
[root@redhat8 gawk]# gawk 'BEGIN{FIELDWIDTHS="3 5 2 5"}
> {print $1,$2,$3,$4}' test2
110 3.141 88 66.66
911 -5.32 84 99.00
027 11.11 33 100.9
```
说明：
- 一旦设置了FIELDWIDTHS变量，gawk就会忽略FS变量，并根据提供的字段宽度来计算字段
- 示例中FIELDWIDTHS变量定义了四个字段，gawk依此来解析数据记录
- 一旦设置了FIELDWIDTHS变量的值，就不能再改变了，这种方法不适用于边长的字段

&#8195;&#8195;变量RS和ORS定义了gawk程序如何处理数据流中的记录，默认情况下都是换行符，RS的默认值表明输入数据流中的每行新文本就是一条新记录，示例如下：
```
[root@redhat8 gawk]# cat test3
Bond Huang
123 ZhongXin street
ShenZhen,518000
0755-12345678

Bond Huang
321 HongFu street
DongGuan,523000
0769-87654321
[root@redhat8 gawk]# gawk 'BEGIN{FS="\n";RS=""}{print $1,$4}' test3
Bond Huang 0755-12345678
Bond Huang 0769-87654321
```
说明：
- 示例中一个字段占据多行，使用默认的FS和RS时候gawk会把每行作为一个单独的记录，并将记录中的空格当作字段分隔符
- 示例中将FS设置成换行符，表示数据流中每行都是一个单独的字段，每行所有数据都是同一个字段
- 把RS设置成空字符串，然后在记录间保留一个空白行，gawk就会把每个空白行当作一个记录分隔符

#### 数据变量
下表列出gawk中的其它内建变量：

变量|描述
:---|:---
ARGC|当前命令行参数个数
ARGIND|当前文件在ARGV中的位置
ARGV|包含命令行参数的数组
CONVFMT|数字的转换个数(参见printf语句)默认值为%.6 g
ENVIRON|当前shell环境变量及其值组成的关联数组
ERRNO|当读取或关闭输入文件发生错误时的系统错误号
FILENAME|用作gawk输入数据的数据文件的文件名
FNR|当前数据文件中的数据行数
IGNORECASE|设成非零值时，忽略gawk命令中出现的字符串的字符大小写
NF|数据文件中的字段总数
NR|已处理的输入记录数
OFMT|数字的输出格式，默认值为%.6 g
RLENGTH|由match函数所匹配的子字符串的长度
RSTART|由match函数所匹配的子字符串的起始位置

ARGC和ARGV变量允许从shell中获得命令行参数总数以及大妈的值：
```
[root@redhat8 gawk]# gawk 'BEGIN{print ARGC,ARGV[1]}' test1
2 test1
```
说明：
- gawk并不会将程序脚本当成命令行参数的一部分，所以示例中两个参数是gawk命令和test1参数
- ARGV数组从索引0开始，代表的是命令，第一个数组值是gawk命令后的第一个命令行参数
- 在脚本中引用gawk变量时候，变量名前面不用加美元符，跟shell变量不一样

ENVIRON变量关联数组来提取shell环境变量：
```
[root@redhat8 gawk]# gawk 'BEGIN{print ENVIRON["HOME"]
> print ENVIRON["PATH"]}'
/root
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
说明：
- 关联数组用文本作为数组的索引值，而不是数值；数组索引中的文本是shell的环境变量名，而数组的值则是shell环境变量的值
- 变量`ENVIRON["HOME"]`从shell中提取了HOME环境变量的值，这种方法可以从shell中提取任何环境变量的值

当在gawk程序中跟踪数据字段和记录时，变量FNR、NR和NF用起来非常方便：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS=":";OFS=":"}
> {print $1,$NF}' /etc/passwd
root:/bin/bash
bin:/sbin/nologin
daemon:/sbin/nologin
adm:/sbin/nologin
lp:/sbin/nologin
```
说明：
- 变量NF可以在当不知道具体位置情况下可以指定最后一个字段数据，不管数据到底有多少个数据字段
- 变量NF含有数据文件最后一个数据段的数据，可以加美元符号作为字段变量

&#8195;&#8195;变量FNR和NR变量类似，也有所区别，FNR变量含有当前数据文件中已处理过的记录数，NR变量则含有已处理过的记录总数，示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS=","}
> {print $1,"FNR="FNR}' test1 test1
Thor FNR=1
Batman FNR=2
Thor FNR=1
Batman FNR=2
[root@redhat8 gawk]# gawk 'BEGIN{FS=","}
> {print $1,"FNR="FNR,"NR="NR}
> END{print "There ware",NR,"records processed"}' test1 test1
Thor FNR=1 NR=1
Batman FNR=2 NR=2
Thor FNR=1 NR=3
Batman FNR=2 NR=4
There ware 4 records processed
```
说明：
- 第一个示例中gawk程序的命令行定义了两个同样的输入文件，脚本会打印第一个数据字段的值和FNR变量的当前值
- 第一个示例中当gawk程序处理第二个数据文件时，FNR值被设回了1
- 第二个示例中，FNR值在处理第二个数据文件时候被重置了，但是NR变量值在处理第二个数据文件时候继续计数
- 如果只使用一个数据文件作为输入，FNR和NR的值是相同的，如果处理多个，FNR值处理每个数据文件时候会被重置，NR的值回继续计数知道处理完所有数据文件

### 自定义变量
&#8195;&#8195;gawk自定义变量可以是任意数目的字母、数字和下划线，但是不能以数字开头，并且区分大小写。
#### 在脚本中给变量赋值
在gawk中给变量赋值一样用赋值语句：
```
[root@redhat8 gawk]# gawk 'BEGIN{test="Miracles happen every day"
> print test;test=189;print test}'
Miracles happen every day
189
[root@redhat8 gawk]# gawk 'BEGIN{x=5;x=x*3+1;print x}'
16
```
说明：
- 一般建议不同的gawk命令放到不同的行，示例中写在一行实际中不推荐
- 第一个示例中print语句输出的是test变量的当前值，gawk变量可以保存数值或文本值，可以重新进行赋值
- 第二个示例中赋值语句包含了数学算式来处理数字值

#### 在命令行上给变量赋值
&#8195;&#8195;可以用gawk命令行来给程序中的变量赋值，允许在正常的代码之外赋值，即使改变变量的值，示例如下：
```
[root@redhat8 gawk]# cat script1
BEGIN{FS=","}
{print $n}
[root@redhat8 gawk]# gawk -f script1 n=1 test1
Thor
Batman
[root@redhat8 gawk]# gawk -f script1 n=3 test1
Captain America
Super Man
```
说明：这个特性可以在不改变脚本代码的情况下就可以改变脚本的行为。

但有一个问题，示例如下：
```
[root@redhat8 gawk]# cat script2
BEGIN{print "The starting value is",n; FS=","}
{print $n}
[root@redhat8 gawk]# gawk -f script2 n=3 test1
The starting value is 
Captain America
Super Man
[root@redhat8 gawk]# gawk -v n=3 -f script2 test1
The starting value is 3
Captain America
Super Man
```
说明：
- 第一个示例中，设置了变量后，这个值在代码的BEGIN部分不可用
- 第二个示例中用`-v`命令行参数解决了此问题，允许在BEGIN代码之前设定变量，`-v`必须放在脚本代码之前

## 处理数组
gawk编程语言使用关联数组提供数组功能，特点：
- 关联数组跟数字数组不同之处在于它的索引值可以是任意的文本字符串
- 不需要用连续的数字来标识数组中的数据原始，相反，关联数组用各种字符串来引用值
- 每个索引字符都必须能够唯一的标识出赋给它的数据元素（跟散列列表和字典概念相同）

### 定义数组变量
&#8195;&#8195;使用标准复制语句来定义数组变量：`var[index] = element`,其中var是变量名，index是关联数组的索引值，element是数据元素值。示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{
> superhero["Asgard"] = "Thor"
> print superhero["Asgard"]
> }'
Thor
[root@redhat8 gawk]# gawk 'BEGIN{num[1] = 10;num[2] = 5
> total = num[1] + num[2]
> print total }'
15
```
### 遍历数组变量
在gawk中遍历一个关联数组，可以用for语句的一种特殊形式：
```sh
for (var in array)
{
    statements
}
```
&#8195;&#8195;此for语句会在每次循环时将关联数组array的下一个索引值赋给遍历var，然后执行一遍statements，变量中存储的是索引值而不是数组元素值，可以将这个变量用作数组的所有，提取出数据元素值，示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{
> var["a"] = 1
> var["b"] = 2
> var["c"] = 3
> for (test in var)
> {
> print "Index:",test," - value :",var[test]
> }
> }'
Index: a  - value : 1
Index: b  - value : 2
Index: c  - value : 3
```
说明：
- 索引值不会按任何特定的顺序返回，但它们都能够指向对应的数据元素值

### 删除数组变量
格式：`delete array[index]`,示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{
> var["a"] = 1;var["g"] = 2
> delete var["g"]
> for (test in var)
> print "Index:",test," -Value:",var[test]
> }'
Index: a  -Value: 1
```
注意：一旦从关联数组中删除了索引值，就没法再用它来提取元素。

## 使用模式
gawk程序支持多种类型的匹配模式来过滤数据记录，和sed编辑器类似。
### 正则表达式
在使用正在表达式时，正则表达式必须出现在它要控制的程序脚本的左花括号前：
```
[root@redhat8 gawk]# cat test1
Thor,Hulk,Captain America,Iron man
Batman,Wonder Woman,Super Man
[root@redhat8 gawk]# gawk 'BEGIN{FS=","} /Th/{print $1}' test1
Thor
```
说明：
- gawk程序会用正则表达式对记录中的所有数据字段进行匹配，包括字段分隔符
- 如果需要用正则表达式匹配某个特定的数据示例，应该使用匹配操作符

### 匹配操作符
&#8195;&#8195;匹配操作符（matching operator）允许将正则表达式限定在记录中的特定数据字段，匹配操作符是波浪线（~）。可以指定匹配操作符、数据字段变量以及要匹配的正则表达式：`$1 ~ /^data/`。
- $1变量代表记录中的第一个数据段
- 这个表达式会过滤出第一个字段以文本data开头的所有记录

示例如下：
```
[root@redhat8 gawk]# cat test4
data11,data12,data13,data14,data15
data21,data22,data23,data24,data25
data31,data32,data33,data34,data35
[root@redhat8 gawk]# gawk 'BEGIN{FS=","} $2 ~ /^data2/{print $0}' test4
data21,data22,data23,data24,data25
[root@redhat8 gawk]# gawk -F: '$1 ~ /huang/{print $1,$NF}' /etc/passwd
huang /bin/bash
```
说明：
- 匹配操作符会用正则表达式`/^data2`来比较第二个数据段，正则表达式指明字符串要以文本data2开头
- 第二个示例中会在第一个数据字段中查找文本huang，如果在记录中找打了，会打印该记录的第一个和最后袷数据字段值

可以使用`!`符号来排除正则表达式的匹配：`$1 !~ /expression/`，示例如下：
```
[root@redhat8 gawk]# gawk -F: '$1 !~ /huang/{print $1,$NF}' /etc/passwd
root /bin/bash
bin /sbin/nologin
daemon /sbin/nologin
adm /sbin/nologin
...
```
### 数学表达式
例如想显示所有属于root用户组（组ID为0）的用户，脚本示例：
```
[root@redhat8 gawk]# gawk -F: '$4 == 0{print $1}' /etc/passwd
root
sync
shutdown
halt
operator
```
常见数学比较表达式：
- x == y:值x等于y
- x <= y:值x小于等于y
- x < y:值x小于y
- x >= y:值x大于等于y
- x > y:值x大于y

对文本数据使用表达式示例：
```
[root@redhat8 gawk]# gawk -F, '$1 == "data"{print $1}' test4
[root@redhat8 gawk]# gawk -F, '$1 == "data21"{print $1}' test4
data21
```
说明：
- 对用文本数据使用表达式时，表达式必须完全匹配，数据必须跟模式严格匹配
- 示例中第一个匹配没用记录，说明没有匹配到，第二个精准匹配到了一条记录

## 结构化命令
### if语句
gawk编程语言支持标准的if-then-else格式的if语句：
- 必须为if语句定义一个求职的条件，并将其用圆括号括起来
- 如果条件求值为TURE，紧跟在if语句后面的语句会执行
- 如果条件求值为FALSE，这条语句就会被跳过

格式：
```sh
if (condition)
    statement1
```
或者：
```
if (condition) statement1
```
可以在单行上使用esle自居，但必须在if语句部分之后使用分号：
```sh
if (condition) statement1; else statement2
```
示例如下：
```
[root@redhat8 gawk]# cat test5
1
8
4
[root@redhat8 gawk]# gawk '{if ($1 > 5) print $1}' test5
8
[root@redhat8 gawk]# gawk '{
> if ($1 > 5)
> {
> a = $1 * 5
> print a
> }
> }' test5
40
[root@redhat8 gawk]# gawk '{
> if ($1 > 5)
> {
> a = $1 * 5
> print a
> } else
> {a = $1 + 100
> print a
> }}' test5
101
40
104
[root@redhat8 gawk]# gawk '{if($1>5) print $1 * 5;else print $1 + 100}' test5
101
40
104
```
### while语句
while语句格式：
```sh
while (condition)
{
    statements
}
```
&#8195;&#8195;while循环允许遍历一组数据，并检查迭代的结束条件，例如在计算中必须使用每条记录中的多个数据值，示例如下：
```
[root@redhat8 gawk]# cat test6
150 120 135
160 113 140
145 180 216
[root@redhat8 gawk]# gawk '{total = 0;i = 1
> while(i < 4)
> {total += $i;i++}
> avg = total / 3
> print "Average:",avg
> }' test6
Average: 135
Average: 137.667
Average: 180.333
```
示例说明：
- while语句会遍历记录中的数据字段，将每个值都加到total变量上，并将计数器变量i增值
- 当计数器值等于4时，while的条件变成了FALSE，循环结束，然后执行脚本中下一条语句，及计算并打印平均值
- 以上过程会在数据文件中的每条记录上不断重复

gawk支持在while循环中使用break语句和continue语句：
```
[root@redhat8 gawk]# gawk '{total = 0;i = 1
> while(i < 4)
> {total += $i
> if (i == 2)
> break
> i++}
> avg = total / 2
> print "The average of the first two data is:",avg
> }' test6
The average of the first two data is: 135
The average of the first two data is: 136.5
The average of the first two data is: 162.5
```
### do-while语句
&#8195;&#8195;do-while语句类似于while语句，但会在检查条件语句之前执行命令,格式如下：
```sh
do
{
    statements
}while (condition)
```
&#8195;&#8195;这种格式保证了语句在条件被求值之前至少执行一次，当需要在求值条件前执行语句是非常方便：
```
[root@redhat8 gawk]# gawk '{total = 0;i = 1
> do
> {total += $i;i++
> }while (total < 150)
> print total }' test6
150
160
325
```
示例说明：
- 示例脚本会读取每条记录的数据字段并将它们加在一起，直到累加结果达到150
- 如果第一个数据字段大于等于150，例如第一条和第二条记录中的一个数据字段，脚本会保证在条件被求值前至少读取第一个数据字段的内容

### for语句
gawk支持C语言风格的for循环：
```sh
for ( variable assignment; condition; iteration process)
```
将多个功能合并到一个语句有助于简化循环：
```
[root@redhat8 gawk]# gawk '{total = 0
> for (i = 1;i < 4; i++)
> {
> total += $i
> }
> avg = total / 3
> print "Average:",avg
> }' test6
Average: 135
Average: 137.667
Average: 180.333
```
## 格式化打印
