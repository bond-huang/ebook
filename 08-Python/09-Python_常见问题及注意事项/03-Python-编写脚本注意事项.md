# Python-其它注意事项
记录一些学习和编写脚本过程中遇到的一些注意事项。
## AIX脚本编写注意
### 命令执行问题
&#8195;&#8195;例如sw_err是python变量，一串字符，想使用awk去处理sw_err中的内容，会将shell命令结果进行执行，而达不到预期结果：
```python
identifier = os.popen('awk \'{print $1}\' ' + sw_err)
```
下面写法一样,得到的不是命令执行结果，而是再去执行命令的结果：
```
ident_cmd = 'awk \'{print $1}\' ' + sw_err
identifier = os.popen(ident_cmd)
```
上面的方法使用是一厢情愿，行不通。下面的方法是可以：
```python
cmd1 = 'rmdev -dl '+hdisk
cmd1 = 'lspv |grep '+hdisk
os.popen(cmd1);os.popen(cmd2)
```
上面示例中hdisk变量只代表“hdisk1”，此方式可以。
### 脚本中运行命令问题
示例代码如下：
```python
>>> disk_list_cmd = 'lspv |awk \'{print $1}\''
>>> disk_list = os.popen(disk_list_cmd)
>>> for disk in disk_list:
...     disk_iostat_cmd = 'iostat -d '+disk+' 1 5 |grep '+disk+'| awk \'{print $2,$3,$4,$5,$6}\''
...     disk_iostat = os.popen(disk_iostat_cmd)
...     for i in disk_iostat:
...             print(i)
```
执行后会报错，只执行到`'iostat -d '+disk+`：
```
/bin/sh[2]: 1:  not found.
/bin/sh[3]: 0403-057 Syntax error at line 3 : `|' is not expected.
```
但是同样的赋值语句代码，下面这种就没问题：
```python
>>> import os
>>> disk = 'hdisk3'       
>>> disk_iostat_cmd = 'iostat -d '+disk+' 1 5 |grep '+disk+'| awk \'{print $2,$3,$4,$5,$6}\''
>>> disk_iostat = os.popen(disk_iostat_cmd)
>>> for i in disk_iostat:
...     print(i)
```
&#8195;&#8195;两个示例代码中，不同的是变量disk，第一个示例代码变量disk是从对象中遍历出来的，估计是有换行符，shell就从1开始执行了，所以报错，进行了如下测试：
```python
>>> disk_list = os.popen(disk_list_cmd)
>>> for disk in disk_list:
...     print(disk)
... 
hdisk3

hdisk4

hdisk5

>>> disk_list = os.popen(disk_list_cmd)
>>> for disk in disk_list:
...     disk = disk.strip()
...     print(disk)
... 
hdisk3
hdisk4
hdisk5
```
所以在最开始报错代码中修改下代码使用`strip()`将变量disk的换行符去掉即可。
### 获取命令使用read()问题
&#8195;&#8195;例如通过os.popen()运行命令后获取了一个对象，在一个if语句中调用read()去读取，在else中继续使用变量，就会发现是read()后的结果，而不是运行os.popen()获取的对象：
```python
def missing_check(self):
    missing_cmd= 'lspath -F "name status path_id parent connection"\
        |grep -i missing'
    missing_path = os.popen(missing_cmd)
    missing_path_list = []
    if len(missing_path.read()) == 0:
        missing_result = 'No missing path was found.'
    else:
        for path in missing_path:
            print(path)
            path = path.strip()
            path = path.split(' ')
            print(path)
```
&#8195;&#8195;示例中应该运行else，但是发现else后面可以运行，但是for循环后面的都没有结果，包括print(path)都不会输出。问题应该是在if语句中使用read()读取了missing_path，修改后代码如下：
```python
def missing_check(self):
    missing_cmd= 'lspath -F "name status path_id parent connection"\
        |grep -i missing'
    missing_path = os.popen(missing_cmd)
    missing_path = missing_path.read()
    missing_path_list = []
    if len(missing_path) == 0:
        missing_result = 'No missing path was found.'
    else:
        missing_path = os.popen(missing_cmd)
        for path in missing_path:
            path = path.strip()
```
## 待补充
