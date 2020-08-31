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

##### 单行next命令
&#8195;&#8195;单行next命令ui将数据流中的下一个文本行移动到sed编辑器的工作空间（称为模式空间）；多行版的next命令（用大写N）会将下一个文本添加到模式空间中已有的文本后，示例如下：
