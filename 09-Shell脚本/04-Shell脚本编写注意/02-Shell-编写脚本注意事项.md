# Shell-编写脚本注意事项
记录一些在编写脚本过程中遇到的需要注意的点。
## 从命令获取值
&#8195;&#8195;当将数据通过awk输出某一列内容时候，赋值给遍变量后，再调用变量时候会发现变成一行，在脚本中使用的时候要注意，示例如下：
```
# lspath
Enabled hdisk0 vscsi0
Enabled hdisk1 vscsi0
Enabled hdisk0 vscsi1
Enabled hdisk1 vscsi1
Enabled hdisk2 vscsi1
# cat test7.sh
#!/bin/ksh
a=`lspath |awk '{print $2}'`
echo $a
echo $a |sed -n '='
echo $a |awk '{print $1}'
# sh test7.sh
hdisk0 hdisk1 hdisk0 hdisk1 hdisk2
1
hdisk0
```
## 字符串比较注意
&#8195;&#8195;变量获取了命令的输出，输出中有空格，如果直接调用去做比较，就会报错，例如下面示例：
```sh
#!/bin/ksh
a=`cat /etc/hosts |sed -n '/teacher01/p'`
b=`cat /etc/hosts |sed -n '/teacher02/p'`
echo $a
echo $b
if [ $a != $b ]
then
    echo "We are different!"
fi
```
运行后示例如下：
```
# sh test8.sh
9.100.103.137 teacher01
9.100.103.138 teacher02
test8.sh[6]: teacher01: 0403-012 A test command parameter is not valid.
```
在进行test的时候，需要加上冒号，示例如下：
```sh
#!/bin/ksh
a=`cat /etc/hosts |sed -n '/teacher01/p'`
b=`cat /etc/hosts |sed -n '/teacher02/p'`
echo $a
echo $b
if [ "$a" != "$b" ]
then
    echo "We are different!"
fi
```
运行后示例如下：
```
# sh test8.sh
9.100.103.137 teacher01
9.100.103.138 teacher02
We are different!
```
## 计数注意事项
参考[Shell-各种shell的区别](https://big1000.com/09-Shell%E8%84%9A%E6%9C%AC/03-Shell_AIX%E8%84%9A%E6%9C%AC/01-Shell-%E5%90%84%E7%A7%8Dshell%E7%9A%84%E5%8C%BA%E5%88%AB.html)章节中的内容。

## 重定向注意事项
&#8195;&#8195;重定向输出可以将命令输出结果指定到某个文件，但是命令执行错误信息不行，示例如下（redhat8中示例）：
```
[root@redhat8 linuxone]# ls -l > mount.log
[root@redhat8 linuxone]# cat mount.log
total 24
lrwxrwxrwx. 1 root root   9 Oct 31 01:09 link1 -> test1.log
-rw-r--r--. 1 root root  44 Oct 31 01:20 ln1
-rwxr-xr-x. 1 root root   0 Oct 31 05:24 mount.log
-rw-r--r--. 1 root root  46 Oct 31 01:40 test1.log
-rw-r--r--. 1 root root 183 Oct 31 01:50 test2.sh
[root@redhat8 linuxone]# mount -a > mount.log
mount: /mnt/not-existing: mount point does not exist.
mount: /mnt/test: special device /tmp/test does not exist.
[root@redhat8 linuxone]# cat mount.log
[root@redhat8 linuxone]# mount -a 2>mount.log
[root@redhat8 linuxone]# cat mount.log
mount: /mnt/not-existing: mount point does not exist.
mount: /mnt/test: special device /tmp/test does not exist.
```
STDERR是Linux标准文件描述符，之前学习过长期不用忘记了，再此处再次记录强调下。

STDERR之前学习笔记：[Shell笔记-数据呈现](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/01-Shell%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/05-Shell%E7%AC%94%E8%AE%B0-%E6%95%B0%E6%8D%AE%E5%91%88%E7%8E%B0.html)

## 正则表达式调用变量
在正则表达式中调用变量时候，加上单引号即可，但是如果变量中有特殊字符，会报错，示例脚本：
```sh
#!/bin/bash
tmp=$(date +"%y%m%d")
mount -a 2> mount$tmp.log
invalid_list=`cat mount$tmp.log | sed -n '/does not exist/p'|gawk 'BEGIN{FS=": "}{print $2}'`
echo $invalid_list
for i in $invalid_list
do
    echo $i
    sed -i '/'$i'/ s/^/# /' /etc/fstab
done
```
运行后示例：
```
[root@redhat8 linuxone]# sh test3.sh
/mnt/not-existing
sed: -e expression #1, char 3: unknown command: `m'
/mnt/test
sed: -e expression #1, char 3: unknown command: `m'
```
解决方法就是匹配到`/`然后在前面都加一个`\`：
```sh
#!/bin/bash
tmp=$(date +"%y%m%d")
mount -a 2> mount$tmp.log
invalid_list=`cat mount$tmp.log | sed -n '/does not exist/p'|gawk 'BEGIN{FS=": "}{print $2}'`
echo $invalid_list
for i in $invalid_list
do
    i=`echo $i |sed 's/\//\\\&/g'`
    echo $i
    sed -i '/'$i'/ s/^/# /' /etc/fstab
done
```
运行后示例：
```
[root@redhat8 linuxone]# sh test3.sh
\/mnt\/not-existing
\/mnt\/test
```
知识点巩固：
- 正则表达式中，`/`需要转义，`&`也需要转移，转义字符`\`本身也需要转义
- `&`符号可以用来代表替换命令中的匹配的模式，本实例中代表匹配模式`/`

## 待补充
