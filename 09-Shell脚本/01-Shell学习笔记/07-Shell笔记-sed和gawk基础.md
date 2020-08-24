# Shell笔记-sed和gawk基础
这两个工具能够极大简化需要进行的数据处理任务。
### sed编辑器
&#8195;&#8195;sed编辑器被称作流编辑器（stream editor），和普通的交互式文本编辑器恰号相反。流编辑器会在编辑器处理数据之前基于预先提供的一组规则来编辑数据流。             
&#8195;&#8195;sed编辑器会根据命令来处理数据流中的数据，这些命令要么从命令行中输入，要么存储在一个命令文本文件中，sed编辑器会执行下列操作：
- 一次从输入中读取一行数据
- 根据所提供的编辑器命令匹配数据
- 按照命令修改流中的数据
- 将新的数据输出到STDOUT

说明：
- 在流编辑器将所有命令与一行数据匹配完毕后，它会读取下一行数据并重复这个过程，直到处理完所有数据行
- 命令是按顺序逐行给出，sed编辑器只需对数据流进行一遍处理就可以完成编辑操作

sed命令格式如下：
```
sed options script file
```
sed命令选项：

选项|描述
:---|:---
-e script|处理输入时，将script重指定的命令添加到已有得命令中
-f file|处理输入时，将file重指定的命令添加到已有的命令中
-n|不产生命令输出，使用print命令来完成输出

&#8195;&#8195;script参数指定了应用于流数据上的单个命令，如果需要使用多个命令，要么使用-e选项在命令行中指定，要么使用-f选项在单独的文件中指定。

##### 在命令行定义编辑器命令
默认情况下，sed编辑器会将指定的命令应用到STDIN输入流上，示例如下：
```
[root@redhat8 sed_gawk]# echo "I am Thor!" | sed 's/Thor/Iron Man/'
I am Iron Man!
[root@redhat8 sed_gawk]# cat test1
I am Captain America!
I am Captain America!
[root@redhat8 sed_gawk]# sed 's/Captain America/Iron Man/' test1
I am Iron Man!
I am Iron Man!
[root@redhat8 sed_gawk]# cat test1
I am Captain America!
I am Captain America!
[root@redhat8 sed_gawk]# sed -e 's/am/am not/; s/Captain America/Thanos/' test1
I am not Thanos!
I am not Thanos!
[root@redhat8 sed_gawk]# sed -e '
> s/am/am not/
> s/Captain America/Thanos/
> ' test1
I am not Thanos!
I am not Thanos!
[root@redhat8 sed_gawk]# cat test2.sed
s/am/am not/
s/Captain America/Thanos/
[root@redhat8 sed_gawk]# sed -f test2.sed test1
I am not Thanos!
I am not Thanos!
```
说明和注意：
- 在第一个示例中使用了`s`命令（substitute），`s`命令会使用斜线间指定第二个文本字符串来替换第一个文本字符串
- 第二个示例中示例修改整个文件，注意sed编辑器并不会修改文本文件的数据
- 第三个示例使用了多个编辑器命令，可以写在一行上，注意命令直接必须用分号隔开，并且在命令末尾和分号之间不能有空格
- 第四个示例采用多个编辑器命令，并且分行写入，注意要在封尾单引号所在行结束命令
- 第五个示例是从文件中读取编辑器命令，不用在每条命令后放一个分号
- 一遍建议将sed编辑器脚本文件以`.sed`作为文件的扩展名

### gawk程序
gawk程序是Unix中的原始awk程序的GNU版本，它提供了一种编辑语言而不只是编辑器命令，可以做的事情如下：
- 定义变量来保存数据
- 使用算数和字符串操作符来处理数据
- 使用结构化编程概念（if语句和循环）来为数据处理增加处理逻辑
- 通过提取数据文件的数据元素，将其重新排列或格式化，生成格式化报告

&#8195;&#8195;gawk最完美的例子是格式化日志文件，可以从日志文件中过滤出需要的数据元素，然后可以将其格式化，使得重要的数据更易于阅读。标准格式如下：
```
gawk options program file
```
gawk程序的可用选项：

选项|描述
:---|:---
-F fs|指定行中划分数据字段的字段分隔符
-f file|指定的文件中读取程序
-v var=value|定义gawk程序中的一个变量及其默认自
-mf N|指定要处理的数据文件中的最大字段数
-mr N|指定数据文件中的最大数据行数
-W keyword|是定gawk的兼容模式或警告等级

##### 从命令行读取程序脚本
示例如下：
```
[root@redhat8 sed_gawk]# gawk '{print "I am Iron Man!"}'
Who are you?
I am Iron Man!
Nice!     
I am Iron Man!
```
说明：
- gawk程序脚本用一对花括号来定义，必须将脚本命令放到两个花括号（{}）中
- 由于gawk命令假定脚本是单个文本字符串，所有还必须将脚本放到单引号中
- 示例程序定义了一个命令：print，它会将文本打印到STDOUT，但是执行后说明也没输出，原因在于没有在命令行上指定文件名，所有gawk会从STDIN接收数据，所以输入了两组数据后，输出了结果
- 终止gawk程序，可以使用组合键`Ctrl+D`来生成EOF（End-of-File）,

##### 使用数据字段变量
&#8195;&#8195;gawk的主要特性之一是其处理文本文件中数据的能力，它会自动给一行中的每个数据元素分配一个变量，默认情况下分配变量：
- $0代表整个文本行
- $1代表文本行中的第1个数据字段
- $2代表文本行中的第2个数据字段
- $n代表文本行中的第n个数据字段

&#8195;&#8195;在文本中，每个数据字段都是通过字段分隔符划分的，gawk中默认的字段分隔符是任意的空白字符（例如空格或制表符），示例如下：
```
[root@redhat8 sed_gawk]# gawk '{print $3 $4}' test1
CaptainAmerica!
CaptainAmerica!
[root@redhat8 sed_gawk]# gawk -F : '{print $1}' /etc/passwd
root
bin
...
[root@redhat8 sed_gawk]# echo "I am Thanos !"|gawk '{$3="Iron Man";print $0}'
I am Iron Man !
[root@redhat8 sed_gawk]# gawk '{
> $3="Iron Man"
> print $0}'
I am Thanos !
I am Iron Man !
```
说明：
- 示例1中gawk读取了文件，并根据需求打印出了$3和$4的值
- 示例2用`-F`指定了将冒号作为字符按分隔符
- 示例3示例了文本替换和使用多条命令，在命令之间加上分号即可
- 示例4中使用了多行输入，当使用了起始单引号，shell会使用次提示符来提示输入数据，直到输入了结尾的单引号
- 示例4中没有在命令行指定文件名，gawk会从SIDIN中获取数据
- 要退出程序，同样使用Ctrl+D组合键

##### 从文件中读取程序
示例如下：
```
[root@redhat8 sed_gawk]# cat test.gawk
{print $1 "'s home directory is " $6}
[root@redhat8 sed_gawk]# gawk -F : -f test.gawk /etc/passwd
root's home directory is /root
bin's home directory is /bin
daemon's home directory is /sbin
...
[root@redhat8 sed_gawk]# cat test1.gawk
{
text="'s home directory is "
print $1 text $6
}
```
说明：
- 示例中指定了用`-F`指定了将冒号作为字符按分隔符，并且用`-f`指定了文件
- 可以在程序文件中指定多条命令，一条命令放一行即可，不需要用分号
- 注意，gawk程序在引用变量时并未像shell脚本一样使用美元符号

##### 在处理数据前运行脚本
&#8195;&#8195;gawk允许指定程序脚本何时运行，默认情况下会从输入中读取一行文本，然后针对该文本进行数据的执行程序脚本，有时候需要在处理数据前提前运行脚本，比如为报告创建标题。BEGIN关键字就是用来实现创建标题，示例如下：
```
[root@redhat8 sed_gawk]# gawk 'BEGIN {print "I am Iron Man!"}'
I am Iron Man!
[root@redhat8 sed_gawk]# gawk 'BEGIN {print "The test fiie contents:"}
> {print $0}' test1
The test fiie contents:
I am Captain America!
I am Captain America!
```
##### 在处理数据后运行脚本
与BEGIN关键字类似，END关键字允许你指定一个程序脚本,在gawk读完数据后执行它，示例如下：
```
[root@redhat8 sed_gawk]# gawk 'BEGIN {print "The test fiie contents:"}
> {print $0}
> END {print "End of file"}' test1
The test fiie contents:
I am Captain America!
I am Captain America!
End of file
```
小程序脚本示例：
```
[root@redhat8 sed_gawk]# cat test2.gawk
BEGIN {
    print "The latest list of uers and shells"
    print " UserID \t  Shell"
    print "------- \t -------"
    FS=":"
}
{
    print $1 "   \t " $7
}
END{
    print "This concludes the listing"
}
[root@redhat8 sed_gawk]# gawk -f test2.gawk /etc/passwd
The latest list of uers and shells
 UserID 	  Shell
------- 	 -------
root   	 /bin/bash
bin   	 /sbin/nologin
daemon   	 /sbin/nologin
adm   	 /sbin/nologin
lp   	 /sbin/nologin
[...]
apache   	 /sbin/nologin
This concludes the listing
```
### sed编辑器基础
##### 更多的替换选项
替换标记示例如下：
```
[root@redhat8 sed_gawk]# cat test2
I am Captain America! I am Captain America!
I am Captain America! I am Captain America!
[root@redhat8 sed_gawk]# sed 's/Captain America/Iron Man/' test2
I am Iron Man! I am Captain America!
I am Iron Man! I am Captain America!
[root@redhat8 sed_gawk]# sed 's/Captain America/Iron Man/2' test2
I am Captain America! I am Iron Man!
I am Captain America! I am Iron Man!
[root@redhat8 sed_gawk]# sed 's/Captain America/Iron Man/g' test2
I am Iron Man! I am Iron Man!
I am Iron Man! I am Iron Man!
[root@redhat8 sed_gawk]# vi test3
[root@redhat8 sed_gawk]# sed 's/Captain America/Iron Man/gw test3' test2
I am Iron Man! I am Iron Man!
I am Iron Man! I am Iron Man!
[root@redhat8 sed_gawk]# cat test3
I am Iron Man! I am Iron Man!
I am Iron Man! I am Iron Man!
[root@redhat8 sed_gawk]# cat test4
This is a test line.
This is a different line.
[root@redhat8 sed_gawk]# sed -n 's/test/trial/p' test4
This is a trial line.
```
上面示例中使用了替换标记（substitution flag）格式：`s/pattern/relpaceement/flags`,有四种可用的替换标记：
- 数字，表明新文本将替换第几处模式匹配的地方
- g，表明新文本将会替换所有匹配的文本
- p，表明替换行的内容打印出来，通常和send的`-n`（禁止send编辑器输出）选项一起使用
- w file,将替换的结果写到文件中

字符替换示例：
```
[root@redhat8 sed_gawk]# sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd
root:x:0:0:root:/root:/bin/csh
[root@redhat8 sed_gawk]# sed 's!/bin/bash!/bin/csh!' /etc/passwd
root:x:0:0:root:/root:/bin/csh
```
&#8195;&#8195;上面的示例中，路径用的/和sed里面/冲突了，需要用反斜杠来转义，但是看起来似乎不方便，可用使用感叹号作为字符串分隔符，这样路径名就容易理解和阅读了。

##### 使用地址
&#8195;&#8195;默认情况下，sed编辑器中使用的命令会作用于文本数据的所有行，如果执行将命令作用于特定的行或某些行，必须使用寻址（line addressing），sed编辑器中有两种形式的行寻址：
- 以数字形式表示的行区间
- 用文本模式来过滤出行

标准格式：`[address]command`,或者将特定地址的多个命令分组：
```sh
address {
    command1
    command2
}
```
数字方式的行寻址示例如下：
```
[root@redhat8 sed_gawk]# sed '1s/Captain America/Batman/' test5
I am Batman!
I am Captain America!
I am Captain America!
[root@redhat8 sed_gawk]# sed '1,3s/Captain America/Batman/' test5
I am Batman!
I am Batman!
I am Batman!
[root@redhat8 sed_gawk]# sed '1,$s/Captain America/Batman/' test5
I am Batman!
I am Batman!
I am Batman!
```
说明：
- sed编辑器会将文本流中的第一行编号为1，按顺序进行编号
- 在命令中指定的地址可以是单个行号，或者`1,3`这种指定的一定区间范围内的行
- 可以使用美元符号，例如`1,$`表示从第一行开始的所有行

使用文本模式过滤器示例如下：
```
[root@redhat8 sed_gawk]# grep huang /etc/passwd
huang:x:1000:1000:huang:/home/huang:/bin/bash
[root@redhat8 sed_gawk]# sed '/huang/s/bash/ksh/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
[...]
huang:x:1000:1000:huang:/home/huang:/bin/ksh
```
说明：
- 指定文本模式来过滤出要作用的行格式：`/pattren/command`
- 示例中演示了只修改huang的默认shell

命令组合示例（使用地址区间一样）：
```
[root@redhat8 sed_gawk]# sed '2{
> s/am/am not/
> s/Captain America/Thanos/
> }' test5
I am Captain America!
I am not Thanos!
I am Captain America!
```
##### 删除行
示例如下:
```
[root@redhat8 sed_gawk]# cat test6
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
[root@redhat8 sed_gawk]# sed '2d' test6
1. I am Captain America!
3. I am Captain America!
[root@redhat8 sed_gawk]# sed '1,$d' test6
[root@redhat8 sed_gawk]# sed '/1. am/d' test6
2. I am Captain America!
3. I am Captain America!
[root@redhat8 sed_gawk]# sed 'd' test6
[root@redhat8 sed_gawk]# cat test6
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
```
说明：
- 删除命令`d`会删除匹配指定寻址模式的所有行
- 可以指定地址和`d`一起使用，可以指定特定区间，或者模式匹配特性也适用于删除命令
- sed编辑器不会修改原始文件，删除的行只是从sed编辑器的输出中删除

更多示例：
```
[root@redhat8 sed_gawk]# sed '/1/,/2/d' test6
3. I am Captain America!
[root@redhat8 sed_gawk]# cat test7
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
1. I am Iron man !
2. I am Thanos 
3. I am Captain America!
test line
[root@redhat8 sed_gawk]# sed '/1/,/2/d' test7
3. I am Captain America!
3. I am Captain America!
test line
[root@redhat8 sed_gawk]# sed '/1/,/4/d' test7
[root@redhat8 sed_gawk]# 
```
说明：
- 上面第一个示例中指定第一个模式会打开行删除功能，第二个模式会关闭行删除功能，编辑器会删除两个行之间的所有行，包括指定的行
- 第二个示例中，发现了第二个数字"1"，然后再次触发了删除命令
- 第三个示例中，没有匹配到结束模式，所有整个数据流都被删掉了

##### 插入和附加文本
sed编辑器允许向数据流插入和附加文本行：
- 插入（insert）命令（i）会在指定行前增加一个新行
- 附加（append）命令（a）会在指定行后增加一个新行

必须指定是要将行插入还是附加到另一行，标准格式如下：
```sh
sed '[address]command\
new line'
```
示例如下：
```
[root@redhat8 sed_gawk]# echo "I am Iron Man" |sed 'i\I am Thanos'
I am Thanos
I am Iron Man
[root@redhat8 sed_gawk]# echo "I am Iron Man" |sed 'a\I am Thanos'
I am Iron Man
I am Thanos
[root@redhat8 sed_gawk]# cat test1
I am Captain America!
I am Captain America!
[root@redhat8 sed_gawk]# sed '2i\
> I am Iron Man!' test1
I am Captain America!
I am Iron Man!
I am Captain America!
[root@redhat8 sed_gawk]# sed '2a\
> I am Iron Man!' test1
I am Captain America!
I am Captain America!
I am Iron Man!
[root@redhat8 sed_gawk]# sed '$a\
> I am Iron Man!\
> I am Batman!' test1
I am Captain America!
I am Captain America!
I am Iron Man!
I am Batman!
```
说明：
- 想要将新行附加到数据流的末尾，只要用代表数据最后一行的美元符号就可以了
- 要插入或附加多行文本，必须对要插入或附加的新文本中的每一行使用反斜线，直到最后一行

##### 修改行
修改（change）命令允许修改数据流中整行文本的内容，示例如下：
```
[root@redhat8 sed_gawk]# sed '2c\
> I will save the world!' test6
1. I am Captain America!
I will save the world!
3. I am Captain America!
[root@redhat8 sed_gawk]# cat test7
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
1. I am Iron man !
2. I am Thanos 
3. I am Captain America!
[root@redhat8 sed_gawk]# sed '/2. I am/c\
> I will save the world!' test7
1. I am Captain America!
I will save the world!
3. I am Captain America!
1. I am Iron man !
I will save the world!
3. I am Captain America!
[root@redhat8 sed_gawk]# sed '2,5c\
> I will save the world!' test7
1. I am Captain America!
I will save the world!
3. I am Captain America!
test line
```
说明：
- 可以使用数字或文本进行寻址，文本模式修改命令会修改它匹配的数据流中的任意文本
- 当使用地址区间时，会用这一行文本替换数据流中的两行文本，而不是逐一修改这两行文本

##### 转换命令
转换（transform）命令（y）是唯一可以处理单个字符的sed编辑器命令。格式如下
```
[address]y/inchars/outchars/
```
示例如下：
```
[root@redhat8 sed_gawk]# sed 'y/123/789/' test7
7. I am Captain America!
8. I am Captain America!
9. I am Captain America!
7. I am Iron man !
8. I am Thanos 
9. I am Captain America!
test line
[root@redhat8 sed_gawk]# echo "This 1 is a test of 1 try." |sed 'y/123/456/'
This 4 is a test of 4 try.
```
说明：
- 转换会对inchars和outchars值进行一对一的映射
- inchars中的第一个字符会被转换为outchars中的第一个字符，第二个字符会被转换成outchars中的第二个字符
- 映射过程会一直持续到处理完指定字符
- 如果inchars和outchars的长度不同，则sed编辑器会产生一条错误消息
- 转换命令是一个全局命令，会对文本中找到的所有指定字符自动进行转换

##### 回顾打印
之前介绍过使用p标记和替换命令显示sed编辑器修改过的行，另外还有三个命令也能用来打印数据流中的信息：
- p命令用来打印文本行
- 等号（=）命令用来打印行号
- 小写字母l（小写L）命令用来列出行

打印行示例：
```
[root@redhat8 sed_gawk]# echo "I am Iron Man!"|sed 'p'
I am Iron Man!
I am Iron Man!
[root@redhat8 sed_gawk]# echo "I am Iron Man!"|sed -n 'p'
I am Iron Man!
[root@redhat8 sed_gawk]# sed -n '/Iron man/p' test7
1. I am Iron man !
[root@redhat8 sed_gawk]# sed -n '2,4p' test7
2. I am Captain America!
3. I am Captain America!
1. I am Iron man !
[root@redhat8 sed_gawk]# sed -n '/Thanos/{
> p
> s/Thanos/Batman/p
> }' test7
2. I am Thanos 
2. I am Batman 
```
说明：
- 在命令行上用`-n`选项可以禁止输出其他行，只打印包含匹配文本模式的行
- 最后一个示例中首先找到包含Thanos的行，并且打印出来了，然后用s命令替换文本，并用p标记打印出替换结果，同时显示了原来的文本和新文本

打印行号示例：
```
[root@redhat8 sed_gawk]# sed '=' test1
1
I am Captain America!
2
I am Captain America!
[root@redhat8 sed_gawk]# sed -n '/Iron man/{
> =
> p
> }' test7
4
1. I am Iron man !
```
说明：
- 等号命令回去打印行在数据流中的当前行号，行号有数据流中的换行符决定
- 第二个示例是数据流中查找特定文本，并标记行号，利用`-n`选项让sed编辑器只显示了包含匹配文本模式的行的行号和文本

&#8195;&#8195;列出（list）命令（l）可以打印数据流中的文本和不可打印的ASCII字符。任何不可打印的字符要么在其八进制前加一个反斜杠，要么使用标准C风格的命名法，比如`\t`来代表制表符。列出行示例：
```
[root@redhat8 sed_gawk]# cat test8
I	am	Iron	Man !

[root@redhat8 sed_gawk]# sed -n 'l' test8
I\tam\tIron\tMan !$
$
```
说明：
- 制表符位置使用`\t`来显示，行尾的灭怨妇表示换行符
- 如果数据流包含了转义字符，列出命令会在毕业时候用八进制来显示

### 使用sed处理文件

##### 写入文件
命令w来向文件写入行，格式：`[address]w filename`。说明如下：
- filename可以使用相对路径或绝对路径，运行sed编辑器的用户必须有文件的写权限
- 地址可以是sed中支持的任意类型的寻址方式，例如单个行号、文本模式、行区间或文本模式

示例如下:
```
[root@redhat8 sed_gawk]# sed '1,2w test9' test7
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
1. I am Iron man !
2. I am Thanos 
3. I am Captain America!
test line
[root@redhat8 sed_gawk]# cat test9
1. I am Captain America!
2. I am Captain America!
[root@redhat8 sed_gawk]# sed -n '/Captain America/w test10' test7
[root@redhat8 sed_gawk]# cat test10
1. I am Captain America!
2. I am Captain America!
3. I am Captain America!
3. I am Captain America!
```
说明：
- 第一个示例中将数据流中的前两行打印到文本文件test9中
- 可以使用`-n`选项让行选项不显示到STDOUT上
- 第二个示例只将包含文本模式的数据行写入到目标文件，次方法非常有用

##### 从文件读取数据
&#8195;&#8195;读取（read）命令（r）允许将一个独立的文件中的数据插入到数据流中，格式：`[address]r filsname`。在读取命令中使用地址区间，只能指定单独的一个行号或文本模式地址，sed编辑器会将文件中的文本插入到指定的地址后。示例如下：
```
[root@redhat8 sed_gawk]# cat test11
I am Iron Man!
I will save the world !
[root@redhat8 sed_gawk]# cat test1
I am Captain America!
I am Captain America!
[root@redhat8 sed_gawk]# sed '1r test11' test1
I am Captain America!
I am Iron Man!
I will save the world !
I am Captain America!
[root@redhat8 sed_gawk]# sed '/Iron Man/r test1' test11
I am Iron Man!
I am Captain America!
I am Captain America!
I will save the world !
[root@redhat8 sed_gawk]# sed '$r test11' test1
I am Captain America!
I am Captain America!
I am Iron Man!
I will save the world !
```
可以和删除命令一起使用：利用一个文件中的数据来替换文件中的占位文本，例如有一份保持hero的文本文件,：
```
[root@redhat8 sed_gawk]# cat hero.std
Would the following hero:
LIST
They will save the world!
[root@redhat8 sed_gawk]# cat test12
Thor,T	Asgard
Hulk,H	Earth
```
现在就将hero.std中的LIST替换成hero清单，示例如下：
```
[root@redhat8 sed_gawk]# sed '/LIST/{
> r test12
> d
> }' hero.std
Would the following hero:
Thor,T	Asgard
Hulk,H	Earth
They will save the world!
```
