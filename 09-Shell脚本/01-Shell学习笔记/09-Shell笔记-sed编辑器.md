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
