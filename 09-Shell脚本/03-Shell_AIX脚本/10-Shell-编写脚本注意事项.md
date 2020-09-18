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
## 待补充
