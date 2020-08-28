# Shell笔记-正则表达式
正则表达式是用户定义的模式模板（pattren template），Linux工具可以用它来过滤文本。

在Linux中，有两种流行的正则表达式引擎：
- POSIX基础正则表达式（basic regular expression，BRE)引擎
- POSIX扩展正则表达式（extended regular expression，ERE）引擎

## BRE模式
&#8195;&#8195;大多数Linux工具都至少符合POSIX BRE引擎规范，有些工具（sed编辑器）只符合了BRE引擎规范的子集。最基本的BRE模式是匹配数据流中的文本字符，本节内容也主要是学习BRE模式。
### 纯文本
回顾下之前学习的sed编辑器和gawk程序中用标准文本字符串来过滤数据：
```
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/Iron/p'
I am Iron Man !
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/iron/p'
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/Captain/p'
[root@redhat8 regular]# echo "I am Iron Man !" |gawk '/Iron/{print $0}'
I am Iron Man !
[root@redhat8 regular]# echo "I am Iron Man !" |gawk '/iron/{print $0}'
[root@redhat8 regular]# echo "I am Iron Man !" |gawk '/Captain/{print $0}'
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/Ir/p'
I am Iron Man !
[root@redhat8 regular]# echo "I am Iron Man !" |gawk '/Ir/{print $0}'
I am Iron Man !
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/I am/p'
I am Iron Man !
[root@redhat8 regular]# echo "I am Iron Man !" |sed -n '/Ir n/p'
[root@redhat8 regular]# cat test1
I am Iron Man!
I am  Iron Man!
[root@redhat8 regular]# sed -n '/  /p' test1
I am  Iron Man!
```
说明：
- 正则表达式不关心模式在数据流中的位置，也不关心模式出现了多少次
- 正则表达式模式都区分大小写，只会匹配大小写相符的模式
- 正则表达式中，不用写出整个单词，只要定义的文本出现在数据流中就能匹配
- 可以在正则表达式中使用空格和数字，空格和其它字符并没有什么区别
- 如果在正则表达式中定义了空格，那么它必须出现在数据流中，才能匹配到数据
- 最后示例单词间有两个空格的行匹配正则表达式模式是用来查看文本中空格问题的好办法

### 特殊字符
有些字符在正则表达式中有特别的含义，正则表达式识别的特殊字符包括：`.*[]^${}\+?|()`。示例如下：
```
[root@redhat8 regular]# cat test2
My current profit is $3,042!
[root@redhat8 regular]# sed -n '/\$/p' test2
My current profit is $3,042!
[root@redhat8 regular]# echo "\ is a special character"|sed -n '/\\/p'
\ is a special character
[root@redhat8 regular]# echo "99 / 9" |sed -n '/\//p'
99 / 9
```
说明：
- 如果要是有某个特殊字符作为文本字符，就必须转义，使用反斜线（\）
- 反斜线（\）也是特殊字符，在正则表达式中使用时候，也必须转义
- 尽管正斜线（/）不是正则表达式的特殊字符，但是使用时候也必须转义

### 锚字符
有两个特殊字符可以用来将模式锁定在数据流中的行首或行尾。
##### 锁定在首行
&#8195;&#8195;脱字符（^）定义从数据流中文本行的行首开始的模式，如果模式出现在行首之外的位置，正则表达式模式则无法匹配，示例如下：
```
[root@redhat8 regular]# echo "Miracles happen every day"|sed -n '/^Miracles/p'
Miracles happen every day
[root@redhat8 regular]# cat test3
Miracles happen every day!
To make each day count. 
Stupid is as stupid does. 
[root@redhat8 regular]# sed -n '/^To/p' test3
To make each day count.
```
说明：
- 要是有脱字符，必须将它放在正则表达式中指定的模式前面
- 脱字符会在每个有换行符决定的新数据行的首行检查模式
- 如果将脱字符放到模式开头之外的其它位置，那么它就和普通字符一样
- 如果指定正则表达式模式时只用了脱字符，就不需要用反斜线来转义

##### 锁定在行尾
特殊字符美元符（$）定义了行尾锚点，要想匹配，文本模式必须是行的最后一部分，示例如下：
```
[root@redhat8 regular]# echo "Life was like a box of chocolates"|sed -n '/chocolate$/p
[root@redhat8 regular]# echo "Life was like a box of chocolates"|sed -n '/chocolates$/p'
Life was like a box of chocolates
```
##### 组合锚点
示例如下：
```
[root@redhat8 regular]# cat test3
Miracles happen every day!
To make each day count. 
Stupid is as stupid does. 
[root@redhat8 regular]# sed -n '/^To make each day count.$/p' test3
To make each day count.
[root@redhat8 regular]# cat test4
Miracles happen every day!

Stupid is as stupid does. 
[root@redhat8 regular]# sed '/^$/d' test4
Miracles happen every day!
Stupid is as stupid does. 
```
说明：
- 第一个示例中sed编辑器过滤了那次不单单包含指定的文本的行
- 第二个示例将两个锚点组合在一起，直接没有任何文本，这样过滤了数据流中的空白行，这是删除文档中空白行的有效方法

### 点号字符
特殊字符点号用来匹配除换行符之外的任意单个字符，示例如下：
```
[root@redhat8 regular]# cat test5
This is a test of a line.
The cat is sleeping.
That is a very nice hat.
This test is at line four.
at ten o'clock we'll go home.
[root@redhat8 regular]# sed -n '/.at/p' test5
The cat is sleeping.
That is a very nice hat.
This test is at line four.
```
说明：
- 如果在点号的位置没用字符，那么模式就不成立，示例中第五行就是，点号放在行首就不会匹配该模式了
- 第四行匹配到了，虽然at前面看起来没有任何字符，实际上有空格，在正则表达式中，空格也是字符

### 字符组
&#8195;&#8195;如果想限定待匹配的具体字符，在正则表达式中，可以使用字符组（character class），可以定义用来匹配文本模式中的某个位置的一组字符，如果字符组中的某个字符出现来了数据流中，那它就匹配了该模式。使用方括号来定义一个字符组，示例如下：
```
[root@redhat8 regular]# sed -n '/[ch]at/p' test5
The cat is sleeping.
That is a very nice hat.
[root@redhat8 regular]# echo "Yes" |sed -n '/[Yy]es/p'
Yes
[root@redhat8 regular]# echo "yes" |sed -n '/[Yy]es/p'
yes
[root@redhat8 regular]# echo "YeS" |sed -n '/[Yy][Ee][Ss]/p'
YeS
[root@redhat8 regular]# sed -n '/[0123]/p' test6
This line has 2 number on it
This line number is 3
```
说明：
- 方括号中包含所有希望出现在该字符组中的字符，并且字符组中必须有个字符来匹配相应的位置
- 不太确定某个字符的大小写时，字符组会非常有用
- 可以在单个表达式中使用多个字符组
- 字符组中还可以使用数字，也可以组合在一起使用

&#8195;&#8195;字符和数字组合在一起使用进行匹配，比如电话号码和邮编，当尝试匹配某种特定的格式时，需要注意，下面示例第一次虽然匹配处理邮政编码，但是也通过了六位数的数字，可以指定行首和行尾进行匹配，示例如下：
```
[root@redhat8 regular]# cat test7
518031
5180
5180555
518
51866
[root@redhat8 regular]# sed -n '
> /[0123456789][0123456789][0123456789][0123456789][0123456789][0123456789]/p
> ' test7
518031
5180555
[root@redhat8 regular]# sed -n '
> /^[0123456789][0123456789][0123456789][0123456789][0123456789][0123456789]$/p
> ' test7
518031
```
字符组一个常见的用法是解析拼错的单词，当然正确拼写的也能匹配出来，示例如下：
```
[root@redhat8 regular]# cat test8
Miracles happen every dey!
To make each day count.
Stupid is as stopid does. 
[root@redhat8 regular]# sed -n '
> /d[ea]y/p
> /st[ou]pid/p
> ' test8
Miracles happen every dey!
To make each day count.
Stupid is as stopid does.
```
### 排除型字符组
&#8195;&#8195;可以寻找组总没有的字符，而不是去匹配组总含有的字符，在字符组的开头加一个脱字符即可，示例如下：
```
[root@redhat8 regular]# sed -n '/[^ch]at/p' test5
This test is at line four.
```
说明：
- 示例中匹配除c或h之外的所有字符，包括空格，所以匹配出来了
- 即使是排除，字符组也必须匹配一个字符，示例中匹配的是空格，所以在test5中第五行依旧没有匹配出来

### 区间
之前邮编匹配例子中每个数字都输入了，比较麻烦，可以使用区间来简化，示例如下：
```
[root@redhat8 regular]# sed -n '
> /^[0-9][0-9][0-9][0-9][0-9][0-9]$/p
> ' test7
518031
[root@redhat8 regular]# echo "x86"|sed -n '/^[0-9][0-9]$/p'
[root@redhat8 regular]# echo "x86"|sed -n '/[0-9][0-9]/p'
x86
[root@redhat8 regular]# echo "x86"|sed -n '/[0-9][0-9]$/p'
x86
[root@redhat8 regular]# sed -n '/[a-i]at/p' test5
The cat is sleeping.
That is a very nice hat.
[root@redhat8 regular]# sed -n '/[a-dg-i]at/p' test5
The cat is sleeping.
That is a very nice hat.
```
说明：
- 使用区间可以减少很多键盘的输入
- 同样方法也适用于字母，也可以都在单个字符组指定多个不连续的字母区间

### 特殊字符组
BRE特殊字符组：

组|描述
:---|:---
\[\[:alpha:]]|匹配任意字母字符，不管是大写还是小写
\[\[:alnum:]]|匹配任意字母数字字符0~9、a~z或A~Z
\[\[:blank:]]|匹配空格或制表符
\[\[:digit:]]|匹配0~9之间的数字
\[\[:lower:]]|匹配小写字母字符a~z
\[\[:print:]]|匹配任意可打印的字符
\[\[:punct:]]|匹配标点符号
\[\[:space:]]|匹配任意空白字符：空格、制表符、NL、FF、VT和CR
\[\[:upper:]]|匹配任意大写字母字符A~Z

可以在正则表达式模式中将特殊字符组像普通字符组一样使用：
```
[root@redhat8 regular]# echo "Hope can set you free"|sed -n '/[[:digit:]]/p'
[root@redhat8 regular]# echo "Hope can set you free"|sed -n '/[[:alpha:]]/p'
Hope can set you free
[root@redhat8 regular]# echo "x86"|sed -n '/[[:digit:]]/p'
x86
[root@redhat8 regular]# echo "Hope can set you free"|sed -n '/[[:punct:]]/p'
[root@redhat8 regular]# echo "Hope can set you free !"|sed -n '/[[:punct:]]/p'
Hope can set you free !
```
### 星号
在字符后面放置星号表明该字符必须在匹配模式的文本中出现0次或多次：
```
[root@redhat8 regular]# echo "lt" | sed -n '/le*t/p'
lt
[root@redhat8 regular]# echo "let" | sed -n '/le*t/p'
let
[root@redhat8 regular]# echo "leet" | sed -n '/le*t/p'
leet
```
经常用于处理常见拼写错误或不同语言中拼写不同的情况，例如color和colour：
```
[root@redhat8 regular]# echo "IBM color is bule!"|sed -n '/colou*r/p'
IBM color is bule!
[root@redhat8 regular]# echo "IBM colour is bule!"|sed -n '/colou*r/p'
IBM colour is bule!
```
&#8195;&#8195;可以将点号特殊字符和星号特殊字符组合使用，这个组合能够匹配任意数量的任意字符，通常用在数据流中两个可能相邻或不相邻的文本字符串之间：
```
[root@redhat8 regular]# echo "All life is a game of luck." | sed -n '
> /of.*luck/p'
All life is a game of luck.
```
星号还能用在字符组上，它允许指定可能在文本中出现多次的字符组或字符区间。
```
[root@redhat8 regular]# echo "lt" | sed -n '/l[ea]*t/p'
lt
[root@redhat8 regular]# echo "lat" | sed -n '/l[ea]*t/p'
lat
[root@redhat8 regular]# echo "let" | sed -n '/l[ea]*t/p'
let
[root@redhat8 regular]# echo "leet" | sed -n '/l[ea]*t/p'
leet
[root@redhat8 regular]# echo "laaeet" | sed -n '/l[ea]*t/p'
laaeet
```
## 扩展正则表达式
&#8195;&#8195;POSIX ERE模式包括了一些可供Linux应用和工具使用的额外符号，gawk程序能够识别ERE模式，但sed编辑器不能。本节内容主要是学习gawk程序中的较常见的ERE模式符号。
### 问号
&#8195;&#8195;问号类似星号，问号表明前面的字符可以出现0次或1次，但是仅限于此，它不会匹配多次出现的字符。示例如下：
```
[root@redhat8 regular]# echo "lt" |gawk '/le?t/{print $0}'
lt
[root@redhat8 regular]# echo "let" |gawk '/le?t/{print $0}'
let
[root@redhat8 regular]# echo "leet" |gawk '/le?t/{print $0}'
leet
[root@redhat8 regular]# echo "lt" |gawk '/l[ae]?t/{print $0}'
lt
[root@redhat8 regular]# echo "let" |gawk '/l[ae]?t/{print $0}'
let
[root@redhat8 regular]# echo "laet" |gawk '/l[ae]?t/{print $0}'
[root@redhat8 regular]# echo "leet" |gawk '/l[ae]?t/{print $0}'
```
### 加号
&#8195;&#8195;加号类似星号的另一个模式符号，加号表明前面的字符可以出现1次或多次，必须至少出现一次。示例如下：
```
[root@redhat8 regular]# echo "lt" |gawk '/le+t/{print $0}'
[root@redhat8 regular]# echo "let" |gawk '/le+t/{print $0}'
let
[root@redhat8 regular]# echo "leet" |gawk '/le+t/{print $0}'
leet
[root@redhat8 regular]# echo "lt" |gawk '/l[ae]+t/{print $0}'
[root@redhat8 regular]# echo "let" |gawk '/l[ae]+t/{print $0}'
let
[root@redhat8 regular]# echo "lat" |gawk '/l[ae]+t/{print $0}'
lat
[root@redhat8 regular]# echo "leat" |gawk '/l[ae]+t/{print $0}'
leat
[root@redhat8 regular]# echo "leet" |gawk '/l[ae]+t/{print $0}'
leet
[root@redhat8 regular]# echo "laaeet" |gawk '/l[ae]+t/{print $0}'
laaeet
```
### 使用花括号
&#8195;&#8195;ERE中的花括号允许为可重复的正则表达式指定一个上限，通常称为间隔（interval），这个特性可以精确调整字符或字符集在模式中具体出现的次数，可以有两种格式来指定区间：
- m：正则表达式准确出现m次
- m,n：正则表达式至少出现m次，至多n次

示例如下：
```
[root@redhat8 regular]# echo "lt"|gawk --re-interval '/le{1}t/{print $0}'
[root@redhat8 regular]# echo "let"|gawk --re-interval '/le{1}t/{print $0}'
let
[root@redhat8 regular]# echo "leet"|gawk --re-interval '/le{1}t/{print $0}'
[root@redhat8 regular]# echo "leet"|gawk --re-interval '/le{2}t/{print $0}'
leet
[root@redhat8 regular]# echo "lt"|gawk --re-interval '/le{1,2}t/{print $0}'
[root@redhat8 regular]# echo "let"|gawk --re-interval '/le{1,2}t/{print $0}'
let
[root@redhat8 regular]# echo "leet"|gawk --re-interval '/le{1,2}t/{print $0}'
leet
[root@redhat8 regular]# echo "leeet"|gawk --re-interval '/le{1,2}t/{print $0}'
[root@redhat8 regular]# echo "lt"|gawk --re-interval '/l[ae]{1,2}t/{print $0}'
[root@redhat8 regular]# echo "let"|gawk --re-interval '/l[ae]{1,2}t/{print $0}'
let
[root@redhat8 regular]# echo "leet"|gawk --re-interval '/l[ae]{1,2}t/{print $0}'
leet
[root@redhat8 regular]# echo "laet"|gawk --re-interval '/l[ae]{1,2}t/{print $0}'
laet
[root@redhat8 regular]# echo "laaeet"|gawk --re-interval '/l[ae]{1,2}t/{print $0}'
```
### 管道符号
&#8195;&#8195;管道符号允许在检查数据流时，用逻辑OR方式指定正则表达式引擎要用的两个或多个模式，如果任何一个模式匹配了数据流文本，文本就通过测试，格式：`expr1|expr2`,示例如下：
```
[root@redhat8 regular]# echo "Hope can set you free" |gawk '/can|yes/{print $0}'
Hope can set you free
[root@redhat8 regular]# echo "Hope can set you free" |gawk '/[ch]an|yes/{print $0}'
Hope can set you free
```
注意：正则表达式和管道符号之间不能有空格，否则会当作正则表达式的一部分。

### 表达式分组
&#8195;&#8195;正则表达式模式也可以用圆括号进行分组，当使用正则表达式模式分组时，改组会被视为一个标准字符，示例如下：
```
[root@redhat8 regular]# echo "Sat"|gawk '/Sat(urday)?/{print $0}'
Sat
[root@redhat8 regular]# echo "Satutday"|gawk '/Sat(urday)?/{print $0}'
Satutday
[root@redhat8 regular]# echo "cat"|gawk '/(c|b)a(b|t)/{print $0}'
cat
[root@redhat8 regular]# echo "cab"|gawk '/(c|b)a(b|t)/{print $0}'
cab
[root@redhat8 regular]# echo "bat"|gawk '/(c|b)a(b|t)/{print $0}'
bat
[root@redhat8 regular]# echo "bab"|gawk '/(c|b)a(b|t)/{print $0}'
bab
[root@redhat8 regular]# echo "tab"|gawk '/(c|b)a(b|t)/{print $0}'
```
说明：
- 可以像普通字符一样给该组使用特殊字符
- 示例中urday分组以及问号，使模式可以匹配完整的Saturday或缩写Sat
- 模式(c|b)a(b|t)会匹配第一组中字母的任意组合以及第二组中字母的任意组合

## 正则表达式实战
为方便查阅，收录在以下章节，：[Shell-正则表达式实例](https://bond-huang.github.io/huang/09-Shell%E8%84%9A%E6%9C%AC/02-Shell%E8%84%9A%E6%9C%AC%E5%BF%AB%E9%80%9F%E6%8C%87%E5%8D%97/04-Shell-%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%AE%9E%E4%BE%8B.html)
