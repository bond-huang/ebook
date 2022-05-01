# RHEL-Bash命令笔记
记录没用过或很少用或用过但是用法新学的命令及示例。
## 编辑命令行
&#8195;&#8195;以交互方式使用时，bash具有命令行编辑功能。可以使用文本编辑器命令在当前键入的命令内移动并进行修改，也可访问命令历史记录。命令行编辑实用快捷键如下表：

快捷键|描述
:---|:---
Ctrl+A|跳到命令行的开头
Ctrl+E|跳到命令行的末尾
Ctrl+U|将光标处到命令行开头的内容清除
Ctrl+K|将光标处到命令行末尾的内容清除
Ctrl+LeftArrow|跳到命令行中前一字的开头
Ctrl+RightArrow|跳到命令行中下一字的末尾
Ctrl+R|在历史记录列表中搜索某一模式的命令

## 基础命令
&#8195;&#8195;`date`命令可显示当前的日期和时间，也可以用它来设置系统时钟。以加号`+`开头的参数可指定日期命令的格式字符串：
```
[root@redhat8 ~]# date
Sun May  1 07:48:59 EDT 2022
[root@redhat8 ~]# date +%R
07:49
[root@redhat8 ~]# date +%x
05/01/2022
[root@redhat8 ~]# date +%r
07:51:04 AM
```
&#8195;&#8195;Linux不需要文件扩展名来根据类型分类文件。`file`命令可以扫描文件内容的开头，显示该文件的类型。要分类的文件作为参数传递至该命令：
```
[root@redhat8 ~]# file /etc/passwd
/etc/passwd: ASCII text
[root@redhat8 ~]# file /bin/passwd
/bin/passwd: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /
lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=a3637110e27e9a48dced9f38b4ae43388d32d0e4, stripped[root@redhat8 ~]# file /var
/var: directory
```
&#8195;&#8195;`less`命令允许在篇幅超过一个终端窗口适合大小的文件中向前和向后翻页。使用UpArrow键和DownArrow键可向上和向下滚动显示。按`q`键退出该命令。

&#8195;&#8195;`wc`命令可计算文件中行、字和字符的数量。它接受`-l`、`-w`或`-c`选项，分别用于仅显示行数、字数或字符数：
```
[root@redhat8 ~]# wc /etc/passwd
  52  112 2814 /etc/passwd
[root@redhat8 ~]# wc -l /etc/shadow ; wc -l /etc/passwd
52 /etc/shadow
52 /etc/passwd
[root@redhat8 ~]# wc -c /etc/hosts /etc/group
 158 /etc/hosts
1087 /etc/group
1245 total
```
&#8195;&#8195;`history`命令显示之前执行的命令的列表，带有命令编号作为前缀。感叹号字符`!`是元字符，用于扩展之前的命令而不必重新键入它们。`!number`命令扩展至与指定编号匹配的命令。`!string`命令扩展至最近一个以指定字符串开头的命令：
```
[user@host ~]$ history
  ...output omitted...
  510  file /var
  511  wc /etc/passwd
  512  wc -l /etc/shadow ; wc -l /etc/passwd
  513  wc -c /etc/hosts /etc/group
  514  ls -l
  515  pwd
  516  history
[root@redhat8 ~]# !ls
ls -l
total 1856
-rw-------. 1 root root    1364 Jul 19  2020  anaconda-ks.cfg
drwxr-xr-x. 2 root root       6 Mar 19  2021  Desktop
...output omitted...
[root@redhat8 ~]# !515
pwd
/root
```
更多使用说明：
- 方向键可用于在shell历史记录中的以往命令之间导航：
    - `UpArrow`编辑历史记录列表中的上一个命令
    - `DownArrow`编辑历史记录列表中的下一个命令
    - `LeftArrow`和`RightArrow`在历史列表中的当前命令中左右移动光标，以便您在运行命令之前进行编辑
- 可以使用`Esc+.`或`Alt+.` 组合键，在光标的当前位置插入上一命令的最后一个单词。重复使用组合键可将该文本替换为历史记录中更早命令的最后一个单词。`Alt+.`组合键尤其方便，可以按住`Alt`键，再反复按`.`键来轻松地回滚到更早的命令。 

