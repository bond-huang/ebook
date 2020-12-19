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
格式化打印命令printf格式：
```sh
printf "format string",var1,var2 ...
```
说明：
- format string是格式化输出的关键，会用文本元素和格式化指定符来具体指定如何呈现格式化输出
- 格式化指定符是一种特殊的代码，会指明显示什么类型的的变量以及如何显示
- gawk程序会将每个格式化指定符作为占位符，供命令中的变量使用
- 第一个格式化指定符对应列出的第一个变量，第二个对应第二个变量

格式化指定符格式如下：
```sh
%[modifier]control-letter
```
说明：
- control-letter是一个单字符代码，用于指明显示说明类型的数据
- modifier定义了可选的格式化特性

下表是可用在格式化指定符中的控制字母：

控制字母|描述
:---|:---
c|将一个数作为ASCII字符显示
d|显示一个整数值
i|显示一个整数值（跟d一样）
e|用科学计数法显示一个数
f|显示一个浮点值
g|用科学计数法或浮点数显示（选择较短的格式）
o|显示一个八进制值
s|显示一个文本字符串
x|显示一个十六进制值
X|显示一个十六进制值，但用大写字母

示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{
> x = 100 * 1000
> printf "The answer is: %e\n",x
> }'
The answer is: 1.000000e+05
```
除了控制字母外，还有3种修饰符可以用来进一步控制输出：
- width：指定了输出字段最小宽度的数字值，如果输出短于这个值，printf会将文本右对齐，并用空格填充；如果输出比指定的宽度要长，则按实际的长度输出
- prec：这是一个数字值，指定了浮点数中小数点后面位数，或文本字符串中显示的最大字符数
- &#45;（减号）：指明在向格式化空间中放入数据时采用左对齐而不是右对齐

在之前学习过程中，使用print命令来显示数据行中的数据字段：
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
使用printf命令来进行格式化输出：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS="\n";RS=""}{printf "%s %s\n", $1,$4}' test3
Bond Huang 0755-12345678
Bond Huang 0769-87654321
```
示例说明：
- 使用printf会产生跟print命令相同的输出，printf命令用`%s`格式化指定符来作为这两个字符串值的占位符 
- 注意，需要在printf命令末尾手动添加换行符来生成新行，如果没添加，会继续在同一行打印后续输出

如果需要用几个单独的printf名留在同一行打印多个输出，就非常有用：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS=","}{printf "%s ",$1} END{printf "\n"}' test4
data11 data21 data31 
```
示例说明：
- 每个printf的输出都出现在同一行
- 为了终止该行，END部分打印了一个换行符

用装饰符来格式化第一个字符串值：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS="\n";RS=""}{printf "%16s %s\n", $1,$4}' test3
      Bond Huang 0755-12345678
      Bond Huang 0769-87654321
```
示例说明：
- 通过添加一个值为16的修饰符，强制将第一个字符串的输出宽度位16个字符
- 默认情况下，printf命令使用右对齐来将数据放到格式化空间中
- 如果要改成左对齐，只需给修饰符加一个减号即可

示例如下：
```
[root@redhat8 gawk]# gawk 'BEGIN{FS="\n";RS=""}{printf "%-16s %s\n", $1,$4}' test3
Bond Huang       0755-12345678
Bond Huang       0769-87654321
```
printf命令处理浮点值也非常方便，：
```
[root@redhat8 gawk]# gawk '{total = 0
> for (i = 1;i < 4;i++)
> {total += $i}
> avg = total / 3
> printf "Average: %5.1f\n",avg
> }' test6
Average: 135.0
Average: 137.7
Average: 180.3
```
示例说明：
- 通过为变量指定一个格式，可以让输出看起来更统一
- 示例中使用`%5.1f`格式指定符来强制printf命令将浮点值近似到小数后一位

## 内建函数
### 数学函数
gawk中内建的数学函数如下表所示：

函数|描述
:---|:---
atan2(x,y)|x/y的反正切，x和y以弧度为单位
cos(x)|x的余弦，x以弧度为单位
exp(x)|x的指数函数
int(x)|x的整数部分，取靠近零一侧的值
log(x)|x的自然对数
rand()|比0大比1小的随机浮点值
sin(x)|x的正弦，x以弧度为单位
sqrt(x)|x的平方根
srand(x)|为计算随机数指定一个种子值

产生较大整数随机数可以使用rand()函数和int()函数创建一个算法：
```sh
x = int(10 * rand())
```
gawk对于它能够处理的数值有一个限定区域，示例如下：
```
[root@redhat8 shell]# gawk 'BEGIN{x=exp(100);print x}'
26881171418161356094253400435962903554686976
[root@redhat8 shell]# gawk 'BEGIN{x=exp(1000);print x}'
gawk: cmd. line:1: warning: exp: argument 1000 is out of range
inf
```
gawk还支持一些按位操作数据的函数：
- and(v1,v2):执行v1和v2的按位与运算
- compl(val):执行val的补运算
- lshift(val,count):将val左移count位
- or(v1,v2):执行v1和v2的按位或运算
- rshift(val,count):将val右移count位
- xor(v1,v2):执行v1和v2的按位异或运算

### 字符串函数
gawk提供的处理字符串的函数如下表所示：

函数|描述
:---|:---
asort(s,&#91;,d&#93;)|将数组s按数据元素值排序。索引值会被替换成表示新的排序顺序的连续数字；如果指定了d则排序后的数组会存储在数组d中
asorti(s,&#91;,d&#93;)|将数组s按索引值排序。生成的数组会将索引值作为数据元素值，用连续数字索引来表明排序顺序；如果指定了d则排序后的数组会存储在数组d中
gensub(r,s,h&#91;,t&#93;)|查找变量`$0`或目标字符串t(如有指定)来匹配正则表达式r。如果h是一个以g或G开头的字符串，就用s替换掉匹配的文本；如果h是一个数字，则表示要替换掉第h处r匹配的地方
gsub(r,s&#91;,t&#93;)|查找变量`$0`或目标字符串t(如有指定)来匹配正则表达式r。如果找到了，就全部替换成字符串s
index(s,t)|返回字符串t在字符串s中的索引值，如果没找到就返回0
length(&#91;s&#93;)|返回字符串s的长度，如果没有指定，则返回`$0`长度
match(s,r&#91;,a&#93;)|返回字符串s中正则表达式r出现位置的索引，如果指定了数组a，会存储s中匹配的正则表达式的那部分
split(s,a&#91;,r&#93;)|将s用FS字符或正则表达式r(如有指定)分开放到数组a中。返回字符串的总数
sprintf(format,variables)|用提供的format和variables返回一个类似于printf输出的字符串
sub(r,s&#91;,t&#93;)|在变量`$0`或目标字符串t中查找正则表达式r的匹配，如果找到了，就用字符串s替换掉第一处匹配
substr(s,i&#91;,n&#93;)|返回s中从索引值i开始的n个字符组成的子字符串。如果未提供n，则返回s剩下的部分
tolower(s)|将s中的所有字符转换成小写
toupper(s)|将s中的所有字符转换成大写

toupper(s)和length(&#91;s&#93;)使用示例：
```
[root@redhat8 shell]# gawk 'BEGIN{
> a = "superuser";print toupper(a);print length(a) }'
SUPERUSER
9
```
asort(s,&#91;,d&#93;)示例如下：
```
[root@redhat8 shell]# gawk 'BEGIN{
> var["a"] = 1;var["d"] = 2
> var["m"] = 3;var["t"] = 4
> asort(var,test)
> for (i in test)
> print"Index:",i," - value:",test[i]}'
Index: 1  - value: 1
Index: 2  - value: 2
Index: 3  - value: 3
Index: 4  - value: 4
```
split函数是将数据字段放到数组中以供进一步处理好方法：
```
[root@redhat8 gawk]# cat test4
data11,data12,data13,data14,data15
data21,data22,data23,data24,data25
data31,data32,data33,data34,data35
[root@redhat8 gawk]# gawk 'BEGIN{FS=","}{
split($0,var);print var[1],var[5]}' test4
data11 data15
data21 data25
data31 data35
```
新的数组使用连续数字作为数组索引，从含有第一个数据字段的索引值1开始。
### 时间函数
&#8195;&#8195;gawk包含一些函数来帮助处理时间，时间函数常用于处理日志文件，日志文件常含有需要比较的日期，可以转换成时间戳进行比较。gawk提供的时间函数如下表所示：

函数|描述
:---|:---
mktime(datespec)|将一个按YYYY MM DD HH MM SS&#91;DST&#91;格式指定的日期转换成时间戳
strftime(format &#91;,timestamp&#91;)|将当前时间的时间戳或timestamp（如有提供）转化格式化日期（采用shell函数date()的格式）
systime()|返回当前时间的时间戳

使用示例：
```
[root@redhat8 gawk]# gawk 'BEGIN{
> date = systime()
> day = strftime("%A,%B %d,%Y",date)
> print day}'
Saturday,December 19,2020
```
示例说明：
- 示例中使用systime函数从系统获取当前的epoch时间戳
- 然后用strftime函数转换成用户可读的格式，使用了shell命令date的日期格式化字符

## 自定义函数
### 定义函数
使用function关键字定义函数（函数名必须唯一）：
```sh
function neme ([variables])
{
    statements
}
```
在调用的gawk程序中可以传给这个函数一个或多个变量。
```sh
function printchird()
{
    print $3
}
```
上面示例会打印记录中的第三个数据字段。
```sh
function myrand(limit)
{
    return int(limit * rand())
}
x = myrand(100)
```
示例说明：
- 函数可以使用return语句返回值，值可以是变量，或者是最终能计算处值的算式
- 可以间该函数的返回值赋给gawk程序中的一个变量，这个变量包含函数的返回值

### 使用自定义函数
示例如下：
```
[root@redhat8 gawk]# gawk 'function myprint()
> {printf "%-16s - %s\n",$1,$4}
> BEGIN{FS="\n";RS=""}
> {myprint()}' test3
Bond Huang       - 0755-12345678
Bond Huang       - 0769-87654321
```
示例说明：
- 在定义函数时，它必须出现在所有代码块之前（包括BEGIN代码块）
- 示例中定义了myeprint()函数，函数会格式化记录中的第一个和第四个数据字段以供打印输出
- gawk程序然后用该函数显示输出数据文件中的数据

### 创建函数库
&#8195;&#8195;gawk提供了一种途径来将多个函数放到一个库文件中，就可以在所有的gawk程序中使用了。首先需要创建一个存储所有gawk函数的文件，示例如下：
```
[root@redhat8 gawk]# cat funclib
function myprint()
{
    printf "%-16s - %s\n",$1,$4
}
function printchird()
{
    print $3
}
function myrand(limit)
{
    return int(limit * rand())
}
```
使用示例：
```
[root@redhat8 gawk]# cat script
BEGIN{FS="\n";RS=""}
{
	myprint()
}
[root@redhat8 gawk]# gawk -f funclib -f script test3
Bond Huang       - 0755-12345678
Bond Huang       - 0769-87654321
```
示例说明：
- 使用`-f`命令行参数可以使用函数库中的函数，但是不能将其和内敛gawk脚本在一起使用
- 需要创建一个包含gawk程序的文件，然后在命令行上同时指定库文件和程序文件
- 可以在同一个命令行中使用多个`-f`参数

## 实例
为方便查阅，收录在以下位置：[Shell-sed&gawk实例](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/02-Shell%E8%84%9A%E6%9C%AC%E5%BF%AB%E9%80%9F%E6%8C%87%E5%8D%97/05-Shell-sed&gawk%E5%AE%9E%E4%BE%8B.html)
