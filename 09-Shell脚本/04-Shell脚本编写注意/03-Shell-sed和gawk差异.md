# Shell-sed和gawk差异
&#8195;&#8195;在AIX中sed使用和RHEL中基本一致，但是使用过程中还是发现不少差异，差异也可能是我使用的AIX版本比较老了，不管怎样，记录下来，毕竟AIX7.1版本使用的还是比较多。在AIX7.1中没有gawk，有awk使用差异也记录下来。
## sed的差别
在编写修改image.data脚本过程中发现不少区别，记录下来避免忘记。
### 地址使用差别
例如下面同样的文本：
```
bash-5.0# cat test1
        PP= 2
        PP= 8
        PP= 2
        PP= 8
        PP= 42
```
在AIX7.1.4.3中使用报错无法解析：
```
bash-5.0# cat test1 | awk '/PP=/{print $0}' | sed '4,/PP=/{s/42/1/}'
sed: 0602-404 Function 4,/PP=/{s/42/1/} cannot be parsed.
bash-5.0# exit
# cd /tmp
# cat test1 | awk '/PP=/{print $0}' | sed '4,/PP=/{s/42/1/}'
sed: 0602-404 Function 4,/PP=/{s/42/1/} cannot be parsed.
```
在RHEL8.0中运行示例如下（也可以使用gawk）：
```
[root@redhat8 test]# cat test1 | awk '/PP=/{print $0}' | sed '4,/PP=/{s/42/1/}'
        PP= 2
        PP= 8
        PP= 2
        PP= 8
        PP= 1
```
换了一个方式，在AIX7.1.4.3中多行可以使用（解决方法目前我发现也就是这种了），使用单行不能解析：
```
# sed '2{
> s/PP=/ls/
> }' test1
        PP= 2
        ls 8
        PP= 2
        PP= 8
        PP= 42
# sed '2{s/PP=/ls/}' test1
sed: 0602-404 Function 2{s/PP=/ls/} cannot be parsed.
bash-5.0# sed '2{s/PP=/ls/}' test1
sed: 0602-404 Function 2{s/PP=/ls/} cannot be parsed.
```
### sed中选项差异
&#8195;&#8195;sed编辑器一般不会对原文本进行修改，当然有w命令可以将输出保存到某一文件，有时候直接想修改原文件，选项`-i`就可以起到这个作用，但是`-i`只是最近才更新的功能，RHEL8中是有的，但是在AIX7.1.4.3（AIX系统更新到7.2.5版本了，不知道后面有没有）中sed编辑器没有-i选项，替换文件并直接保存在RHEL中两种方法示例如下：
```
[root@redhat8 test]# sed -i 's/PP=/ls/' test1
[root@redhat8 test]# cat test1
        ls 2
        ls 8
        ls 2
        ls 8
        ls 42
[root@redhat8 test]# sed -in-place -e 's/ls/PP=/' test1
[root@redhat8 test]# cat test1
        PP= 2
        PP= 8
        PP= 2
        PP= 8
        PP= 42
```
在AIX7.1.4.3中运行示例如下：
```
bash-5.0# sed -i 's/PP=/ls/' test1
sed: Not a recognized flag: i
Usage:  sed [-n] [-u] Script [File ...]
        sed [-n] [-u] [-e Script] ... [-f Script_file] ... [File ...]
bash-5.0# sed -in-place -e 's/PP=/ls/' test1
sed: Not a recognized flag: i
Usage:  sed [-n] [-u] Script [File ...]
        sed [-n] [-u] [-e Script] ... [-f Script_file] ... [File ...]
```
解决方法：将输出重定向到一个文件中，然后覆盖掉原文件：
```
bash-5.0# sed 's/PP=/ls/' test1 >test1.tmp
bash-5.0# mv test1.tmp test1
bash-5.0# cat test1
        ls 2
        ls 8
        ls 2
        ls 8
        ls 42
```

### 列数据转换成行差异
RHEL8中把一列数据转换成一行：
```
[root@redhat8 test]# cat test2 | awk '/LPs=/{print $2}' | sed ':a;N;s/\n/ /g;ta'
1 4 1 4 21 4 24 1 8 1 2 2
```
在AIX7.1.4.3中使用示例:
```
# cat image.data | awk '/LPs=/{print $2}' | sed ':a;N;s/\n/ /g;ta' 
sed: 0602-417 The label :a;N;s/\n/ /g;ta is greater than eight characters.# 
bash-5.0# cat image.data | awk '/LPs=/{print $2}' | sed ':a;N;s/\n/ /g;ta'
sed: 0602-417 The label :a;N;s/\n/ /g;ta is greater than eight characters.bash-5.0# 
```
使用tr命令可以解决，示例如下：
```
# cat image.data | awk '/LPs=/{print $2}' | tr "\n" " "
1 4 1 4 21 4 24 1 8 1 2 2 # 
bash-5.0# cat image.data | awk '/LPs=/{print $2}' | tr "\n" " "
1 4 1 4 21 4 24 1 8 1 2 2 bash-5.0# 
```
RHEL8中也可以，示例如下：
```
[root@redhat8 test]# cat test2 | awk '/LPs=/{print $2}' | tr "\n" " "
1 4 1 4 21 4 24 1 8 1 2 2 [root@redhat8 test]# 
```
### next命令差异
&#8195;&#8195;在RHEL中，学习next命令时候，小写的n命令会告诉sed编辑器移动到数据流中的下一行文本，具体可以参考[Shell笔记-sed编辑器](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/01-Shell%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/09-Shell%E7%AC%94%E8%AE%B0-sed%E7%BC%96%E8%BE%91%E5%99%A8.html)中的内容，但是在AIX7.1.4.3中，同样命令使用不行，会报错：
```
bash-5.0# cat test3
Miracles happen every day !

To make each day count !

Stupid is as stupid does !
bash-5.0# sed '/happen/{n;d}' test3
sed: 0602-404 Function /happen/{n;d} cannot be parsed.
```
&#8195;&#8195;没有有效解决方法，但是在实例总找到了替代方案。实际应用中是想在iostat命令输出中，输出adapter下面的一行，取适配器的编号，但是用上述方法会报错：
```
bash-5.0# iostat -a |sed '/Vadapter:/{n;p}'
sed: 0602-404 Function /Vadapter:/{n;p} cannot be parsed.
```
使用的替代方案：
```sh
bash-5.0# iostat -a |awk '{RS=""}{
> if ($1 == "Vadapter:" || $1 == "Adapter:")
> print $0}'|sed '/dapter:/d'
vscsi2                      0.0      0.7        0.0       0.7            1
vscsi3                      0.0      0.0        0.0       0.0            2
```
## gawk差异
&#8195;&#8195;AIX7.1.4.3中默认没有gawk，有awk程序，使用方法差不多，目前发现awk中没用`-mr`选项，使用会报错，应该也没有`-mr`和`-W`选项：
```
# lparstat -i |awk -mr 6 '/Entitled Capacity/{print $4}'
awk: Not a recognized flag: m
Usage: awk [-u] [-F Character][-v Variable=Value][-f File|Commands][Variable=Value|File ..
.]
```
