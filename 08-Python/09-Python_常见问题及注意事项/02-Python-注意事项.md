# Python-其它注意事项
记录一些学习和实践过程中遇到的一些注意事项。
## jinja2
### 模板使用问题
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
### 变量使用问题
&#8195;&#8195;在jinja2中，变量是一个字典，调用变量可以使用&#123;&#123; &#125;&#125;语法表示一个变量， 获取字典变量中值可以类似使用切片方法，但是下面代码使用不行：
```html
{% for abnormal in abnormal_path_result %}
    <h3>Check the {{ abnormal['name'] }} path</h3>
    {% if {{ abnormal['result'] }} is string %}
        <p>{{ abnormal['result'] }}</p>
```
abnormal_path_result是一个Python字典变量，使用会报错：
```
  File "./base.html", line 265, in template
    {% if {{ abnormal['result'] }} is string %}
jinja2.exceptions.TemplateSyntaxError: expected token ':', got '}'
```
&#8195;&#8195;原因在控制结构&#123;&#37; &#37;&#125;中间，不需要使用&#123;&#123; &#125;&#125;再去表示一个变量，就好比第一行代码那样，所以代码修改如下即可：
```html
{% for abnormal in abnormal_path_result %}
    <h3>Check the {{ abnormal['name'] }} path</h3>
    {% if abnormal['result'] is string %}
        <p>{{ abnormal['result'] }}</p>
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
