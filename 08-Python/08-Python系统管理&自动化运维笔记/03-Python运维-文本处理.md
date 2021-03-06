# Python运维学习笔记-文本处理
&#8195;&#8195;文本处理主要是一些基础字符串处理及正则表达式处理文本的知识，之前在Python基础学习中已经有所学习记录，此处记录一些编码及Jinja2模板学习笔记。学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## 字符集编码
学习字符集编码的历史，Unicode的概念及Python程序中如何处理字符集编码问题。
### 编码历史
&#8195;&#8195;ASCII编码将128个英文字符和符号映射到一个字节后的7bit，只有128个英文字符和符号，局限性很大。Unicode（万国码，国际码，统一码）时计算机科学领域李的一项业界标准，能够将实际上所有的符号都纳入其中。Unicode编码范围值0至0x10FFFF,通常记作U+xxxx，在Python中一般记作\uxxxx，其中xxxx为十六进制的数字。
### UTF-8编码
&#8195;&#8195;Unicode制定了各种存储编码的方式，称作UTF(Uniform Transformation Format)，流行的有UTF-8、UTF-16和UTF-32。实际工作中，大部分情况下使用UTF-8，主要因为：
- UTF-8采用字节为编码单元，不存在字节的大端和小端问题
- UTF-8中每个ASCII字符只需一个字节去存储，向后兼容，也就是一个ASCII文本本身也是一个UTF-8的文本

UTF-8编码规则：
- 对于单字节的字符，字节的第一位设为0，后7位为这个符号的unicode码，对于英文字母，UTF-8编码和ASCII码时相同的
- 对于x字节的符号，第一个字节千x位都这位1，第x+1位设为0，后面的字节前两位一律设为10，其它二进制位为这个符号的unicode码

编码语法表如下：

Unicode符号范围|UTF-8编码方式
:---|:---
0000 0000-0000 007F|0xxxxxxx
0000 0080-0000 07FF|110xxxxx 10xxxxxx
0000 0800-0000 FFFF|1110xxxx 10xxxxxx 10xxxxxx
0001 0000-0010 FFFF|11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

示例如下：
```
>>> name = '黄'
>>> name
'黄'
>>> name.encode('utf-8')
b'\xe9\xbb\x84'
```
### 字符集问题
&#8195;&#8195;在Python3中默认使用Unicode，书中使用Python2示例，我使用Python3示例不了书中的问题，略过此节：
```
>>> name = '刘德华'
>>> name
'刘德华'
>>> print(name)
刘德华
>>> len(name)
3
>>> name[0:1]
'刘'
```
### Python中的Unicode
&#8195;&#8195;在Python3中，字符串默认时Unicode，在Python2中使用Unicode需要在字符串签名加上一个“u”前缀，如果需要默认使用Unicode的字符串，可以使用下面方法导入：
```
>>> name = u'刘德华'
>>> from __future__ import unicode_literals
```
Python3中，内置的open函数可以指定字符集编码：
```
>>> name
'刘德华'
>>> with open('/tmp/data.txt','w',encoding='utf-8') as f:
... f.write(name)
```
&#8195;&#8195;在Python编程中，应该把编码和解码操作放在程序的最外围来处理，程序的核心部分都是用Unicode，可以在代码中使用下面的辅助函数，函数能够接受str或Unicode类型并返回需要的字符串类型，Python3的字符集处理辅助函数：
```python
def to_str(bytes_or_str):
    if isinstance(bytes_or_str,bytes):
        value = bytes_or_str.decode('utd-8')
    else:
        value = bytes_or_str
    return value
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str,str):
        value = bytes_or_str.decode('utd-8')
    else:
        value = bytes_or_str
    return value
```
## Jinja2模板
学习在Python的web开发中广泛使用的模板语言Jinja2。
### 模板介绍
&#8195;&#8195;作为工程师，可以使用Jinja2管理工作中的配置文件，Ansible就是用Jinja2来管理配置文件。Python的标准库自带了一个简单的模板，但功能有限，无法在模板中使用控制语句和表达式，不支持继承和重用等操作。模板使用示例：
```
>>> from string import Template
>>> s = Template('$who is a $role')
>>> s.substitute(who='Thor',role='God')
'Thor is a God'
>>> s.substitute(who='Captain America',role='SuperHero')
'Captain America is a SuperHero'
```
### Jinja2语法入门
&#8195;&#8195;Jinja2是Flask作者开发的一个模板系统，其灵活、快速和安全等优点被广泛使用，主要具有以下优点：
- 相对于Template，Jinja2更加灵活，提供了控制结构、表达式和继承等
- 相对于Mako，Jinja2提供了仅有的控制结构，不允许在模板中编写太多的业务逻辑，避免乱用
- 相对于Django模板，Jinja2的性能更好
- Jinja2模板的可读性很好

如果安装了Flask，Jinja2会随之安装，也可以进行单独安装：
```
[root@redhat8 ~]# pip install jinja2
[root@redhat8 ~]# python3 -c "import jinja2"
```
#### 语法块
&#8195;&#8195;Jinja2可以应用于任何基于文本的格式，如HTML和XML。Jinja2使用大括号的方式表示Jinja2的语法，主要有三种语法：
- 控制结构&#123;&#37; &#37;&#125;
- 变量取值&#123;&#123; &#125;&#125;
- 注释&#123;&#35; &#35;&#125;

示例如下：
```
{# note:disable template because we no longer user this
    {% for user in users %}
    ...
    {% endfor %}
#}
```
&#8195;&#8195;在Jinja2中for循环使用与Python类似，但没了复合语句末尾的冒号，需要使用endfor作为结束标志，if语句也一样，需要使用endif作为结束标志，示例如下：
```
{% if users %}
    <ul>
    {% for user in users %}
        <li>{{ user.username }}</li>
    {% endfor %}
    </ul>
{% endif %}
```
#### 变量
&#8195;&#8195;Jinja2模板中使用&#123;&#123; &#125;&#125;语法表示一个变量。Jinja2识别所有的Python数据类型，包括复杂的数据类型，例如列表、字典和对象等，如下所示：
```
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list,with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>
```
#### Jinja2中的过滤器
&#8195;&#8195;变量可以通过“过滤器”进行修改，可以理解为Jinja2中的内置函数和字符串处理函数，常用的过滤器如下表所示：

过滤器名|说明
:---|:---
safe|渲染值时不转义
capitalize|把值的首字母转换成大写，其它字母转换成小写
lower|把值转换成小写形式
upper|把值转换成大写形式
title|把值中每个单词的首字母都转换成大写
trim|把值的首位空格去掉
striptags|渲染之前把值中所有的HTML标签都删掉
join|拼接多个值为字符串
replace|替换字符串的值
round|默认对数字进行四舍五入，也可以用参数进行控制
int|把值转换成整型

过滤器与变量通过管道（|）分割，多个过滤器可以链式调用，前一个过滤器的输出会作为后一个过滤器的输入，示例如下：
```
{{ "Hello World" | replace("Hello","Good") }}
    ->Good World
{{ "Hello World" | replace("Hello","Good") | upper }}
    ->GOOD WORLD
{{ 58.59 | round }}
    ->59.0
{{ 58.59 | round | int }}
    ->59
```
更多过滤器可以参考官方文档： [https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters](https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters)
#### Jinja2的控制结构
&#8195;&#8195;if语句类似Python中if语句，但需要使用endif作为结束标志，可以判断一个变量是否定义，是否为空，是否为真值，也可以使用elif和else构建多个分支，示例如下：
```
{% if Thanos.sick %}
    Thanos is sick.
{% elif Thanos.dead %}
    Thor kill Thanos!He is a superhero!
{% else %}
    Thanos looks okay!
{% endif %}
```
#### Jinja2的for循环
&#8195;&#8195;Jinja2中的for循环用于迭代Python的数据类型，包括列表、元组和字典，在Jinja2中不存在while循环。在Jinja2中迭代列表示例如下：
```
<hl>Members</hl>
<ul>
    {% for user in users %}
        <li>{{ user.username }}</li>
    {% endfor %}
</ul>
```
遍历字典示例：
```
<dl>
    {% for key,value in d.iteritems() %}
        <dt>{{ key }}</dt>
        <dd>{{ value }}</dd>
    {% endfor %}
</dl>
```
&#8195;&#8195;Jinja2还提供了一些特殊变量，不需要定义就可以直接使用，如下表所示：

变量|描述
:---|:---
loop.index|当前循环迭代的次数（从1开始）
loop.index0|当前循环迭代的次数（从0开始）
loop.revindex|到循环结束的次数（从1开始）
loop.revindex0|到循环结束的次数（从0开始）
loop.first|如果是第一次迭代，为True，否则为False
loop.last|如果是最后一次迭代，为True，否则为False
loop.length|序列中的项目数
loop.cycle|在一串序列间取值的辅助函数

&#8195;&#8195;有一个联系人信息的字典，key是名字，value是电话，想以表格的形式显示在HTML页面上，还希望第一列是序号，在Python代码中实现示例如下：
```python
data = dict(Thor=13856788888,Hulk=18675557777,Batman=18033339999)
index = 0
for key,value in data.viewtiems():
    index += 1
    print(index,key,value,sep=",")
```
在Jinja2中示例如下：
```
{% for key,value in data.iteritems() %}
    <tr class="info" >
        <td>{{ loop.index }}</td>
        <td>{{ key }}</td>
        <td>{{ value }}</td>
    </tr>
{% endfor %}
```
#### Jinja2的宏
&#8195;&#8195;宏类似于编程语言中的函数，用于将行为抽象成可重复调用的代码块，与函数一样，宏分为定义和嗲用，示例如下：
```
{% macro input(name,type='text',value='') %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value}}">
{% endmacro %}
```
宏说明：
- 在宏的定义中，使用macro关键字定义一个宏，input是宏的名称
- input有三个参数，分别是name、type和value，其中type和value参数有默认值
- 与Jinja2中的for循环一样，不需要使用符合语句的冒号，用endmacro结束宏的定义

下面是宏的调用示例：
```
<p>{{ input('username',value='user') }}</p>
<p>{{ input('password','password') }}</p>
<p>{{ input('submit','submit','Submit') }}</p>
```
#### Jinja2的继承和Super函数
&#8195;&#8195;使用Jinja2进行文件管理，级别用不到继承功能，如果是进行web开发，Jinja2的继承功能使用广泛，最强大的部分就是模板继承。模板允许构建一个包含站点的共同元素的基本模板的“骨架”，并定义子模版可以覆盖的块。例如有个base.html的文档，内容如下：
```html
<html lang="en">
<head>
    {% bolck head %}
    <link rel="stylesheet" href="style.css" />
        <title>{% block title %}{% endblock %}-My Homepage</title>
    {% endblock %}
</head>
<body>
<div id="content">
    {% block content %}{% endblock %}
</div>
</body>
```
&#8195;&#8195;在base.html中，使用&#123;&#37; bolck name  &#37;&#125;的方式定义了三个块，这些块可以在子模块中进行替换或调用。下面是名为index.html的文档，内容如下：
```html
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
    {{ super() }}
    <style type="test/css">
    .important { color: #336699; }
    </style>
{% endblock %}
{% block content %}
    <h1>Index</h1>
    <p class="important">Welcome on my homepage. </p>
{% endblock %}
```
&#8195;&#8195;在index.html中，使用&#123;&#37; extends "base.html"  &#37;&#125;继承base.html后，其中的所有内容都会在index.html中展现，并在index.html中重新定义了title和contend这两个块的内容。
#### Jinja2的其它运算
&#8195;&#8195;Jinja2提供了算数操作、比较操作和逻辑操作，使用Jinja2模板时，尽量在Python代码中进行逻辑处理，在Jinja2中仅处理显示问题，所以一般很少使用Jinja2的变量和变量的运算操作，部分运算操作如下：
- 算数运算：`+-*/ // % * **`
- 比较运算：`== != > >= < <=`
- 逻辑运算：`not and or`

### Jinja2实战
&#8195;&#8195;在Flask中使用Jinja2，只需使用Flask包下的render_template函数访问模板即可。如果使用Jinja2管理配置文件，需要了解Jinja2提供的API。    
&#8195;&#8195;Jinja2模板中有Environment类，用于存储配置和全局对象，然后从文件系统或其它位置加载模板。大多数应用会在初始化时创建一个Environment对象并用它加载模板，配置Jinja2为应用加载文档的最简单方式如下：
```python
from jinja2 import Environment, PackageLoader
env = Environment(loader=PackageLoader('yourapplication','templates'))
```
&#8195;&#8195;上面代码创建了一个Environment对象和一个包加载器，该加载器会在yourapplication这个Python的templates目录下查找模板。然后以模板的名字作为参数调用Environment.get_template方法杰克，会返回一个模板，最后使用模板的render方法进行渲染，如下所示：
```python
template = env.get_template('mytemplate.html')
print(template.render(the='variables',go='here'))
```
&#8195;&#8195;除使用包加载器外，还可以使用文件系统加载器，不需要模板位于一个Python包下，可以直接访问系统中的文件。便于功能演示，在接下来的例子中使用下面这个辅助函数：
```python
import os
import jinja2
def render(tpl_path,**kwargs):
    path,falename = os.path.split(tpl_path)
    return jinja2.Environment(
        loader=jinja2.FileSystemLoader(path or './')
    ).get_template(filename).render(**kwargs)
```
#### 基本功能演示
学习模板渲染示例，例如有一个名为simple.html的文件，内容如下：
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <!-- 使用过滤器处理表达式的结果 -->
        <title>{{ title | trim }}</title>
    </head>
    <body>
        <!-- 注释 -->
        {# This is a Comment #}
        <ul id="navigation">
            <!-- for语句，以endfor结束 -->
            {% for item in items %}
                <!-- 访问变量的属性 -->
                <li><a href="{{ item.href }}">{{ item['cation'] }}</a></li>
            {% endfor %}
        </ul>
        <p>{{ content }}</p>
    </body>
</html>
```
示例说明：
- 在示例HTML模板中，使用for循环遍历一个列表，列表中每一项是一个字典
- 字典中包含了文字和链接，将使用字典中的数据渲染成HTML的超链接
- 示例中还使用了Jinja2提供的过滤器trim删除title中的空格

执行下面Python代码：
```python
import os
import jinja2
def render(tpl_path,**kwargs):
    path,falename = os.path.split(tpl_path)
    return jinja2.Environment(
        loader=jinja2.FileSystemLoader('./')
    ).get_template('simple.html').render(**kwargs)

def test_simple():
    title = "Title H   "
    items = [{'herf':'big1000.com','caption':'big1000'},{'herf':'google.com','caption':'go
ogle'}]    content="This is content"
    result = render('simple.html',**locals())
    print(result)
if __name__ == '__main__':
    test_simple()
```
执行后渲染模板结果如下：
```html
[root@redhat8 jinja2]# python3 simple.py
<!DOCTYPE html>
<html lang="en">
    <head>
        <!-- 使用过滤器处理表达式的结果 -->
        <title>Title H</title>
    </head>
    <body>
        <!-- 注释 -->
        
        <ul id="navigation">
            <!-- for语句，以endfor结束 -->
            
                <!-- 访问变量的属性 -->
                <li><a href=""></a></li>
            
                <!-- 访问变量的属性 -->
                <li><a href=""></a></li>
            
        </ul>
        <p>This is content</p>
    </body>
</html>
```
&#8195;&#8195;示例中，使用Jinja2渲染模板后title中的空格已经被删除，for循环也正确渲染了多个超链接标签。
#### 继承功能演示
使用base.html和index.html演示继承功能。     
base.html内容如下：
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <!-- 定义代码块，可以在子模版中重载 -->
        {% block head %}
            <link rel="stylesheet" href="style.css" />
            <title>{% block title %}{% endblock %} - My HomePage</title>
        {% endblock %}
    </head>
    <body>
        <div id="content">
            <!-- 定义代码块，没用提供默认内容 -->
            {% block content %}
            {% endblock %}
        </div>
        <div id="footer">
            <!-- 定义代码块，没用提供默认内容 -->
            {% block footer %}
            {% endblock %}
        </div>
    </body>
</html>
```
index.html内容如下：
```html
<!-- 写在开头，用以继承 -->
{% extends "base.html" %}
<!-- 标题模块被重载 -->
{% block title %}Index{% endblock %}
<!-- head模块被重载，并且使用super继承了base.html中的head内容 -->
{% block head %}
    {{ super() }}
<style type="test/css"> .important { color:#3333FF; }</style>
{% endblock %}
<!-- 覆盖了content模块 -->
{% block content %}
<h1>This is h1 content</h1>
<p class="important">Welcome on my homepage</p>
{% endblock %}
```
使用下面的Python代码渲染Jinja2模板：
```python
import os
import jinja2
def render(tpl_path,**kwargs):
    path,falename = os.path.split(tpl_path)
    return jinja2.Environment(
        loader=jinja2.FileSystemLoader('./')
    ).get_template(index.html).render(**kwargs)

def test_extend():
    result = render('index.html')
    print(result)
if __name__ == '__main__':
    test_extend()
```
渲染后生成的结果如下：
```html
[root@redhat8 jinja2]# python3 extend.py
<!-- 写在开头，用以继承 -->
<!DOCTYPE html>
<html lang="en">
    <head>
        <!-- 定义代码块，可以在子模版中重载 -->
        
    
            <link rel="stylesheet" href="style.css" />
            <title>Index - My HomePage</title>
        
<style type="test/css"> .important { color:#3333FF; }</style>

    </head>
    <body>
        <div id="content">
            <!-- 定义代码块，没用提供默认内容 -->
            
<h1>This is h1 content</h1>
<p class="important">Welcome on my homepage</p>

        </div>
        <div id="footer">
            <!-- 定义代码块，没用提供默认内容 -->
            
            
        </div>
    </body>
</html>
```
示例说明：
- 示例中渲染的是index.html，并没有直接渲染base.html，但最后生成的模板中保护玩的HTML框架
- 虽然在index.html中定义了title模块，但示例中使用&#123;&#123; super() &#125;&#125;引用了base.html中的HEAD模块，所以最后渲染结果中有base.html中的head块和index.html中的head块，结果中的`Index - My HomePage`就是来自于此
- 示例在index.html中重新定义了content块的内容，所以最后生成的文档中正确位置显示了content块的内容

### 案例演示
使用Jinja2生成HTML表格和XML配置文件。
#### 使用Jinja2生成HTML表格
