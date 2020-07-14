# Python-操作系统接口

### os_多种操作系统接口
在 Python 中，使用字符串类型表示文件名、命令行参数和环境变量。 在某些系统上，在将这些字符串传递给操作系统之前，必须将这些字符串解码为字节。

os 模块提供了许多与操作系统交互的函数,收录一些常用的。
##### os.getcwd & os.chdir
os.getcwd是获取当前工作目录，用法：`os.getcwd()`
os.chdir是更改当前工作目录，用法：`os.chdir(path)`
示例如下：
```python
>>> import os
>>> os.getcwd()
'/tmp/rpm/python'
>>> os.chdir('/tmp/rpm')
>>> os.getcwd()
'/tmp/rpm'
```
##### os.system
在子shell中执行命令（字符串）。这是调用标准C函数system()来实现的，因此限制条件与该函数相同，用法：`os.system(command)`
执行命令后并返回一个结果，示例如下：
```python
>>> import os
>>> os.system('mkdir test')
0
>>> os.system('pwd')
/tmp/rpm
0
```
##### os.popen
格式如下：
```python
os.popen(cmd, mode='r', buffering=-1)
```
打开一个管道，它通往 / 接受自命令：cmd。返回值是连接到管道的文件对象；根据 mode 是 'r' （默认）还是 'w' 决定该对象可以读取还是写入。buffering 参数与内置函数 open() 相应的参数含义相同。返回的文件对象只能读写文本字符串，不能是字节类型。
示例如下：
```python
>>> import os
>>> a = os.popen('pwd',mode='r',buffering=-1)
>>> print(a)
<os._wrap_close object at 0xa000000001524c8>
>>> print(a.read())
/tmp/rpm/python

>>> 
```
本方法是使用 subprocess.Popen 实现的，可以参阅该类的文档。

### subprocess_子进程管理
subprocess 模块允许生成新的进程，连接它们的输入、输出、错误管道，并且获取它们的返回码。此模块打算代替一些老旧的模块与功能：`os.system`及`os.spawn*`等
替代`os.system`如下：
```python
sts = os.system("mycmd" + " myarg")
# becomes
sts = subprocess.call("mycmd" + " myarg", shell=True)
```
##### run()函数
run() 函数是在 Python 3.5 中新增的。
格式如下：
```python
subprocess.run(args, *, stdin=None, input=None, stdout=None,stderr=None,capture_output=False, shell=False, cwd=None,timeout=None,check=False, encoding=None, errors=None,text=None,env=None, universal_newlines=None, **other_popen_kwargs)
```
功能：运行被 arg 描述的指令。等待指令完成, 然后返回一个 CompletedProcess 实例.
简单示例如下：
```python
>>> import subprocess
>>> subprocess.run('ls -l',shell=True)
>>> subprocess.run('ls -l',shell=True)
total 100528
-rw-r-----    1 root     system         1477 Jul 13 19:58 aix_reboot_check.py
-rw-r-----    1 root     system       243981 Jul 13 07:28 bzip2-1.0.8-2.aix6.1.ppc.rpm
-rw-r-----    1 root     system       284219 Jul 13 07:28 gdbm-1.18.1-1.aix6.1.ppc.rpm
-rw-r-----    1 root     system     40329593 Jul 13 07:14 python3-3.7.6-1.aix6.1.ppc.rpm
-rw-r-----    1 root     system       233022 Jul 13 11:17 python3-devel-3.7.6-1.aix6.1.ppc
.rpm-rw-r-----    1 root     system        13392 Jul 13 07:55 python3-tools-3.7.6-1.aix6.1.ppc
.rpm-rw-r-----    1 root     system      2525190 Jul 13 07:28 readline-8.0-2.aix6.1.ppc.rpm
-rw-r-----    1 root     system      7822627 Jul 13 07:28 sqlite-3.28.0-1.aix6.1.ppc.rpm
CompletedProcess(args='ls -l', returncode=0)
```
class subprocess.CompletedProcess是run()的返回值, 代表一个进程已经结束.

##### Popen 构造函数
格式如下：
```python
class subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None,preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=None,startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *,encoding=None, errors=None, text=None)
```
在POSIX中，如果 args 是一个字符串，此字符串被作为将被执行的程序的命名或路径解释。但是，只有在不传递任何参数给程序的情况下才能这么做。
在POSIX中，当 shell=True，shell 默认为 /bin/sh。如果args是一个字符串，此字符串指定将通过shell执行的命令。这意味着字符串的格式必须和在命令提示符中所输入的完全相同。这包括，例如，引号和反斜杠转义包含空格的文件名。如果args是一个序列，第一项指定了命令，另外的项目将作为传递给 shell（而非命令）的参数对待。也就是说， Popen 等同于:
```python
Popen(['/bin/sh', '-c', args[0], args[1], ...])
```
简单示例如下：
```python
>>> import subprocess
>>> subprocess.Popen(['ls','-l'])
total 0
<subprocess.Popen object at 0xa00000000151288>
-rw-r--r--    1 root     system            0 Jul 14 08:55 aix.py
>>> subprocess.Popen('ls -l',shell=True)
<subprocess.Popen object at 0xa0000000014c9c8>
>>> total 0
-rw-r--r--    1 root     system            0 Jul 14 08:55 aix.py

>>> subprocess.Popen(['/bin/sh','-c','ls -l'])
<subprocess.Popen object at 0xa00000000151108>
>>> total 0
-rw-r--r--    1 root     system            0 Jul 14 08:55 aix.py

>>> 
```
有些命令可能比较复杂，可以用shlex.split()进行序列化
shlex类可以轻松地为类似于Unix shell的简单语法编写词法分析器。 这对于解析带引号的字符串通常很有用。
示例如下：
```python
>>> import shlex,subprocess
>>> command = input()
lslpp -h bos.rte
>>> args = shlex.split(command)
>>> print(args)
['lslpp', '-h', 'bos.rte']
>>> b = subprocess.Popen(args)
>>>   Fileset         Level     Action       Status       Date         Time        
  ----------------------------------------------------------------------------
Path: /usr/lib/objrepos
  bos.rte
                 7.1.4.30   COMMIT       COMPLETE     01/22/19     20:20:24    

Path: /etc/objrepos
  bos.rte
                 7.1.4.30   COMMIT       COMPLETE     01/22/19     20:20:24    

>>> 
```
### command模块
Python 3.8中没有这个模块，以后估计也不会有了。
