# Shell-sed&gawk实例
收录一些实用脚本示例。
## sed实例
### 加倍行间距
在文本的行间插入空白行简单sed脚本:
```
[root@redhat8 sed]# sed 'G' test2
Miracles happen every day !

To make each day count !

Stupid is as stupid does !

[root@redhat8 sed]# sed '$!G' test2
Miracles happen every day !

To make each day count !

Stupid is as stupid does !
```
说明：
- G命令会将保持空间内容附加到模式空间内容后，当启动sed时候，保持空间只有一个空行
- 第一次示例在最后一行也加入了一个空白行，可以使用排除符号（!）和尾行符号（$）来确定不将空白行加入到数据流最后一行后面
- 第二个示例中，当sed编辑器到了最后一行时，就会跳过G命令

### 含有空白行的文件加倍行间距
使用上面的方法，已有的空白行也会倍加倍，示例如下：
```
[root@redhat8 sed]# cat test3
All life is a game of luck!
Everything you see exists

together in a delicate balance !
[root@redhat8 sed]# sed '$!G' test3
All life is a game of luck!

Everything you see exists



together in a delicate balance !
```
解决办法就是先删除空白行后再添加，示例：
```
[root@redhat8 sed]# sed '/^$/d ; $!G' test3
All life is a game of luck!

Everything you see exists

together in a delicate balance !
```
说明：
- 首先删除数据流中的所有空白行，然后用G命令在所有行后插入新的空白行
- 删除已有的空白行，需要将d命令和一个匹配空白行的模式一起使用：`/^$/d`

### 给文件中的行编号
之前学过用等号来显示数据流中的行号：
```
[root@redhat8 sed]# sed '=' test2
1
Miracles happen every day !
2
To make each day count !
3
Stupid is as stupid does !
```
解决方法就是通过N命令放置到同一行：
```
[root@redhat8 sed]# sed '=' test2 | sed 'N; s/\n/./'
1.Miracles happen every day !
2.To make each day count !
3.Stupid is as stupid does !
```
说明：
- 在获得等号的输出后，通过管道将输出传给另一个sed编辑器，使用N命令来合并这两行
- 示例中，用替换命令将换行符更换成了点号，根据个人喜好更换任意字符
- 在查看错误消息的行号时候，这个工具很好用

bash shell命令也有添加行号的功能，示例如下：
```
[root@redhat8 sed]# nl test2
     1	Miracles happen every day !
     2	To make each day count !
     3	Stupid is as stupid does !
[root@redhat8 sed]# cat -n test2
     1	Miracles happen every day !
     2	To make each day count !
     3	Stupid is as stupid does !
```
### 打印末尾行
使用p命令打印数据流末尾行：
```
[root@redhat8 sed]# sed -n '$p' test2
Stupid is as stupid does !
```
如果只需要处理一个长输出（比如日志文件）中的末尾几行，可以使用创建滚动窗口方法，示例如下：
```
[root@redhat8 sed]# cat test5
1.Miracles happen every day !
2.To make each day count !
3.Stupid is as stupid does !
4.All life is a game of luck!
[root@redhat8 sed]# sed '{
> :start
> $q ; N ; 3,$D
> b start
> }' test5
3.Stupid is as stupid does !
4.All life is a game of luck!
```
说明：
- 滚动窗口是检验模式空间中文本行块的常用方法，使用N命令将这些块合并起来
- N命令将下一行文本附加到模式空间中已有的文本后面，示例中模式空间有了2行文本，用美元符号来检查是否处于数据流尾部，如果不在，继续向模式空间增加行，同时删除原来的行（D命令会删除模式空间的第一行）
- 通过循环N命令和D命令，向模式空间的文本增加新行的同时也删除了旧行，
- 使循环借宿，识别最后一行文本并用q命令退出即可
- 示例中先检查这行是不是数据流中的最后一行，如果是，退出命令q会停止循环
- N命令会将下一行附加到模式空间中当前行之后，如果当前行在第2行后面，3,$D命令会删除模式空间中的第一行
- 最后，示例脚本显示了文本的最后2行

### 删除行
删除数据流中不需要的空白行。
#### 删除连续的空白行
删除连续空白行的方法是用地址区间来检查数据流，示例如下：
```
[root@redhat8 sed]# cat test7
1.Miracles happen every day !


2.To make each day count !

3.Stupid is as stupid does !



4.All life is a game of luck!
[root@redhat8 sed]# sed '/./,/^$/!d' test7
1.Miracles happen every day !

2.To make each day count !

3.Stupid is as stupid does !

4.All life is a game of luck!
```
说明：
- 删除连续空白行的关键在于创建包含一个非空白行和一个空白行的地址区间
- 如果sed编辑器遇到了这个地址区间，它不会删除行，对于不匹配这个区间的行（两行或更多空白行），就会删除掉
- 操作脚本：`/./,/^$/!d`，其中区间是`/./`到`/^$/`，区间开始地址会匹配任何含有一个字符的行，区间结束地址会匹配一个空白行，这区间内的行不会被删除
- 无论文本的数据行之间出现多个空白行，输出结果只会在行间保留一个空白行

#### 删除开头的空白行
一样使用地址空间来解决那些行要删掉，示例如下：
```
[root@redhat8 sed]# cat test8

To make each day count !

Stupid is as stupid does !
[root@redhat8 sed]# sed '/./,$!d' test8
To make each day count !

Stupid is as stupid does !
```
说明：
- 使用的地址区间从含有字符的行开始，一直到数据流的结束
- 在此区间的任何行都不会从输出中删除，所有第一行被删除了
- 不管数据会之前有多少行，都会被删除，而数据行中的空白行会保留

#### 删除结尾的空白行
利用循环来实现，示例如下：
```
[root@redhat8 sed]# cat test9
To make each day count !
Stupid is as stupid does !


[root@redhat8 sed]# sed '{
> :start
> /^\n*$/{$d ; N ; b start }
> }' test9
To make each day count !
Stupid is as stupid does !
```
说明：
- 在正常脚本的花括号里还有花括号，这允许在整个命令脚本中将一些命令分组，该命令组会被应用在指定的地址模式上
- 地址模式能够匹配只含有一个换行符的行，如果匹配到了，而且是最后一行，删除命令就会删除它
- 如果不是最后一行，N命令会将下一行附加到它后面，分支命令会跳转到循环起始位置重新开始

### 删除HTML标签
&#8195;&#8195;从网站下载文本时，有时其中也包含了用户数据格式化的HTML标签，如果只是查看数据，标签会显得很不方便，例如下面的HTML文件示例：
```
[root@redhat8 sed]# cat test10
<html>
<head>
<title>This is the page title</title>
</head>
<body>
<p>This is the <b>first</b> line in the web page.
This should provide some <i>useful</i>
information to use in our sed script.
</body>
</html>
```
说明：
- HTM了标签由小于号和大于号来识别
- 大多数都是成对出现的：一个起始标签（`<b>`用来加粗），一个结束标签（`</b>`用来结束加粗）

如果直接查找并替换小于号和大于号中间的文本字符串，结果出乎意料：
```
[root@redhat8 sed]# sed 's/<.*>//g' test10





 line in the web page.
This should provide some 
information to use in our sed script.


```
解决方法就是让sed编辑器忽略掉任何嵌入到原始标签中的大于号：
```
[root@redhat8 sed]# sed 's/<[^>]*>//g' test10


This is the page title


This is the first line in the web page.
This should provide some useful
information to use in our sed script.


[root@redhat8 sed]# sed 's/<[^>]*>//g ; /^$/d' test10
This is the page title
This is the first line in the web page.
This should provide some useful
information to use in our sed script.
```
说明：
- 示例中中创建一个字符组来排除大于号：`s/<[^>]*>//g`
- 示例中可以看到需要的数据，但是还是有很多空格，加一条删除命令来删除多余空白行即可

## gawk实例
&#8195;&#8195;gawk可以用来处理出数据文件中的数据值，例如表格化销售数据或计算保龄球得分等。处理数据文件的关键是把相关的数据放在一起，然后对相关数据进行必要的计算。例如有个数据文件，包含了两只队伍（每队两名选手）的保龄球比赛的分情况：
```
Captain America,MCU,125,113,119
Iron Man,MCU,117,109,127
Batman,DC,120,115,114
Wonder Woman,DC,110,129,121
```
下面脚本对每队的成绩进行了排序，并计算了总分和平均分：
```sh
#!/bin/bash
for team in $(gawk -F, '{print $2}' scores.txt |uniq)
do
     gawk -v team=$team 'BEGIN{FS=",";total=0}
     {
          if ($2==team)
          {
               total += $3 + $4 + $5;
          }
     }
     END {
          avg = total / 6
          print "Total for",team,"is",total,",the average is",avg
     }' scores.txt
done
```
说明：
- for循环中的第一条语句过滤出数据文件中的队名，然后使用uniq命令去重，for循环再对每个队进行迭代
- for循环的内部gawk语句进行计算操作：
     - 对每一条记录，首先确定队名是否和正在循环的队名相符
     - 通过利用gawk的`-v`选项来实现，该选项允许在gawk程序中传递shell变量
     - 如果队名相符，代码会对数据中的三场比赛得分求和，然后将每条记录的值再相加，只要数据记录属于同一队
- 最后gawk代码会显示出总分以及平均分

运行后输出结果如下：
```
[root@redhat8 gawk]# sh bowling.sh
Total for MCU is 710 ,the average is 118.333
Total for DC is 709 ,the average is 118.167
```
