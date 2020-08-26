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
