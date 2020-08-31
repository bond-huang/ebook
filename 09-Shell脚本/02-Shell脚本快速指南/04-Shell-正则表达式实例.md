# Shell-正则表达式实例
### 目录文件计数
脚本实例如下：
```sh
#!bin/bash
mypath=$(echo $PATH | sed 's/:/ /g')
count=0
for directory in $mypath
do 
    check=$(ls $director)
    for item in $check
    do  
        count=$[ $count + 1 ]
    done
    echo "$directory - $count"
    count=0
done
```
运行后示例如下：
```
[root@redhat8 regular]# sh test1.sh
/usr/local/sbin - 9
/usr/local/bin - 9
/usr/sbin - 9
/usr/bin - 9
/root/bin - 9
```
说明：
- 首先获取了到了$PATH中的目录，并且通过正则表达式替换了$PATH变量中的冒号，用空格替代，方便遍历
- 然后用for循环遍历了$PATH中的目录，并且用ls命令列出了目录中的所有文件
- 再用另一个for循环来遍历每个文件，并未文件计数器进行增值进行计数
- 输出了一个目录和其文件计数后，将计数器归零，再继续下一个$PATH中的目录
- 最后得到了$PATH中的目录下对应的文件数量

### 验证电话号码
之前打算用中国电话号码，中国号码格式比较简单，美国奇葩点，美国常见电话号码有如下常见的形式：
- (123)456-7890
- (123) 456-7890
- 123-456-789
- 123.456.789

从左边开始，匹配思路：
- 最开始可能有左圆括号，也可能没有，使用模式：`^\(?`:脱字符表明数据开始，左圆括号是特殊字符，需要进行转义，问号表明左括号可能有也可能没有，匹配0~1次
- 接着是三位区号，使用模式：`[2-9][0-9]{2}`:美国区号以数字2开头，所以第一个是2~9，后面任意两个数字匹配两次，一共就是三位
- 在区号后，收尾的右圆括号可能存在也可能不存在，使用模式：`\)?`
- 在区号后，可能情况：一个空格，无空格，单破折线或一个点，使用模式：`(| |-|\.)`:使用管道符来匹配三种情况，点号是特殊字符需要转义，三种情况使用圆括号进行分组
- 接下来是交换机号码，使用模式：`[0-9]{3}`
- 交换机号码后，需要匹配：一个空格、一条单破折线或者一个点，使用模式：`(| |-|\.)`
- 最后是尾部四位本地电话分机号：`[0-9]{4}$`

完整格式如下：
```
^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}(| |-|\.)[0-9]{4}$
```
写成脚本后如下（注意在gawk程序在使用正则表达式间隔时候，必须使用--re-interval命令行选项）：
```sh
#!/bin/bash
# script to filter out bad phone numbers
gawk --re-interval '/^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}(| |-|\.)[0-9]{4}$/{print $0}'
```
脚本使用示例如下：
```
[root@redhat8 regular]# echo "421-777-3249" |sh isphone.sh
421-777-3249
[root@redhat8 regular]# echo "123-777-3249" |sh isphone.sh
[root@redhat8 regular]# echo "(324) 777-3249" |./isphone.sh
(324) 777-3249
[root@redhat8 regular]# cat test9
000-000-0000
324-523-3425
(634) 673-3678
123-456-7890
452.578.7839
[root@redhat8 regular]# cat test9 | ./isphone.sh
324-523-3425
(634) 673-3678
452.578.7839
```
### 解析邮件地址
邮件级别格式：`username@hostname`,username值可以用字母数字字符以及以下特殊字符构成：
- 点号
- 单破折号
- 加号
- 下划线

邮件地址的hostname部分由一个或多个域名和一个服务器名组成，服务器名只允许字母数字字符以及以下特殊字符：
- 点号
- 下划线

正则表达式模式思路：
- username正则表达式模式：`^([a-zA-Z0-9_\-\.\+]+)@`:分组指定了username中允许的字符，后面加号表示至少有一个字符，就是匹配一次或多次
- hostname使用相同方法匹配服务器名和子域名：`([a-zA-Z0-9_\-\.]+)`,这个模式可以匹配：
    - server
    - server.subdomain
    - server.subdomain.subdomain
- 对于一些顶级域名，只能是字母字符，至少有两个字符，并且长须不得超过5个字符，正则表达式模式：`\.([a-zA-Z]{2,5})$`

整个模式如下：
```
^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
```
写成脚本后如下（注意这里没有使用--re-interval命令行选项）：
```sh
#!/bin/bash
gawk '/^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$/{print $0}'
```
使用示例如下：
```
[root@redhat8 regular]# echo "test@163.com" | sh isemail.sh
test@163.com
[root@redhat8 regular]# echo "test123@cn.ibm.com" | ./isemail.sh
test123@cn.ibm.com
[root@redhat8 regular]# echo "bond-test@gmail-com" | ./isemail.sh
[root@redhat8 regular]# echo "bond-test@gmail.com" | ./isemail.sh
bond-test@gmail.com
[root@redhat8 regular]# echo "bond/test@gmail.com" | ./isemail.sh
[root@redhat8 regular]# echo "bond_test@gmail.com.c" | ./isemail.sh
[root@redhat8 regular]# echo "bond_test@gmail.com" | ./isemail.sh
bond_test@gmail.com
```
