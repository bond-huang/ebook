# Shell-各种shell的区别
记录学习过程中发现的各种shell的区别，主要是AIX的ksh和所学习的bash的差异。
## HMC shell的区别
&#8195;&#8195;HMC底层是RedHat，hscroot用户使用的shell是hmcbash，应该也是基于bash，只不过是修改过。目前就发现了HMC shell中没有bc计算器，计算方法可以用双括号或者方括号，具体可以参照：
[HMC-shell脚本使用](https://bond-huang.github.io/huang/01-IBM_Power_System/01-HMC/03-HMC-shell%E8%84%9A%E6%9C%AC%E4%BD%BF%E7%94%A8.html)
## AIX shell的区别
&#8195;&#8195;AIX和VIOS默认是ksh，AIX也可以安装一个bash，但是使用AIX系统的用户大多数情况下是不会安装的。ksh和bash基本差不多，但是还是有点区别，目前发现了几点，记录下来。

### 数字范围差别
在RHEL bash中表示数字范围三种方法示例：
```
[root@redhat8 test]# for ((i=0;i<=2;i++)); do echo $i; done
[root@redhat8 test]# for i in {0..2}; do echo $i; done
[root@redhat8 test]# for i in `seq 0 2`;do echo $i; done
```
在AIX7.1.4.3 ksh中使用上面三种方法示例：
```
# for ((i=0;i<=2;i++)); do echo $i; done
ksh: 0403-057 Syntax error: `(' is not expected.
# for i in {0..2}; do echo $i; done
{0..2}
# for i in `seq 0 2`;do echo $i; done
ksh: seq:  not found.
```
在AIX7.1.4.3 bash中使用上面三种方法示例：
```
bash-5.0# for ((i=0;i<=2;i++)); do echo -n $i; done
0
1
2
bash-5.0# for i in {0..2}; do echo $i ; done
0
1
2
bash-5.0# for i in `seq 0 2`;do echo $i; done
bash: seq: command not found
```
总结：
- AIX ksh中三种方法都不行，在没安装bash情况下很不方便，目前我还不知道直接替代方案，都是间接去替代
- AIX 安装bash后可以使用前两种方法，第三种方法说明AIX中没有seq命令，前两种已经够用了

### 数组表示差异
在RHEL8中可以使用圆括号表示数组，并且可以进行切片操作：
```
[root@redhat8 test]# a=(1 2 3)
[root@redhat8 test]# b=(4 5 6)
[root@redhat8 test]# for i in `seq 0 2`;do
> echo "${a[i]} ${b[i]}"
> done
1 4
2 5
3 6
```
在AIX7.1.4.3中圆括号不行，改成bash也不行，直接去掉括号改成其它也不行，示例如下：
```
bash-5.0# cat test5.sh
#!/bin/ksh
a=(1 2 3)
b=(4 5 6)
for i in 0 1 2
do
echo "${a[i]} ${b[i]}"
done
bash-5.0# sh test5.sh
test5.sh[2]: 0403-057 Syntax error at line 2 : `(' is not expected.
```
目前还没找到有效直接替代方法。

### case差异
在RHEL的bash中，read命令和case可以结合使用,示例如下：
```sh
#!/bin/bash
read -n1 -p "Need to synchronize the Cluster?[Y/N] " answer
case $answer in
Y | y)  echo
        echo "OK,Synchronize the Cluster now,please waiting..." ;;
N | n)  echo
        echo "OK,Please synchronize manually if necessary!"
        exit ;;
esac
```
但是在AIX的ksh中，没有-n等，使用示例如下：
```sh
#!/bin/ksh
read answer?"Need to synchronize the Cluster?[Y/N] " 
case $answer in
Y | y)  echo
        echo "OK,Synchronize the Cluster now,please waiting..." ;;
N | n)  echo
        echo "OK,Please synchronize manually if necessary!"
        exit ;;
esac
```
### 计数方法差异（应该是没有方括号计算方法）
在RHEL中使用如下方法进行计数：
```sh
#!/bin/bash
count=0
for i in 1 2 3
do
        count=$[ $count + 1 ]
done
echo $count
```
运行后示例如下：
```
[root@redhat8 test]# sh test2.sh
3
```
在AIX 7.1.4.3中运行（使用bash一样不行）：
```sh
#!/bin/ksh
count=0
for i in 1 2 3
do
        count=$[ $count + 1 ]
done
echo $count
```
运行后示例如下：
```
# sh test9.sh
test9.sh[5]: 0:  not found.
test9.sh[5]: 0:  not found.
test9.sh[5]: 0:  not found.
0
```
可以使用双括号的方法进行解决：
```sh
#!/bin/ksh
count=0
for i in 1 2 3
do
        count=$(($count + 1))
done
echo $count
```
运行后示例如下：
```
# sh test9.sh
3
```
### 其它差异
AIX7.1.4.3中没有mktemp命令，没有lsof命令，可以单独安装。