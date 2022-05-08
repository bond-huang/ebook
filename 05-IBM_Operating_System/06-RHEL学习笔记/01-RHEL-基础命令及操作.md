# RHEL-基础命令及操作
复习已知知识，记录学到的新知识。
## RHEL中获取帮助
### man page
&#8195;&#8195;本地系统上通常可用的一个文档源是系统手册页，或称为`man page`。这些手册页是作为文档所涉及的相应软件包的一部分而提供的，使用`man`命令从命令行进行访问。Man Page导航：

命令|描述
:---|:---
Space|向前（向下）滚动一个屏幕
PageDown|向前（向下）滚动一个屏幕
PageUp|向后（向上）滚动一个屏幕
DownArrow|向前（向下）滚动一行
UpArrow|向后（向上）滚动一行
D|向前（向下）滚动半个屏幕
U|向后（向上）滚动半个屏幕
/string|在man page中向前（向下）搜索string
N|在man page中重复之前的向前（向下）搜索
Shift+N|在man page中重复之前的向后（向上）搜索
G|转到man page的开头。
Shift+G|转到man page的末尾。
Q|退出man，并返回到命令shell提示符

&#8195;&#8195;通过`man -k keyword`对man page执行关键字搜索，会显示与关键字匹配的`man page`主题和章节编号的列表。`man page`常见的标题有：

标题|描述
:---|:---
NAME|主题名称。通常是命令或文件名。非常简短的描述
SYNOPSIS|命令语法的概要
DESCRIPTION|提供对主题的基本理解的深度描述
OPTIONS|命令执行选项的说明
EXAMPLES|有关如何使用命令、功能或文件的示例
FILES|与man page相关的文件和目录的列表
SEE ALSO|相关的信息，通常是其他man page主题
BUGS|软件中的已知错误。
AUTHOR|有关参与编写该主题的人员的信息

使用`man -k zip`命令来列出关于ZIP存档的详细信息：
```
[student@workstation ~]$ man -k zip
...output omitted...
zipinfo (1)          - list detailed information about a ZIP archive
zipnote (1)          - write the comments in zipfile to stdout, edit comments and rename files in zipfile
zipsplit (1)         - split a zipfile into smaller zipfiles
```
&#8195;&#8195;所有`man page`都位于`/usr/share/man`。使用`whereis`命令，查找位于`/usr/share/man`目录中的二进制文件、源代码和`man page`。示例：
```
[root@redhat8 ~]# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz
```
使用`man -t`命令，创建`ls`man page的格式化文件。
```
[root@redhat8 ~]# man -t ls >ls.ps
[root@redhat8 ~]# ls -al
total 1956
-rw-r--r--.  1 root root   20051 May  2 14:39  ls.ps
[root@redhat8 ~]# file ~/ls.ps
/root/ls.ps: PostScript document text conforming DSC level 3.0
```
### 阅读Info文档
&#8195;&#8195;`man page`的格式作为命令参考时很有用，但作为普通文档却用处不大。对于此类文档，GNU项目开发了一种不同的在线文档系统，称为`GNU Info`。Info文档是Red Hat Enterprise Linux系统上重要的资源。要启动Info文档查看器，可使用`pinfo`命令。`pinfo`会在顶级目录中打开：
```
[root@redhat8 ~]# man passwd
[root@redhat8 ~]# pinfo passwd
```
## 命令及命令行基础
### 编辑命令行
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

### 基础命令
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
[root@redhat8 ~]# date +%s
1651512964
```
`passwd`命令更改用户自己的密码。必须指定该帐户的原始密码，之后才允许进行更改：
- 默认情况下，需要强密码，其包含小写字母、大写字母、数字和符号，
- 并且不以字典中的单词为基础
- 超级用户可以使用`passwd`命令更改其他用户的密码

&#8195;&#8195;Linux不需要文件扩展名来根据类型分类文件。`file`命令可以扫描文件内容的开头，显示该文件的类型。要分类的文件作为参数传递至该命令：
```
[root@redhat8 ~]# file /etc/passwd
/etc/passwd: ASCII text
[root@redhat8 ~]# file /bin/passwd
/bin/passwd: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /
lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=a3637110e27e9a48dced9f38b4ae43388d32d0e4, stripped[root@redhat8 ~]# file /var
/var: directory
```
&#8195;&#8195;命令`cat`可以创建单个或多个文件，查看文件内容，串联多个文件中的内容，以及将文件内容重定向到终端或文件。查看多个文件时候使用空格间隔，示例如下：
```
[root@redhat8 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
[root@redhat8 ~]# cat /etc/hosts /etc/passwd
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
  ...output omitted...
```
&#8195;&#8195;`less`命令允许在篇幅超过一个终端窗口适合大小的文件中向前和向后翻页。使用UpArrow键和DownArrow键可向上和向下滚动显示。按`q`键退出该命令。    
&#8195;&#8195;`head`和`tail`命令分别显示文件的开头和结尾部分。默认情况下显示文件的10行，都有`-n`选项用来指定不同的行数。要显示的文件作为参数传递至这些命令：
```
[root@redhat8 ~]# head -n 3 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
[root@redhat8 ~]# tail -n 2 /etc/passwd
harry:x:1005:1005::/home/harry:/bin/bash
sarah:x:1006:1006::/home/sarah:/sbin/nologin
```  
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
- 可以使用`Esc+.`或`Alt+.` 组合键，在光标的当前位置插入上一命令的最后一个单词。重复使用组合键可将该文本替换为历史记录中更早命令的最后一个单词。`Alt+.`组合键尤其方便，可以按住`Alt`键，再反复按`.`键来轻松地回滚到更早的命令

### TAP补全
&#8195;&#8195;`Tab`补全允许用户在提示符下键入足够的内容以使其唯一后快速补全命令或文件名。如果键入的字符不唯一，则按`Tab`键两次可显示以键入的字符为开头的所有命令。许多命令可以通过`Tab`补全匹配参数和选项，例如`useradd`：
```
[root@redhat8 ~]# pas  "Push Tap twice"
passwd       paste        pasuspender 
[root@redhat8 ~]# useradd -- "Push Tap twice"
--base-dir        --expiredate      --home-dir        --no-log-init     --prefix          --skel
--comment         --gid             --inactive        --non-unique      --root            --system
--create-home     --groups          --key             --no-user-group   --selinux-user    --uid
--defaults        --help            --no-create-home  --password        --shell           --user-group
```
### 在另一行上继续长命令
&#8195;&#8195;具有多个选项和参数的命令可能会很快变得很长，当光标到达右边缘时，命令窗口会自动换行。为了提高命令的易读性，可以使用多行来键入长命令。使用反斜杠字符`\`（称为转义字符）忽略紧跟在反斜杠后面的字符的含义。for循环等一些语句中不需要，示例：
```
[root@redhat8 sed_gawk]# sed '/2. I am/c\
> I will save the world!' test7
[root@redhat8 ~]# for i in {2..5}
> do
> echo number-$i
> done
number-2
number-3
number-4
number-5
```
## 命令行管理文件
### RHEL重要目录
重要目录及描述如下：
- `/usr`：安装的软件、共享的库，包括文件和只读程序数据。重要的子目录有：
  - `/usr/bin`：用户命令
  - `/usr/sbin`：系统管理命令
  - `/usr/local`：本地自定义软件
- `/etc`：特定于此系统的配置文件
- `/var`：特定于此系统的可变数据，在系统启动之间保持永久性。动态变化的文件（如数据库、缓存目录、日志文件、打印机后台处理文档和网站内容）
- `/run`：自上一次系统启动以来启动的进程的运行时数据。这包括进程ID文件和锁定文件等。此目录中的内容在重启时重新创建。此目录合并了早期版本的RHEL中的`/var/run`和`/var/lock`
- `/home`：主目录是普通用户存储其个人数据和配置文件的位置
- `/root`：管理超级用户root的主目录
- `/tmp`：供临时文件使用的全局可写空间。10天内未访问、未更改或未修改的文件将自动从该目录中删除：
  - 还有一个临时目录`/var/tmp`，该目录中的文件如果在30 天内未曾访问、更改或修改过，将被自动删除
- `/boot`：开始启动过程所需的文件
- `/dev`：包含特殊的设备文件，供系统用于访问硬件

### 导航路径
导航路径相关命令说明：
- `pwd`命令显示该shell的当前工作目录的完整路径名。这可以帮助确定使用相对路径名来访问文件的语法
- `ls`命令列出指定目录的目录内容；如果未指定目录，则列出当前工作目录的内容。常见且最有用的选项是:
  - `-l`（长列表格式）
  - `-a`（包含隐藏文件在内的所有文件）：列表顶部的两个特殊目录是当前目录`.`和父目录`..`
  - `-R`（递归方式，包含所有子目录的内容） 
- 使用`cd`命令可更改shell的当前工作目录：
  - 没有为该命令指定任何参数，将切换主目录
  - `cd -`命令可更改到用户在进入当前目录之前所处的目录
- 当当前工作目录是主目录时，提示符显示波形符字符`~`
- `touch`该命令通常将文件的时间戳更新为当前日期和时间，而不进行其他修改。通常可用于创建空文件，因为`touch`不存在的文件名会导致创建该文件

`cd -`命令示例：
```
[root@redhat8 ~]# pwd
/root
[root@redhat8 ~]# cd /home/huang
[root@redhat8 huang]# cd -
/root
```
将工作目录从当前位置上移两个级别：
```
[root@redhat8 test]# pwd
/home/huang/test
[root@redhat8 test]# cd ../..
[root@redhat8 home]# 
```
### 常用文件管理命令
最常见的文件管理命令：
  
命令|描述|命令语法
:---|:---|:---
mkdir|创建目录|mkdir directory
cp|复制文件|cp file new-file
cp -r|复制目录及其内容|cp -r directory new-directory
mv|移动或重命名文件或目录|mv file new-file
rm|删除文件|rm file
rm -r|删除含有文件的目录|rm -r directory
rmdir|删除空目录|rmdir directory

### 创建目录
使用`mkdir`命令创建目录说明及注意事项：
-  `mkdir`命令可创建一个或多个目录或子目录。取要创建的目录的路径列表作为参数
- 如果创建目录已存在，或者试图在一个不存在的目录中创建子目录，`mkdir`命令将失败并出现错误。
- `-p`（父级）选项将为请求的目标位置创建缺失的父目录。使用`mkdir -p`命令时应小心，拼写错误会创建不需要的目录，而不会生成错误消息
- 创建多个父目录及子目录时候使用空格分隔

示例：
```
[root@redhat8 ~]# mkdir superhero/thor
mkdir: cannot create directory ‘superhero/thor’: No such file or directory
[root@redhat8 ~]# mkdir -p superhero/thor
[root@redhat8 ~]# ls -l superhero/thor
total 0
```
### 复制文件
&#8195;&#8195;`cp`命令可复制文件，在当前目录或指定目录中创建新文件。它也可将多个文件复制到某一目录中。 注意事项：
- 如果目标文件已存在，则`cp`命令会覆盖该文件
- 在通过一个命令复制多个文件时，最后一个参数必须为目录
- 复制的文件在新的目录中保留其原有名称
- 如果目标目录中存在具有相同名称的文件，则会覆盖现有文件
- 默认情况下，`cp`不复制目录，而会忽略它们
- 如果列出了两个目录，A和B，只有最后参数B可以有效作为目标，A目录被忽略

### 移动文件
&#8195;&#8195;`mv`命令可将文件从一个位置移动到另一个位置。如果将文件的绝对路径视为它的全名，那么移动文件实际上和重命名文件一样，文件内容保持不变。
### 删除文件和目录
`rm`命令可删除文件：
- 默认情况下，除非添加了`-r` 或`--recursive`选项，否则`rm`不会删除包含文件的目录
- 在删除文件或目录之前，最好先使用`pwd`验证当前工作目录
- 如果试图使用`rm`命令来删除目录，但不使用`-r`选项，命令将失败：
  - `rm -r`命令首先遍历每个子目录，在删除每个目录之前逐一删除其中的文件
  - 可以使用`rm -ri`命令以交互方式提示确认，然后再删除
- `-f`选项强制删除而不提示用户进行确认
- 如果同时指定了`-i`和`-f`选项，`-f`选项将具有优先权，在`rm`删除文件之前，不会提示进行确认
- 不带任何选项的`rm `命令无法删除空目录。必须使用`rmdir`命令（不能删除非空目录）、`rm -d`（等同于rmdir）或`rm -r`

示例如下：
```
[root@redhat8 ~]# ls -l superhero/thor
total 0
-rw-r--r--. 1 root root 0 May  2 14:03 Asgard.txt
[root@redhat8 ~]# rmdir superhero/thor
rmdir: failed to remove 'superhero/thor': Directory not empty
[root@redhat8 ~]# rm superhero/thor
rm: cannot remove 'superhero/thor': Is a directory
[root@redhat8 ~]# rm -r superhero/thor
rm: descend into directory 'superhero/thor'? y
rm: remove regular empty file 'superhero/thor/Asgard.txt'? y
rm: remove directory 'superhero/thor'? y
[root@redhat8 ~]# ls -l superhero/thor
ls: cannot access 'superhero/thor': No such file or directory
```
### 硬链接
&#8195;&#8195;从初始名称到文件系统上的数据，每个文件都以一个硬链接开始。当创建指向文件的新硬链接时，也会创建另一个指向同一数据的名称。新的硬链接与原始文件名的作用完全相同。通过`ls -l`命令来确定某个文件是否有多个硬链接：
```
[root@redhat8 huang]# ls -l testfile
-rw-rw-r--. 1 huang huang 259 Jul 26  2020 testfile
```
&#8195;&#8195;示例中，`testfile`的链接数为1。有一个绝对路径/home/huang/testfile。可以使用`ln`命令创建一个指向现有文件的新硬链接（另一个名称）。该命令至少需要两个参数，即现有文件的路径以及要创建的硬链接的路径：
```
[root@redhat8 huang]# ln testfile /tmp/testfile-llink2
[root@redhat8 huang]# ls -l testfile /tmp/testfile-llink2
-rw-rw-r--. 2 huang huang 259 Jul 26  2020 testfile
-rw-rw-r--. 2 huang huang 259 Jul 26  2020 /tmp/testfile-llink2
```
&#8195;&#8195;使用`ls`命令的`-i`选项，以列出文件的索引节点编号。如果文件位于同一文件系统上，而且它们的索引节点编号相同，那么这两个文件就是指向同一数据的硬链接：
```
[root@redhat8 huang]# ls -il testfile /tmp/testfile-llink2
35330643 -rw-rw-r--. 2 huang huang 259 Jul 26  2020 testfile
35330643 -rw-rw-r--. 2 huang huang 259 Jul 26  2020 /tmp/testfile-llink2
``` 
硬链接的特点：
- 引用同一文件的所有硬链接将具有相同的链接数、访问权限、用户和组所有权、时间戳，以及文件内容
- 如果使用一个硬链接更改其中的任何信息，指向同一文件的所有其他硬链接也会显示新的信息，因为每个硬链接都指向存储设备上的同一数据
- 即使原始文件被删除，只要存在至少一个硬链接，该文件的内容就依然可用。只有删除了最后一个硬链接时，才会将数据从存储中删除

硬链接局限性：
- 硬链接只能用于常规文件，不能使用`ln`来创建指向目录或特殊文件的硬链接 
- 只有当两个文件都位于同一文件系统上时，才能使用硬链接

### 软链接
&#8195;&#8195;命令`ln -s`可创建软链接，也称为符号链接。软链接不是常规文件，而是指向现有文件或目录的特殊类型的文件。软链接相比硬链接有一定的优势及不同： 
- 软链接可以链接位于不同文件系统上的两个文件
- 软链接可以指向目录或特殊文件，而不仅限于常规文件 
- 硬链接是将名称指向存储设备上的数据 
- 软链接则是将名称指向另一个名称，硬链接指向存储设备上的数据
- 软链接可以指向目录，之后软链接发挥目录一样的作用：
  - 通过`cd`更改为软链接将使当前工作目录变为链接目录
  - 有些工具可以跟踪使用软链接到达当前工作目录的事实。例如，默认情况下，`cd`将使用软链接的名称（而非实际目录的名称）来更新当前工作目录
  - `cd`命令有一个选项`-P`会将其更新为实际目录的名称

软链接示例：
```
[root@redhat8 ~]# ln -s /home/huang/testfile-llink2 /tmp/testfile-symlink
[root@redhat8 ~]# ls -l /home/huang/testfile-llink2 /tmp/testfile-symlink
-rw-r--r--. 1 root root  0 May  2 09:21 /home/huang/testfile-llink2
lrwxrwxrwx. 1 root root 27 May  2 09:22 /tmp/testfile-symlink -> /home/huang/testfile-llink2
[root@redhat8 ~]# rm -f /home/huang/testfile-llink2
[root@redhat8 ~]# ls -l /tmp/testfile-symlink
lrwxrwxrwx. 1 root root 27 May  2 09:22 /tmp/testfile-symlink -> /home/huang/testfile-llink2
[root@redhat8 ~]# cat /tmp/testfile-symlink
cat: /tmp/testfile-symlink: No such file or directory
```
示例软链接说明：
- 示例中`/tmp/testfile-symlink`的长列表的第一个字符是`l`，而不是`-`。表示该文件是软链接而不是常规文件
- 当原始常规文件被删除后，软链接依然会指向该文件，但目标已消失。指向缺失的文件的软链接称为`悬挂的软链接` 
- 在上例中，悬挂的软链接有一个副作用，如果稍后创建了一个与已删除文件同名的新文件 (/home/huang/testfile-llink2)，那么软链接将不再`悬挂`，而是指向此新文件

## 使用Shell扩展匹配文件名
### 模式匹配
元字符和匹配项表：

模式|匹配项
:---|:---
\*|由零个或更多字符组成的任何字符串
?|任何一个字符
\[abc...]|括起的类（位于两个方括号之间）中的任何一个字符
\[!abc...]|不在括起的类中的任何一个字符
\[^abc...]|不在括起的类中的任何一个字符
\[[:alpha:]]|任何字母字符
\[[:lower:]]|任何小写字符
\[[:upper:]]|任何大写字符
\[[:alnum:]]|任何字母字符或数字
\[[:punct:]]|除空格和字母数字以外的任何可打印字符
\[[:digit:]]|从0到9的任何单个数字
\[[:space:]]|任何一个空白字符。这可能包括制表符、换行符、回车符、换页符或空格

简单模式匹配示例：
```
[root@redhat8 ~]# mkdir glob;cd glob
[root@redhat8 glob]# touch alfa bravo charlie delta echo able baker cast dog easy
[root@redhat8 glob]# ls
able  alfa  baker  bravo  cast  charlie  delta  dog  easy  echo
[root@redhat8 glob]# ls c*
cast  charlie
[root@redhat8 glob]# ls *c*
cast  charlie  echo
[root@redhat8 glob]# ls [ac]*
able  alfa  cast  charlie
[root@redhat8 glob]# ls ????
able  alfa  cast  easy  echo
[root@redhat8 glob]# ls ?????
baker  bravo  delta
```
### 波形符扩展
波形符`~`可匹配当前用户的主目录：
- 如果开始时使用斜杠`/`以外的字符串，shell就会将该斜杠之前的字符串解译为用户名
- 如果存在匹配项，则用该用户的主目录绝对路径来替换此字符串
- 如果找不到匹配的用户名，则使用实际波形符加上该字符串代替

使用`echo`命令显示波形符字符的值示例：
```
[root@redhat8 glob]# echo ~root
/root
[root@redhat8 glob]# echo ~/glob
/root/glob
[root@redhat8 glob]# echo ~
/root
[root@redhat8 glob]# echo ~huang
/home/huang
```
### 大括号扩展
&#8195;&#8195;大括号扩展用于生成任意字符串。大括号包含字符串的逗号分隔列表或顺序表达式。结果包含大括号定义之前或之后的文本。大括号扩展可以互相嵌套。此外，双句点语法`..`可扩展成一个序列，使得`{m..p}`扩展为`m n o p`：
```
[root@redhat8 glob]# echo file{1..4}.txt
file1.txt file2.txt file3.txt file4.txt
[root@redhat8 glob]# echo file{b..e}.txt
fileb.txt filec.txt filed.txt filee.txt
[root@redhat8 glob]# echo file{1,2}{a,b}.txt
file1a.txt file1b.txt file2a.txt file2b.txt
[root@redhat8 glob]# echo file{a{1,2},b,c}.ext
filea1.ext filea2.ext fileb.ext filec.ext
[root@redhat8 glob]# echo {Monday,Tuesday,Wednesday,Thursday}.log
Monday.log Tuesday.log Wednesday.log Thursday.log
[root@redhat8 glob]# mkdir ../RHEL{1..4}
[root@redhat8 glob]# LS ../RHEL*
bash: LS: command not found...
Similar command is: 'ls'
[root@redhat8 glob]# ls ../RHEL*
../RHEL1:
../RHEL2:
../RHEL3:
../RHEL4:
```
### 变量扩展
&#8195;&#8195;变量的作用类似于可以在内存中存储值的命名容器。通过变量，可以从命令行或在 shell脚本内轻松访问和修改存储的数据。将数据作为值分配给变量语法如下：
```
[root@redhat8 ~]# VARIABLENAME=value
```
&#8195;&#8195;可以使用变量扩展将变量名称转换为命令行上的值。如果字符串以美元符号`$`开头，那么shell就会尝试将该字符串的其余部分用作变量名称，并将它替换为变量中包含的任何值。
为了避免因其他shell扩展而引起的错误，可以将变量的名称放在大括号中，如`${VARIABLENAME}`，示例如下：
```
[root@redhat8 ~]# SUPERHERO=batman
[root@redhat8 ~]# echo $SUPERHERO
batman
[root@redhat8 ~]# echo ${SUPERHERO}
batman
```
### 命令替换
&#8195;&#8195;命令替换允许命令的输出替换命令行上的命令本身。当命令括在括号中并且前面有美元符号`$`时，会发生命令替换。`$(Command)`形式可以互相嵌套多个命令扩展：
```
[root@redhat8 ~]# echo Today is $(date +%A).
Today is Monday.
[root@redhat8 ~]# echo The time is $(date +%M) minutes past $(date +%l%p).
The time is 45 minutes past 11AM.
```
较旧形势的命令喜欢使用反引号`Command`，缺点：
- 反引号在视觉上很容易与单引号混淆
- 反引号无法嵌套

### 防止参数被扩展
&#8195;&#8195;Bash shell中，许多字符有特殊含义。为了防止shell在命令行的某些部分上执行 shell扩展，可以为字符和字符串加引号或执行转义。反斜杠`\`是Bash shell中的转义字符：
```
[root@redhat8 ~]# echo My home directory is $HOME
My home directory is /root
[root@redhat8 ~]# echo My home directory is \$HOME
My home directory is $HOME
```
&#8195;&#8195;如果要保护较长的字符串，则使用单引号`'`或双引号`"`来括起字符串。单引号将阻止所有shell扩展，双引号则阻止大部分shell扩展，双引号可以阻止通配和shell扩展，但依然允许命令和变量替换，示例如下：
```
[root@redhat8 ~]# echo "$HOSTNAME home directory is $HOME"
redhat8 home directory is /root
[root@redhat8 ~]# echo '$HOSTNAME home directory is $HOME'
$HOSTNAME home directory is $HOME
```
## 练习