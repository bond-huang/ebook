# Python运维学习笔记-Linux系统管理实例
学习过程中摘自教材：《Python Linux系统管理与自动化运维》（赖明星著）中的实例。
## 文件读写实例
### 案例：将文件中所有的单词的首字母变成大写
代码如下：
```python
with open('test.txt') as inf,open('test3.txt','w') as outf:
    for line in inf:
        outf.write(' '.join([word.capitalize() for word in line.split()]))
        outf.write("\n")
```
运行后示例如下：
```
[root@redhat8 jinja2]# python3 captialize.py
[root@redhat8 jinja2]# cat test3.txt
Miracles Happen Evary Day!
I Was Messed Up For A Long Time.
```
示例说明：
- 示例中，使用上下文管理器同时管理了两个文件，以读模式打开文件，然后以写模式打开输出文件
- 对于输入文件，使用for循环依次遍历文件的每一行，然后使用字符串处理函数split来拆分这一行中的单词
- 然后使用capitalize函数将单词的首字母转换为大写
- 最后使用`''.join()`（注意示例中的空格）将各个单词链接起来，并写入到输出文件中

上面示例也可以使用print函数来简化输出语句，如下所示：
```Python
with open('test.txt') as inf,open('test3.txt','w') as outf:
    for line in inf:
        print(*[word.capitalize() for word in line.split()],file=outf)
```
## 文件与文件路径管理
### 案例：打印最常用的Linux命令
&#8195;&#8195;在Linux系统中，`~/.bash_history`文件保存了命令的历史，可以读取此文件进行统计，统计时只统计命令的名称，同一个命令不同参数的也算同一个命令，下面是找出出现次数最多的五条命令：
```python
import os
from collections import Counter
c = Counter()
with open(os.path.expanduser('~/.bash_history')) as f:
    for line in f:
        cmd = line.strip().split()
        if cmd:
            c[cmd[0]] += 1
print(c.most_common(5))
```
运行后示例如下：
```
[root@redhat8 test]# python3 test.py
[('ls', 171), ('vi', 95), ('cd', 87), ('python3', 71), ('cat', 67)]
```
## 查找文件
