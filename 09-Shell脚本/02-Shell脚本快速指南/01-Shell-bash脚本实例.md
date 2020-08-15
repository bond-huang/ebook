# Shell笔记-脚本实例
学习Shell脚本过程中，一些比较有实际意义的脚本实例，统一整理在此方便查阅。
### 查找可执行文件
想找出系统中有哪些可执行文件可以使用，扫描PATH环境变量中所有目录即可，代码如下：
```sh
#!/bin/bash
IFS=:
for folder in $PATH
do
    echo "$folder:"
    for file in $folder/*
    do
        if [ -x $file ]
        then
            echo "  $file"
        fi
    done
done
```
运行后输出如下（截取部分）：
```
[root@redhat8 instance]# sh test1.sh |more
/usr/local/sbin:
/usr/local/bin:
/usr/sbin:
  /usr/sbin/accept
```
### 创建多个用户账户
创建一个文件，里面写入需要创建的用户：
```
[root@redhat8 instance]# cat users.csv
Steve,Captain America
Tony,Iron Man
```
文件中第一个列入两行，每一行两个条目，第一个条目是用户ID，第二个条目是用户的全名，之间中逗号隔开，这形成了一种名为逗号分隔符的文件格式（.csv),这种格式在电子表格中极为常见，文件保存格式为.csv。
```sh
#!/bin/bash
#process new user accounts
input="users.csv"
while IFS=',' read -r userid name
do
    echo "Adding user:$userid"
    useradd -c "$name" -m $userid
done < "$input"
```
运行脚本：
```
[root@redhat8 instance]# sh test2.sh
Adding user:Steve
Adding user:Tony
```
查看/etc/passwd文件，可以看到用户已经创建了：
```
Steve:x:1001:1001:Captain America:/home/Steve:/bin/bash
Tony:x:1002:1002:Iron Man:/home/Tony:/bin/bash
```
知识点：
- 将IFS设置成逗号，read会根据格式读取行内容，并将值赋给定义变量，并且迭代一次后自动读取文本的下一行内容，当read读完整个文件时候，也就是读不到数据返回False，while循环就会终止
- 把数据从文件中送入while命令，只需要在while命令尾部使用一个输入重定向符号

### 文件重定向实例
&#8195;&#8195;文件重定向常见于脚本需要读入文件和输出文件时。例如读取.csv格式表格，然后创建INSERT语句将数据插入MySQL数据库，脚本示例如下：
```sh
#!/bin/bash
outfile='members.sql'
IFS=','
while read lname fname address city state zip
do 
    cat >> $outfile << EOF
    INSERT INTO members (lname,fname,address,city,state,zip) VALUES
('$lname','$fname','$address','$city','$state','$zip');
EOF
done < ${1}
```
读取的文件内容：
```
Captain,America,Fu Tian,Shen Zhen,Guang Dong,518000
Iron,Man,Long Hua,Shen Zhen,Guang Dong,518000
```
脚本说明：
- while循环使用read语句从数据中读取文本
- 在done语句中，当允许程序时，$1代表第一个命令行参数，它指明了待读取数据的文件，read会使用IFS字符解析读入的文本，这里将IFS指定为逗号
- 在cat语句中，包含一个输出追加重定向（双大于号）和一个输入追加重定向（双小于号），输出重定向将cat命令的输出追加到由$outfile变量指定的文件中。cat命令的输入不再取自标准的输入，而是被重定向到脚本中存储的数据，EOF标记了追加到文件中的数据的岂止
- 在脚本中，由一个标准的SQL INSERT语句，其中的数据会由变量来替换，变量的内容则是read语句存入的
- 在脚本中，while循环一次读取一行数据，将这些值放入INSERT语句模板中，然后将结果输出到输出文件中

脚本运行后示例如下：
```
[root@redhat8 function]# sh test1.sh test.csv
[root@redhat8 function]# cat members.sql
    INSERT INTO members (lname,fname,address,city,state,zip) VALUES
('aptain','America','Fu Tian','Shen Zhen','Guang Dong','518000');
    INSERT INTO members (lname,fname,address,city,state,zip) VALUES
('Iron','Man','Long Hua','Shen Zhen','Guang Dong','518000');
```
