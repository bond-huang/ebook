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
