# Python运维学习笔记-Linux系统管理
学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## 文件读写
&#8195;&#8195;文件可以进行多维度管理，例如重命名文件、获取文件属性、判断是否存在、备份文件及读写文件等。此小节主要学习文件的读写操作。
### Python内置的open函数
&#8195;&#8195;Python内置的open函数可以打开文件，函授接受文件名和打开模式作为参数，返回一个文件对象，用户通过文件对象来操作文件，完成后使用close方法关闭即可。使用示例如下：
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
&#8195;&#8195;Python标准库的os模块对操作系统的API进行了封装，并使用统一的API访问不同的操作系统。os模块包含与操作系统的系统环境、文件系统、用户数据库以及权限进行交互的函数。
### 使用os.path进行路径和文件管理
&#8195;&#8195;os模块下getcwd函数和listdir函数，getcwd函数用来获取当前目录，listdir函数用来列出目录下的所有文件和文件夹，示例如下：
```
>>> os.getcwd()
'/python/jinja2'
>>> os.listdir('.')
['simple.py', '.simple.html.swp', 'simple.html', 'index.html', 'extend.py', 'base.html', '
test1.txt', 'test.txt', 'test2.txt', 'test3.txt', 'captialize.py']
```
#### 拆分路径
&#8195;&#8195;os.path模块用来对文件和路径进行管理，它包含很多拆分路径的函数，相关函数有：
- split：返回一个二元组，包含文件的路径与文件名
- dirname：返回文件的路径
- basename：返回文件的文件名
- splitext：返回一个除去文件扩展名的部分和扩展名的二元组

这几个函数功能示例如下：
```
>>> import os
>>> path = '/python/jinja2/test.txt'
>>> os.path.dirname(path)
'/python/jinja2'
>>> os.path.split(path)
('/python/jinja2', 'test.txt')
>>> os.path.basename(path)
'test.txt'
>>> os.path.splitext(path)
('/python/jinja2/test', '.txt')
```
#### 构建路径
&#8195;&#8195;os.path也包含了用以构建路径的函数，最常用的是expanduser、abspath和join函数：
- expanduser：展开用户的HOME目录，如~、~username
- abspath：得到文件或路径的绝对路径
- join：根据不同的系统平台，使用不同的路径分隔符拼接路径

这几个函数功能示例如下：
```
>>> import os
>>> os.path.expanduser('~')
'/root'
>>> os.path.expanduser('~huang')
'/home/huang'
>>> os.path.abspath('.')
'/python/jinja2'
>>> os.path.abspath('./test.txt')
'/python/jinja2/test.txt'
>>> os.path.join('~','test','test.py')
'~/test/test.py'
>>> os.path.join(os.path.expanduser('~huang'),'test','test.py')
'/home/huang/test/test.py'
```
os.path模块中isabs函数用来检查一个路径是否为绝对路径：
```
>>> os.path.isabs('/python/jinja2/test.txt')
True
>>> os.path.isabs('.')
False
```
&#8195;&#8195;在Python中，可以使用`__file__`这个特殊的遍历表示当前代码所在的源文件。在编写代码时，有时需要导入当前源文件父目录下的软件包，代码如下：
```python
#!/usr/bin/python3
#_*_ coding: UTF-8 _*_
import os
print('Current directory:',os.getcwd()) 
path = os.path.abspath(__file__)
print('Full path of current file:',path)
print('Parent directory of current file:',
os.path.abspath(os.path.join(os.path.dirname(path),os.path.pardir)))
```
运行后示例：
```
[root@redhat8 jinja2]# python3 test.py
Current directory: /python/jinja2
Full path of current file: /python/jinja2/test.py
Parent directory of current file: /python
```
#### 获取文件属性
&#8195;&#8195;os.path模块也包含了若干函数用来获取文件的属性，包括文件的创建时间、修改时间、文件大小等：
- getatime：获取文件的访问时间
- getmtime：获取文件的修改时间
- getctime：获取文件的创建时间
- getsize：获取文件的大小

#### 判断文件类型
&#8195;&#8195;os.path模块中有若干函数来判断路径是否存在，以及路径所指文件的类型，这类函数一般以`is`开头，并返回一个Boolean型结果：
- exists：参数path所指向的路径是否存在
- isfile：参数path所指向的路径存在，并且是一个文件
- isdir：参数path所指向的路径存在，并且是一个文件夹
- islink：参数path所指向的路径存在，并且是一个链接
- ismount：参数path所指向的路径存在，并且是一个挂载点

获取当前用户home目录下的所有文件列表：
```python
[item for item in os.listdir(os.path.expanduser('~')) if os.path.isfile(item)]
```
获取当前用户home目录下的所有目录列表：
```python
[item for item in os.listdir(os.path.expanduser('~')) if os.path.isdir(item)]
```
获取当前用户home目录下的所有目录的目录名到绝对路径之间的字典：
```python
{item: os.path.realpath(item) for item in os.listdir(os.path.expanduser('~')) if os.path.isdir(item)}
```
获取当前用户home目录下的所有文件到文件大小之间的字典：
```python
{item: os.path.getsize(item) for item in os.listdir(os.path.expanduser('~')) if os.path.isfile(item)}
```
### 使用os模块管理文件和目录
os模块的文件和目录的操作函数：
- chdir：修改当前目录
- unlink/remome：删除path路径所指向的文件
- rmdir：删除path路径所指向的文件夹，该文件夹必须为空，否则报错
- mkdir：创建一个文件夹
- rename：重命名文件或文件夹

使用示例：
```
>>> import os
>>> os.getcwd()
'/python/jinja2'
>>> os.chdir(os.path.expanduser('~'))
>>> os.getcwd()
'/root'
[root@redhat8 test]# ls
11.png  12.png  test1  test1.py  test2  test2.py  test.py
>>> os.remove('11.png')
>>> os.unlink('12.png')
>>> os.rmdir('test1')
>>> os.removedirs('test2')
[root@redhat8 test]# ls
test1.py  test2.py  test.py
>>> os.mkdir('test3')
>>> os.rename('test2.py','test3.py')
[root@redhat8 test]# ls
test1.py  test3  test3.py  test.py
```
&#8195;&#8195;os模块也包含了文件权限、判断文件权限的函数，即chmod和access，用三个常量来表示读、写、可执行权限，即R_OK、W_OK、X_OK，示例如下：
```python
#!/usr/bin/python
#_*_ coding: UTF-8 _*_
import os
import sys
def main():
    sys.argv.append('')
    filename = sys.argv[1]
    if not os.path.isfile(filename):
        raise SystemExit(filename + ' does not exists!')
    elif not os.path.access(filename,os.R_OK):
        os.chmod(filename,0777)
    else:
        with open(filename) as f:
            print(f.read())
if __name__ == '__main__':
    main()
```
示例说明：
- 示例中首先通过命令行读取文件的名称，先判断文件是否存在，如果不存在就直接退出
- 然后判断文件是否具有读权限，如果没有，则将文件赋予`777`权限
- 如果文件存在并且具有读权限，则读取文件内容

### 案例：打印最常用的10条Linux命令
为方便查阅，收录在[Python运维-Linux系统管理实例](https://ebook.big1000.com/14-Python%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E4%B8%8E%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/01-Python%E8%BF%90%E7%BB%B4-%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/05-Python%E8%BF%90%E7%BB%B4-Linux%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E5%AE%9E%E4%BE%8B.html)

