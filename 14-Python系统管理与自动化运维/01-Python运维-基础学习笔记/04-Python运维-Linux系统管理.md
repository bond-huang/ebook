# Python运维学习笔记-Linux系统管理
学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## 文件读写
&#8195;&#8195;文件可以进行多维度管理，例如重命名文件、获取文件属性、判断是否存在、备份文件及读写文件等。此小节主要学习文件的读写操作。
### Python内置的open函数
&#8195;&#8195;ython内置的open函数可以打开文件，函授接受文件名和打开模式作为参数，返回一个文件对象，用户通过文件对象来操作文件，完成后使用close方法关闭即可。使用示例如下：
```
>>> a = open('extend.py')
>>> print(a.read())
import os
import jinja2
def render(tpl_path,**kwargs):
    path,falename = os.path.split(tpl_path)
    return jinja2.Environment(
        loader=jinja2.FileSystemLoader('./')
    ).get_template('index.html').render(**kwargs)

def test_extend():
    result = render('index.html')
    print(result)
if __name__ == '__main__':
    test_extend()
>>> a.close()
>>> 
```
open函数默认以'r'模式打开，也可以指定文件的打开模式，如下表所示：

模式|含义
:---|:---
'r'|默认以读模式打开文件，如果文件不存在，抛出FileNotFoundError异常
'w'|以写模式打开文件，如果文件非空，则文件已有的内容会被清空；如果文件不存在则创建文件
'x'|创建一个新的文件，如果文件已经存在，抛出FileExistsError异常
'a'|在文件末尾追加文件

演示示例如下：
```
>>> a = open('test.txt','w')
>>> 
[root@redhat8 jinja2]# ls
base.html  extend.py  index.html  simple.html  simple.py  test.txt
>>> a = open('test.txt','w')
>>> a.write('Miracles happen evary day!')
26
>>> a.close()
>>> a = open('test.txt','x')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileExistsError: [Errno 17] File exists: 'test.txt'
>>> a = open('test1.txt','x')
>>> a.write('Miracles happen evary day!')
26
>>> a.close()
[root@redhat8 jinja2]# cat test1.txt
Miracles happen evary day!
```
### 避免文件句柄泄露
&#8195;&#8195;计算机程序中，每打开一个文件就需要占用一个文件句柄，一个进程拥有的文件句柄数是有限的。Python中使用finally语句来关闭文件句柄，任何情况下都会被关闭，示例如下：
```python
try:
    f = open('test.txt')
    print(f.read())
finally:
    f.close()
```
&#8195;&#8195;Python中有更简洁优美的写法，即使用上下文管理器，使用上下文管理器处理文件打开、处理在再关闭的代码如下：
```python
with open('test.txt') as f:
    print(f(read))
```
### 常见文件操作函数
&#8195;&#8195;Python的文件对象有多种类型的函数，刷新缓存的flush函数，获取文件位置的tell函数，改变文件读取偏移量的seek函数。使用比较多的是读写类函数：
- read：读取文件中的所有内容
- readline：一次读取一行
- readlines：将文件内容存到一个列表中，列表中的每一行对应于文件中的一行
- write：写字符串到文件中，并返回写入的字符数
- writelines：写一个字符串列表到文件中

读函数示例：
```
>>> f = open('test.txt')
>>> f.read()
'Miracles happen evary day!\nI was messed up for a long time.\n'
>>> f.seek(0)
0
>>> f.readline()
'Miracles happen evary day!\n'
>>> f.seek(0)
0
>>> f.readlines()
['Miracles happen evary day!\n', 'I was messed up for a long time.\n']
```
注意：read和readlines函数都是一次将所有内容读入到内存中，如果处理大文件，可能为出现Out Of Memory错误。    
写函数示例：
```
>>> f = open('test2.txt','w')
>>> f.write('Stupid is as stupid does.')
25
>>> f.writelines(['Miracles happen evary day!\n', 'I was messed up for a long time.\n'])
[root@redhat8 jinja2]# cat test2.txt
Stupid is as stupid does.Miracles happen evary day!
I was messed up for a long time.
```
&#8195;&#8195;Python中还可以使用print函数将结果输出到文件中，比write和writelines函数更加灵活，示例如下：
```
with open('test.txt','w') as f:
    print(1,2,'You cannot change the past',seq",",file=f)
```
### Python的文件是一个可迭代对象
&#8195;&#8195;在Python中，Python的文件对象实现了迭代器协议，for循环可以使用迭代器协议遍历可迭代对象，所以可以使用for循环遍历文件，用来依次处理文件的内容，for循环遍历文件内容的代码如下：
```python
with open('test.txt') as inf:
    for line in inf:
        print(line.upper())
```
### 案例
将文件中所有的单词的首字母变成大写。为方便查阅，收录在[Python运维-Linux系统管理实例](https://ebook.big1000.com/14-Python%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E4%B8%8E%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/01-Python%E8%BF%90%E7%BB%B4-%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/05-Python%E8%BF%90%E7%BB%B4-Linux%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E5%AE%9E%E4%BE%8B.html)

## 文件与文件路径管理

