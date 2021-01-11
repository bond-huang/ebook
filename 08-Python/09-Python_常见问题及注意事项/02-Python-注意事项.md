# Python-其它注意事项
记录一些学习和实践过程中遇到的一些注意事项。
## jinja2模板
&#8195;&#8195;Python中使用jinja2模板处理文本时候，例如下面代码,本来获取到的是一个list，我转化成字符串，在Python中打印最后结果也是换行显示，但是将变量传入jinja2模板渲染后，结果html打开后显示是一行文本：
```python
sw_err = os.popen(sw_ckcmd)
    swerr_list = []
    for i in sw_err:
        j = i[0:10]
        k = [x[0:10] for x in swerr_list]
        if not re.findall(j,str(k)):
            swerr_list.append(i)
    sw_err = ''.join(swerr_list)
    sw_err = sw_err.strip()
    swerr_result = 'Found some software event,please check:\n'+ sw_err
```
&#8195;&#8195;如果直接在jinja2中遍历上面变量，会一个字母一个字母遍历，达不到预期效果，可以传入list，然后在jinja2模板中遍历即可，Python代码修改如下：
```python
sw_err = os.popen(sw_ckcmd)
    swerr_list = []
    for i in sw_err:
        j = i[0:10]
        k = [x[0:10] for x in swerr_list]
        if not re.findall(j,str(k)):
            swerr_list.append(i)
    swerr_result = ['Found some software event,please check:\n']+ swerr_list
```
jinja2模板中代码如下：
```html
<h3>System Software Error Event</h2> 
    {% for swerr in swerr_result %}
        <p>{{ swerr }}</p>
    {% endfor %}
```
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
## 浮点运算
&#8195;&#8195;在浮点运算中，用sum()内置函数进行运算的时候，会出现小数点位特别多的情况，而实际上原来相加的小数点是有限的，示例如下：
```python
>>> sum([0.1,0.1])
0.2
>>> sum([0.1,0.2])
0.30000000000000004
>>> sum([1.4,0.2])
1.5999999999999999
>>> a = 1.4;b = 0.2
>>> (a + b) == 1.6
False
```
可以使用round()内置函数来进行四舍五入取浮点数位数：
```python
>>> c = round((a + b),2)  
>>> print(c)
1.6
```
另一个方法是使用math.fsum()函数：
```python
>>> import math
>>> sum([0.1] * 10) == 1.0
False
>>> math.fsum([0.1] * 10) == 1.0
True
```
math.fsum()官方介绍：[math—数学函数](https://docs.python.org/zh-cn/3.6/library/math.html#math.fsum)

## 待补充
