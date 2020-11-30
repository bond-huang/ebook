# Python运维学习笔记-打造命令行工具
记录一些基础知识学习笔记。学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## 与命令行相关的Python语言特性
### 使用sys.argv获取命令行参数
&#8195;&#8195;在Python中，sys库下有一个名为argv的列表，保存了所有的命令行参数。例如下面的Python文件导入sys库，然后打印argv列表中的内容：
```python
from __future__ import print_function
import sys
print(sys.argv)
```
对上面文件进行运行测试：
```
[root@redhat8 python]# python3 test_argv.py
['test_argv.py']
[root@redhat8 python]# python3 test_argv.py localhost 9527
['test_argv.py', 'localhost', '9527']
```
说明：
- 如果不传递任何参数，则`sys.argv`有且仅有一个元素，即Python程序名字
- 如果传递其它命令行参数时，所有的参数都以字符串的形式保存到`sys.argv`中
- `from __future__ import print_function`在Python2的环境是使用Python3语句,我使用环境就是Python3，后续都不会加了，主要是print用法的区别

&#8195;&#8195;`sys.argv`是一个保存命令行参数的普通列表，可以直接修改`sys.argv`的内容，示例如下：
```py
import os
import sys
def main():
    sys.argv.append("")
    filename = sys.argv[1]
    if not os.path.isfile(filename):
        raise SystemExit(filename + ' does not exists')
    elif not os.access(filename, os.R_OK)
        raise SystemExit(filename + 'is not accessible')
    else:
        print(filename + ' is accessible')
if __name__ == '__main__'
    main()
```
示例说明：
- 示例中，从命令行参数获取文件的名称，然后判断文件是否存在
    - 如果文件不存在，则提示用户该文件不存在
    - 如果文件存在，则使用os.access函数判断是否具有对文件的读权限
- 示例中通过`sys.argv[1]`获取文件的名称
    - 如果用户直接运行程序不传递任何命令行参数，会出现索引越界的错误
    - 避免此错误在访问`sys.argv`之前向其添加了一个空字符串`sys.argv.append("")`
    - 无论用户是否提供了命令行参数，在添加空字符串之后，访问`sys.argv[1]`都不会报错
    - 如果用户传递了命令行参数，`sys.argv[1]`得到的是用户提供的命令行参数

### 使用sys.stdin和fileinput读取标准输入
#### 使用sys.stdin
&#8195;&#8195;在Python标准库的sys库中，有stdin（标准输入）、stdout（标准输出）和stderr（错误输出）三个文件描述符。不需要调用open函数就可以直接使用，例如下面的read_stdin.py文件从标准输入中读取内容，然后打印到命令行终端：
```py
import sys
for line in sys.stdin:
    print(line,end="")
```
像shell脚本一样，通过标准输入给该程序输入内容：
```
root@redhat8 python]# cat /etc/passwd |python3 read_stdin.py
root:x:0:0:root:/root:/bin/bash
...
[root@redhat8 python]# python3 read_stdin.py < /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
[root@redhat8 python]# python3 read_stdin.py -

```
&#8195;&#8195;可以使用`sys.stdin`调用文件对象的方法，如调用read函数读取标准输入中的所有内容，如下示例调用readlines函数将标准输入的内容读取到一个列表中：
```py
import sys
def get_content():
    return sys.stdin.readlines()
print(get_content())
```
#### 使用fileinput
&#8195;&#8195;在Linux下，可以使用Python语言替代awk进行数据处理，awk对多文件处理提供了支持，在Python中可以使用fileinput进行多文件处理，可以依次读取命令行参数中给出的多个文件，fileinput会遍历`sys.argv[1:]`列表,如果列表为空，默认读取标准输入中的内容。例如文件read_from_fileinput.py：
```py
#!/usr/bin/python3
import fileinput
for line in fileinput.input():
    print(line,end="")
```
示例说明：
- 示例中直接调用了fileinput模块的input方法按行读取内容
- 示例中先导入了fileinput模块，然后在for循环中遍历文件的内容

fileinput既可以从标准输入中读取数据，也可以从文件中读取数据，示例：
```
[root@redhat8 python]# cat /etc/passwd |python3 read_from_fileinput.py
root:x:0:0:root:/root:/bin/bash
...
[root@redhat8 python]# python3 read_from_fileinput.py < /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
[root@redhat8 python]# python3 read_from_fileinput.py /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
[root@redhat8 python]# python3 read_from_fileinput.py /etc/passwd /etc/hosts
root:x:0:0:root:/root:/bin/bash
...
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```
fileinput提供了一些方法告知当前所读取的内容属于哪一个文件：
- filename：当前正在读取的文件名
- fileno：文件的描述符
- fileineno：正在读取的行是当前文件的第几行
- isfirstlin：正在读取的行是否当前文件的第一行
- isstdin fileinput：正在读取文件还是直接从标准输入读取内容

这些方法的使用示例：
```py
#!/usr/bin/python3
import fileinput
for line in fileinput.input():
    mate = [fileinput.filename(),fileinput.fileno(),fileinput.filelineno(),fileinput.isfirstlin(),fileinput.isstdin()]
    print(*mate.end="")
    print(*line.end="")
```
### 使用SystemExit异常打印错误信息
&#8195;&#8195;`sys.stdout`与`sys.stderr`使用方法与sys.stdin类似，下面是示例分别使用`sys.stdout`与`sys.stderr`输出内容，文件名为test_stdout_stderr.py:
```py
import sys
sys.stdout.write('Captain')
sys.stderr.write('American')
```
&#8195;&#8195;通过重定向来验证'Captain'被输出到了标准输出，'American'被输出到了错误输出，示例如下：
```
[root@redhat8 python]# python3 test_stdout_stderr.py > /dev/null
American
[root@redhat8 python]# python3 test_stdout_stderr.py 2> /dev/null
Captain
```
&#8195;&#8195;一般情况下，不会直接用`sys.stdout`来输出内容，如果Python程序执行是被，需要在标准错误中输出错误信息，然后以非零的返回码退出程序，示例：
```py
import sys
sys.stderr.write('error message')
sys.exit(1)
```
也可以直接爆出一个SystemExit异常，示例如下：
```
[root@redhat8 python]# cat test_system_exit.py
raise SystemExit("Error Message")
[root@redhat8 python]# python3 test_system_exit.py
Error Message
[root@redhat8 python]# echo $?
1
```
### 使用getpass库读取密码
&#8195;&#8195;`getpass`主要包含`getuser`函数和`getpass`函数，前者用于从环境变量中获取用户名，后者用于等待用户输入密码，和`input`区别在于不会将输入显示在命令行中，使用示例：
```py
import getpass
user = getpass.getuser('Please input your username: ')
passwd = getpass.getpass('Please input your passwor: ')
print(user,passwd)
```
## 使用ConfigParser解析配置文件
&#8195;&#8195;一个典型的配置文件包含一到多个（section），每个章节下面可以包含一个或多个选项（option），里面下面的MySQL配置文件：
```
[client]
port        = 3306
user        = mysql
password    = mysql
host        = 127.0.0.1

[mysqld]
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /tmp
skip-external-locking
```
&#8195;&#8195;在Python3中，ConfigParse模块重命名为configparser，使用有细微差异。要解析一个配置文件，需先创建一个ConfigParse对象（名称自定义），然后用read方法读取配置文件，也可以使用readfp从一个已经打开的文件中读取配置：
```python
import configparser
cf = configparser.ConfigParser(allow_no_value=True)
cf.read('my.cnf')
```
&#8195;&#8195;参数`allow_no_value`默认取值是`False`，表示配置文件中是否允许选项没有值的情况，之前配置文件中`skip-external-locking`选项只有名称没有选区制，所有需指定`allow_no_value`值为`True`。ConfigParser中有很多方法，如下：
- sections：返回一个包含所有章节的列表
- has_section：判断章节是否存在
- items：以元组的形式返回所有选项
- options：返回一个包含章节下所有选项的列表
- get,getboolean,gitint,getfloat：获取选项的值
- remove_section：删除一个章节
- add_setcion：添加一个章节
- remote_option：删除一个选项
- set：添加一个选项
- write将ConfigParser对象中的数据保存到文件中

以之前的MySQL配置文件为例，测试各种方法的使用：
```python
import configparser
cf = configparser.ConfigParser(allow_no_value=True)
cf.read('my.cnf')
print(cf.get('client', 'host'))
print(cf.sections())
print(cf.has_section('client'))
cf.remove_option('mysqld','tmpdir')
cf.remove_section('client')
cf.add_section('mysql')
cf.set('mysql','host','127.0.0.1')
cf.write(open('new_my.cnf','w'))
```
运行示例如下：
```
[root@redhat8 python]# python3 configparse_test.py
127.0.0.1
['client', 'mysqld']
True
[root@redhat8 python]# cat new_my.cnf
[mysqld]
basedir = /usr
datadir = /var/lib/mysql
skip-external-locking

[mysql]
host = 127.0.0.1
```
## 使用argparse解析命令行参数
&#8195;&#8195;在Python中，argparse是标准库中用来解析命令行参数的模块，能够根据程序中的定义从sys.argv中解析参数，并自动生成帮助和使用信息。
### ArgumentParse解析器
