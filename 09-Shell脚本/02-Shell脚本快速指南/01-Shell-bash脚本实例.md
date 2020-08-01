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

### 待补充
