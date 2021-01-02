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
## 浮点运算
在浮点运算中，用sum()内置函数进行运算的时候，会出现小数点位特别多的情况，而实际上原来相加的小数点是有限的，示例如下：
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
