# Shell笔记-正则表达式
正则表达式是用户定义的模式模板（pattren template），Linux工具可以用它来过滤文本。
### 正则表达式类型
在Linux中，有两种流行的正则表达式引擎：
- POSIX基础正则表达式（basic regular expression，BRE)引擎
- POSIX扩展正则表达式（extended regular expression，ERE）引擎

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
