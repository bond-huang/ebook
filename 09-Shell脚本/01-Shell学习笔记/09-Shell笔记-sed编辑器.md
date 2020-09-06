# Shell笔记-sed编辑器
&#8195;&#8195;之前学习过一些sed编辑器的基础命令，能够满足大多数日常文本编辑的需求，本节学习sed编辑器提供的一些高级特性。
### 多行命令
&#8195;&#8195;之前学习的sed编辑器是定义好的脚本命令一次处理一行数据，处理完成后再到下一行重复这个过程。有时候需要跨多行的数据执行操作，sed编辑器包含了三个可用来处理多行文本的特殊命令：
- N:将数据流中的下一行加进来创建一个多行组（multiline group）来处理
- D:删除多行组中的一行
- P:打印多行组中的一行

#### next命令
##### 单行next命令
&#8195;&#8195;小写的n命令会告诉sed编辑器移动到数据流中的下一行文本，而不用重新回到命令的最开始再执行一遍。示例如下：
```
[root@redhat8 sed]# cat test1
Miracles happen every day !

To make each day count !

Stupid is as stupid does !
[root@redhat8 sed]# sed '/^$/d' test1
Miracles happen every day !
To make each day count !
Stupid is as stupid does !
[root@redhat8 sed]# sed '/happen/{n;d}' test1
Miracles happen every day !
To make each day count !

Stupid is as stupid does !
```
说明：
- 示例中一共五行，两行是空的，目标是删除首行之后的空白行，而留下另一个空白行，使用删除空白行的sed脚本会删除两个空白行
- 可以使用n命令解决，示例中，脚本会查找含有单词happen的行，找到后n命令会让sed编辑器移动到文本的下一行，也就是空白行，然后继续执行命令列表，下一个就是d命令，也就是删除
- sed编辑器执行完脚本后，会从数据流中读取下一行文本，并从头开始执行脚本，但是找不到含有单词happen的行了

##### 合并文本行
&#8195;&#8195;单行next命令ui将数据流中的下一个文本行移动到sed编辑器的工作空间（称为模式空间）；多行版的next命令（用大写N）会将下一个文本添加到模式空间中已有的文本后，示例如下：
```
[root@redhat8 sed]# cat test2
Miracles happen every day !
To make each day count !
Stupid is as stupid does !
[root@redhat8 sed]# sed '/count/{N;s/\n/ /}' test2
Miracles happen every day !
To make each day count ! Stupid is as stupid does !
```
说明：
- 示例中将数据流中的两个文本行合并到同一个模式空间中，虽然又换行符，但是sed编辑器会当作一行来处理
- sed编辑器找到含有count的那行文本，然后用N命令将下一行合并到此行，然后用替换命令s将换行符替换成空格

在数据文件中查找一个可能分散在两行中的文本短语示例：
```
[root@redhat8 sed]# cat test4
Today,the RHEL System
Administrator's group meeting will be held.
All System Administrators should attend.
Thank you for your attenddance.
[root@redhat8 sed]# sed 'N;s/System Administrator/Desktop User/' test4
Today,the RHEL System
Administrator's group meeting will be held.
All Desktop Users should attend.
Thank you for your attenddance.
[root@redhat8 sed]# sed 'N;s/System.Administrator/Desktop User/' test4
Today,the RHEL Desktop User's group meeting will be held.
All System Administrators should attend.
Thank you for your attenddance.
```
说明：
- 示例中用N命令将发现的第一个单词的那行和下一行进行合并处理，即使出现了换行符，同样可以找到
- 替换命令在两个单词之间使用了通配符模式（.）来匹配空格和换行符

&#8195;&#8195;上面示例中将两行合并成一行，有时候也行并不需要，可以使用两个替换命令，一个用来匹配出现在多行的情况，一个用来匹配出现在单行情况，示例如下：
```
[root@redhat8 sed]# sed 'N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' test4
Today,the RHEL Desktop
User's group meeting will be held.
All Desktop Users  should attend.
Thank you for your attenddance.
```
&#8195;&#8195;脚本也有问题，当到最后一行时候，没有下一行了，N命令会让sed编辑器停止，如果要匹配的文本正好在最后一行，就不会匹配到数据了，解决方法就是将单行命令放到N命令前面，并将多行命令放到N命令后面，示例如下：
```
[root@redhat8 sed]# cat test4
Today,the RHEL System
Administrator's group meeting will be held.
All System Administrators should attend.
[root@redhat8 sed]# sed 'N;s/System Administrator/Desktop User/' test4
Today,the RHEL System
Administrator's group meeting will be held.
All System Administrators should attend.
[root@redhat8 sed]# sed 'N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' test4
Today,the RHEL Desktop
User's group meeting will be held.
All System Administrators should attend
[root@redhat8 sed]# sed '
> s/System Administrator/Desktop User/
> N
> s/System\nAdministrator/Desktop\nUser/
> ' test4
Today,the RHEL Desktop
User's group meeting will be held.
All Desktop Users  should attend.
```
#### 多行删除命令
&#8195;&#8195;sed编辑器提供了多行删除命令D，它只删除模式空间中的第一行，会删除到换行符（含换行符）为止的所有字符，示例如下：
```
[root@redhat8 sed]# sed 'N;/System\nAdministrator/D' test4
Administrator's group meeting will be held.
All System Administrators  should attend.
Thank you for your attenddance.
[root@redhat8 sed]# cat test3

All life is a game of luck!
Everything you see exists

together in a delicate balance !
[root@redhat8 sed]# sed '/^$/{N;/luck/D}' test3
All life is a game of luck!
Everything you see exists

together in a delicate balance !
```
说明：
- 第一个示例中，文本的第二行被N命令加到了模式空间，但仍然完好，此方法可以用来删除目标数据字符串所在行的前一行文本
- 第二个示例中，删除了数据流中出现的第一行前的空白行；先查找空白行，然后N命令将下一个文本行添加到模式空间，新的模式空间中含有luck单词，D命令会删除模式空间中的第一行
- 如果不结合使用N命令和D命令，就不可能在不删除其它空白行的情况下只删除一个空白行

#### 多行打印命令
&#8195;&#8195;多行打印命令（P）沿用了和之前同样的方法，它只打印多行模式空间中的第一行，直到换行符为止的所有字符。当用-n选项来阻止脚本输出时，和单行p命令用法大同小异，示例如下：
```
[root@redhat8 sed]# sed -n 'N;/System\nAdministrator/P' test4
Today,the RHEL System
```
### 保持空间
&#8195;&#8195;模式空间（pattern space）是一块活跃的缓冲区，在sed编辑器执行命令时它会保存待检查的文本。但它并不是sed编辑器保存文本的唯一空间。
&#8195;&#8195;sed编辑器又另一块称作保持空间（hold space）的缓冲区域，可以用来临时保存一些行，有5条命令可用来操作保持空间，如下表所示：

命令|描述
:---:|:---:
h|将模式空间复制到保持空间
H|将模式空间附加到保持空间
g|将保持空间复制到模式空间
G|将保持空间附加到模式空间
x|交换模式空间和保持空间的内容

示例如下：
```
[root@redhat8 sed]# sed -n '/1./{h;p;n;p;g;p}' test5
1.Miracles happen every day !
2.To make each day count !
1.Miracles happen every day !
```
说明：
- sed编辑器在地址中用正则表达式来过滤出含有`1.`的行
- 当含有`1.`的行出现时，h命令将改行放到保持空间
- p命令打印模式空间也就是第一个数据行的内容
- n命令提取数据流中的下一行（2.开头那行），并将它放到模式空间
- p命令打印模式空间的内容，也就是第二个数据行
- g命令将保持空间的内容（1.开头那行）放回到模式空间，替换当前文本
- p命令打印模式空间的内容，也就是第一个数据行

少一个p就会改变输出，此方法可以用来将整个文件的文本进行反转，示例如下：
```
[root@redhat8 sed]# sed -n '/1./{h;n;p;g;p}' test5
2.To make each day count !
1.Miracles happen every day !
```
### 排除命令
&#8195;&#8195;感叹号命令（!）用来排除（negate）命令，也就是让原本会起作用的命令不起作用，示例如下：
```
[root@redhat8 sed]# sed -n '/happen/!p' test5
2.To make each day count !
3.Stupid is as stupid does !
4.All life is a game of luck!
```
&#8195;&#8195;正常p会答应匹配的的那一行，示例中恰号相反，其它的行都打印了。在之前有一种情况是sed编辑器无法处理数据流中的最后一行文本，因为只会再也没有其他行了，可以用感叹号来解决这个问题。
```
[root@redhat8 sed]# sed 'N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' test4
Today,the RHEL Desktop
User's group meeting will be held.
All System Administrators should attend
[root@redhat8 sed]# sed '$!N;
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' test4
Today,the RHEL Desktop
User's group meeting will be held.
All Desktop Users  should attend.
```
说明：
&#8195;&#8195;美元符号表示数据流中的最后一行文本，当sed编辑器到了最后一行时，它没有执行N命令，但它对所有其他行都执行了这个命令。

可以用来反转数据流中文本行的顺序，可以参照如下方法使用模式空间：
- 在模式空间中放置一行
- 将模式空间中的行放到保持空间中
- 在模式空间中放入下一行
- 将保持空间附加到模式空间后
- 将模式空间中的所有内容都放到保持空间中
- 重复执行第3-5步，直到所有行都反序放到了保持空间中
- 提取并打印行

方法说明：
- 不想在处理过程中打印行，要使用sed的-n命令行选项
- 下一步决定如何将保持空间文本附加到模式空间文本后面，可以使用G命令完成
- 不想将保持空间附加到要处理的第一行文本后面，可以使用感叹号解决：`1!G`
- 下一步是将新的模式空间（含已反转的行）放到保持空间，用h命令就行
- 将模式空间中的整个数据流都反转后，就是打印结果，打印需要用`$p`

示例如下：
```
[root@redhat8 sed]# cat test2
Miracles happen every day !
To make each day count !
Stupid is as stupid does !
[root@redhat8 sed]# sed -n '{1!G ; h ; $p }' test2
Stupid is as stupid does !
To make each day count !
Miracles happen every day !
```
在Linux中，tac命令会倒序显示一个文本文件，cat命令拼写相反，示例如下：
```
[root@redhat8 sed]# tac test2
Stupid is as stupid does !
To make each day count !
Miracles happen every day !
```
### 改变流
&#8195;&#8195;一般sed编辑器会从脚本的顶部开始，一直执行号脚本的结尾（D命令除外，它会强制sed返回到脚本的顶部）。sed编辑器提供了一个方法来改变命令脚本的执行流程，其结果与结构化编程类型。
#### 分支
&#8195;&#8195;sed编辑器可以基于地址、地址模式或地址区间排除一整块命令，允许只对数据流中的特定行执行一组命令，分支（branch）命令b的格式：`[address]b [label]`。说明：
- address参数决定了哪些行的数据会触发分支命令
- label参数定义了要跳转到的位置
- 如果没有label参数，跳转命令会跳转到脚本的结尾

示例如下：
```
[root@redhat8 sed]# cat test6
This is the header line!
This is the first data line!
This is the second data line!
This is the last line!
[root@redhat8 sed]# sed '{2,3b;s/This is/Is this/;s/line!/test?/}' test6
Is this the header test?
This is the first data line!
This is the second data line!
Is this the last test?
```
说明：
- 分支命令在数据流中的第2行和第3行处跳过了两个替换命令
- 不希望跳到脚本的几位，可以为分支命令定义一个标签，最多七个字符长度，格式：`:label12`
- 需要指定标签时候将其加到b命令后即可
- 使用标签允许跳过地址匹配出的命令，但仍然会执行脚本中的其它命令

示例如下：
```
[root@redhat8 sed]# sed '{/first/b jump1;s/This is the/No jump on/
> :jump1
> s/This is the/Jump here on/}' test6
No jump on header line!
Jump here on first data line!
No jump on second data line!
No jump on last line!
```
说明：
- 跳转命令指定如果文本中出现了first，程序应该跳到标签为jump1的脚本行
- 如果分支命令的模式没有匹配，sed会继续执行脚本中的命令，包括分支标签后的命令
- 所有替换命令都会在不匹配分支模式的行上执行
- 如果某行匹配了分支模式，sed会跳转到带有分支标签的那行，所有只有最后一个替换命令会执行

&#8195;&#8195;上面示例中是跳转到sed脚本后面的标签上，可以跳转到脚本中靠前的标签上，这样可以达到循环效果，示例如下：
```
[root@redhat8 sed]# echo "A, test, to, remove, commas."|sed -n '{
> :start
> s/,//1p
> b start
> }'
A test, to, remove, commas.
A test to, remove, commas.
A test to remove, commas.
A test to remove commas.
^C
[root@redhat8 sed]# echo "A, test, to, remove, commas."|sed -n '{
> :start
> s/,//1p
> /,/b start
> }'
A test, to, remove, commas.
A test to, remove, commas.
A test to remove, commas.
A test to remove commas.
```
说明：
- 脚本每次迭代都会删除文本中的第一个逗号，并打印字符串
- 第一次示例中有问题，脚本陷入了死循环，不停查找逗号，需要用ctrl+c终止脚本
- 防止进入死循环，可以为分支命令指定一个地址模式来查找，第二次示例中使用了逗号，当脚本中最后一个逗号被删除后，分支命令就不会再执行了

#### 测试
&#8195;&#8195;测试（test）命令（t）也可以用来改变sed编辑器脚本的执行流程，测试命令会根据替换命令的结果跳转到某个标签，而不是根据地址跳转。格式：`[address]t [label]`，说明：
- 如果替换命令成功匹配并替换了一个模式，测试命令就会跳转到指定的标签
- 如果替换命令未能匹配指定的模式，测试命令就不会跳转
- 在没有指定标签的情况下，如果测试成功，sed会跳转到脚本的结尾
- 测试命令提供了对数据流中的文本执行基本的if-then语句的一个低成本的方法

例如已经做了一个替换，不需要再做另一个替换，可以使用测试命令：
```
[root@redhat8 sed]# sed '{
> s/first/matched/
> t
> s/This is the/No match on/
> }' test6
No match on header line!
This is the matched data line!
No match on second data line!
No match on last line!
[root@redhat8 sed]# echo "A, test, to, remove, commas."|sed -n '{
> :start
> s/,//1p
> t start
> }'
A test, to, remove, commas.
A test to, remove, commas.
A test to remove, commas.
A test to remove commas.
```
说明：
- 第一个替换命令会查找模式文本first，匹配了行中的模式，就会替换文本，而且测试命令会跳过后面的替换命令
- 如果第一个替换命令未能匹配到模式，第二个替换命令就会被执行
- 第二个示例演示了用测试命令结束之前用分支命令形成的无限循环
- 当无需替换时，测试命令不会跳转而是继续实行剩下的脚本

### 模式替代
&#8195;&#8195;在使用通配符时，很难知道到底哪些文本会匹配模式。例如想在行中匹配的单词边上放上引号，如果只是要匹配模式中的一个单词，就比较简单,如果模式中使用通配符（.）来匹配多个单词，就满足不了需求：
```
[root@redhat8 sed]# echo "The cat sleeps in his hat."|sed 's/cat/"cat"/'
The "cat" sleeps in his hat.
[root@redhat8 sed]# echo "The cat sleeps in his hat ."|sed 's/.at/".at"/g'
The ".at" sleeps in his ".at" .
```
#### &符号
&#8195;&#8195;&符号可以用来代表替换命令中的匹配的模式，不管模式匹配的是什么文本，都可以在替换模式中使用&符号来使用这段文本，这样就可以操作模式所匹配到的任何单词了，示例如下：
```
[root@redhat8 sed]# echo "The cat sleeps in his hat ."|sed 's/.at/"&"/g'
The "cat" sleeps in his "hat" .
```
说明：
- 当模式匹配到了单词cat，"cat"就会出现在了替换后的单词里面
- 当匹配到了单词hat，"cat"就会出现在了替换后的单词里面

#### 替换单独的单词
&#8195;&#8195;&符号提取匹配替换命令中的指定模式的整个字符串，有时候只想提取字符串的一部分。在sed编辑器中用圆括号来定义替换模式的子模式，子模式说明：
- 可以在替换模式中使用特殊字符来引用每个子模式，替代字符用反斜线和数字组成，数字表示子模式的位置
- sed编辑器会给第一个子模式分配字符\1,给第二个分配字符\2，依此类推

示例如下：
```
[root@redhat8 sed]# echo "The System Administrator manual" | sed '
> s/\(System\) Administrator/\1 User/'
The System User manual
```
说明：
- 在替换命令中使用圆括号时必须用转义字符将他们标为分组而不是普通圆括号
- 示例中替换命令用一对圆括号将单词System括起来，表示其为一个子模式
- 然后在替换模式中使用\1来提起第一个匹配的子模式

&#8195;&#8195;如果需要用一个单词来替换一个短语，而这个单词刚好是该短语的子字符串，但是那个子字符串恰巧使用了通配符，这时候使用子模式会很方便：
```
[root@redhat8 sed]# echo "The furry cat is pretty"|sed 's/furry \(.at\)/\1/'
The cat is pretty
[root@redhat8 sed]# echo "The furry hat is pretty"|sed 's/furry \(.at\)/\1/'
The hat is pretty
```
说明：
- 这种情况下，不能用&符号，因为他会替换整个匹配的模式
- 子模式允许选择将模式中的某部分作为替代模式

当需要在两个或多个子模式间插入文本时，尤为有用。下面示例中使用子模式在大数字中插入逗号：
```
[root@redhat8 sed]# echo "1234567"|sed '{
> :start
> s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/
> t start
> }'
1,234,567
```
说明：
- 示例脚本将匹配模式分成了两部分：`.*[0-9]`和`[0-9]{3}`
- 这个模式会查找两个子模式，第一个子模式是以数字结尾的任意长度的字符，第二个是若干组三位数字，注意正则表达式中使用花括号时候用了转义字符
- 如果这个模式在文本中找到阿里，替代文本会在两个子模式之间加一个逗号，每个子模式都会通过其位置来标示
- 示例脚本使用测试命令来遍历这个数字，知道放置好所有的逗号

### 在脚本中使用sed
为方便查阅，收录在以下位置：[Shell-脚本中使用sed](https://bond-huang.github.io/huang/09-Shell%E8%84%9A%E6%9C%AC/02-Shell%E8%84%9A%E6%9C%AC%E5%BF%AB%E9%80%9F%E6%8C%87%E5%8D%97/05-Shell-%E8%84%9A%E6%9C%AC%E4%B8%AD%E4%BD%BF%E7%94%A8sed.html)
