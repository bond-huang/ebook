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
