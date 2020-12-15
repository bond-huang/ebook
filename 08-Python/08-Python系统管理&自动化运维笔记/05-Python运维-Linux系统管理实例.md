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
### 案例：找到目录下最大（或最老）的十个文件
&#8195;&#8195;查找某个目录及子目录下最大的十个文件，或者最老，或者包含某个名字的文件，这些需求有一个共同需求，即找到某个目录及其子目录下的某种文件，更加通用需求是，找到某个目录树中，除部分特殊目录外，其它目录中的某一些文件。可以用函数先实现这个通用需求，然后调用这个函数实现其它需求，函数代码如下：
```python
#!/usr/bin/python
#_*_ coding: UTF-8 _*_
import os
import fnmatch
def is_file_match(filename,patterns):
    for pattern in patterns:
        if fnmatch.fnmatch(filename,pattern):
            return True
    return False
def find_specific_files(root,patterns=['*'],exclude_dirs=[]):
    for root,dirnames,filenames in os.walk(root):
        for filename in filenames:
            if is_file_match(filename,patterns):
                yield os.path.join(root,filename)
        for d in exclude_dirs:
            if d in dirnames:
                dirnames.remove(d)
```
&#8195;&#8195;上面代码中定义了find_specific_files函数，接受三个参数，分别是查找根路径，匹配的文件模式列表和需要排除的目录列表，匹配模式列表和排除的目录列表都有默认值（默认 情况下找到根路径下的所有文件）。    
查找目录下的所有文件：
```python
for item in find_specific_files("."):
    print(item)
```
查找目录下所有图片：
```python
patterns=['*.jpg','*.png','*.jpeg','*.tif','*.tiff','*.gif']
for item in find_specific_files(".",patterns):
    print(item)
```
查找目录树下，除test目录宜为其它目录下所有图片：
```python
patterns=['*.jpg','*.png','*.jpeg','*.tif','*.tiff','*.gif']
exclude_dirs=['test']
for item in find_specific_files(".",patterns,exclude_dirs):
    print(item)
```
找到某个目录极其子目录下最大的十个文件：
```python
files = {name: os.path.getsize(name) for name in find_specific_files('.')}
result = sorted(files.items(),key=lambda d: d[1],reverse=True)[:10]
for i , t in enumerate(result,1):
    print(i,t[0],t[1])
```
示例说明：
- 示例中首先通过字典推导创建了一个字典，字典的key是找到的文件，字典的value是文件的大小
- 构建出字典后，使用Python内置的sorted函数对字典进行逆序排序，排序完成后即可获得最大的十个文件

也可以指定参数排除`.git`目录：
```python
files = {name: os.path.getsize(name) for name in find_specific_files('.'),exclude_dirs=['.git']}
result = sorted(files.items(),key=lambda d: d[1],reverse=True)[:10]
for i , t in enumerate(result,1):
    print(i,t[0],t[1])
```
找到某个目录及其子目录下最老的十个文件：
```python
files = {name: os.path.gettime(name) for name in find_specific_files('.')}
result = sorted(files.items(),key=lambda d: d[1])[:10]
for i , t in enumerate(result,1):
    print(i,t[0],time.ctime(t[1]))
```
找到某个目录及其子目录下，所有文件名中包含“test”的文件：
```python
files = [name for name in find_specific_files('.',patternas=['*test*'])]
for i , name in enumerate(files,1):
    print(i,name)
```
示例说明：
- 示例中，除了传递目录外，还传递了相应的匹配模式
- 为了支持多种匹配模式，模式匹配这个参数以列表的形式表示

找到某个目录及子目录下，排除`.git`子目录以后所有Python源文件：
```python
files = [name for name in find_specific_files('.',patternas=['*.py'],exclude_dirs=['.git'])]
for i , name in enumerate(files,1):
    print(i,name)
```
删除某个目录及其子目录下的所有pyc文件：
```python
files = [name for name in find_specific_files('.',patternas=['*.pyc'])]
for name in files:
    os.remove(name)
```
## 文件内容管理
