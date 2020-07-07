# Python学习笔记-枚举
### 枚举基本概念
枚举其实是个类
有一类数据，经常用一些数字代表一些数据类型，例如，1代表Thor，2代表Hulk，3代表Thanos，4代表Batman，当我们读到数字的时候，无法指定其代表意义。

字典可能比较合适表示这类型数据，这里就需要枚举这个数据类型表示这种数据。
编写枚举需要导入一个枚举类，从enum模块导入类Enum；
枚举在Python本质是一个类，需要定义一个类，然后继承默认的Enum，所有枚举类都是Enum的子类；
示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
    Thanos = 3
    Batman = 4
print(SuperHero.Thor)
print(type(SuperHero.Thor))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
SuperHero.Thor
<enum 'SuperHero'>
```
枚举的标识名字，一般建议全部用大写，示例中首字母大写不推荐；
打印出来的结果就是SuperHero.Thor，类型可以看到是个枚举。这样才是枚举的意义所在，我们看代码时候，不在乎Thor具体数值是多少（可以为其它字符），对于写代码和读代码没多大意义，我们只需要知道代表的意义就行了。
### 枚举的特点和优势
上面的示例中，可以用字典表示下：{'Thor':1,'Hulk':2,'Thanos':3,'Batman':4},也可以用普通类表示，例如：
```python
class SuperHero():
    Thor = 1
    Hulk = 2
    Thanos = 3
    Batman = 4
```
上面两种表示方法都是可变的，就是可以在代码中轻易改变它们的值，并且没有防止相同标签的功能。
例如改变dict的值：
```python
a = {'Thor':1,'Hulk':2,'Thanos':3,'Batman':4}
a['Thanos'] = 5
print(a)
```
可以看到被改变了。

不同的类型用不同的数字来代表，字典和普通的类可以拥有相同的数值。在枚举中，定义的是常量，不能轻易被更改，
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
    Thanos = 3
    Batman = 4
print(SuperHero.Thor)
SuperHero.Thor = 5
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
SuperHero.Thor
Traceback (most recent call last):
  File "module_1.py", line 27, in <module>
    SuperHero.Thor = 5
    raise AttributeError('Cannot reassign members.')
AttributeError: Cannot reassign members.
{'Thor': 1, 'Hulk': 2, 'Thanos': 5, 'Batman': 4}
```
可以看到执行后报错了，用Thor = 1 后，也不能用Thor = 2，同意执行后会报错。
### 枚举类型、枚举名称与枚举值
如何获取枚举类型某一个标签所对应的具体数值，获取的方式通过value这个值，格式如下：SuperHero.Thor.value,通过.name获取标签的名字，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
    Thanos = 3
print(SuperHero.Thor.value)
print(SuperHero.Thor.name)
print(type(SuperHero.Thor.name))
print(SuperHero['Hulk'])
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
1
Thor
<class 'str'>
SuperHero.Hulk
```
之前有打印SuperHero.Thor，可以看到是个枚举类型，现在我们打印的name是个字符串。SuperHero['Hulk']表示名称所对应的枚举类型。
SuperHero.Thor是枚举类型，Thor是枚举名称，1就是枚举值。
### 枚举的比较运算
两个枚举之间是可以进行等值比较的，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
result = SuperHero.Thor == SuperHero.Hulk
print(result)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
False
```
如果将枚举和一个数值进行比较：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
result = SuperHero.Thor == 1
print(result)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
False
```
可以看到不会报错，但是得不到预期的结果，判断的结果是False。

枚举类型之间是不支持进行大小比较的，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
result = SuperHero.Thor < SuperHero.Hulk
print(result) 
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Traceback (most recent call last):
  File "module_1.py", line 60, in <module>
    result = SuperHero.Thor < SuperHero.Hulk
TypeError: '<' not supported between instances of 'SuperHero' and 'SuperHero'
```
可以看到报错了，说明不支持。
枚举类型直接支持身份比较，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
result = SuperHero.Thor is SuperHero.Thor
print(result) 
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Ture
```
不同的枚举之间的枚举类型数值是意义的，但是不同枚举之间不能做等值比较。
### 枚举注意事项
不能有两个相同的标签；
不同标签可以有两个相同的数值，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 1
print(SuperHero.Hulk) 
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
SuperHero.Thor
```
可是输出结果确实枚举类型却是Thor，而不是Hulk，在这种情况下，可以把Hulk看作成Thor的一个别名，本身不能代表一个枚举类型，改成Thor_ALIAS更适合。

枚举的遍历需要注意：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 1
    Thanos = 3
for a in SuperHero:
    print(a)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
SuperHero.Thor
SuperHero.Thanos
```
可以看到只打印出了SuperHero.Thor，如果一定要打印出来：
可以采用下面方法，遍历SuperHero下面的members属性，使用模块__members__下的items方法：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 1
    Thanos = 3
for a in SuperHero.__members__.items():
    print(a)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
('Thor', <SuperHero.Thor: 1>)
('Hulk', <SuperHero.Thor: 1>)
('Thanos', <SuperHero.Thanos: 3>)
```
不调用items方法，直接调用__members__输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Thor
Hulk
Thanos
```
### 枚举转换
将一个数值传换成枚举类型，使用类名，把a传递进来：SuperHero(a)，示例如下：
```python
from enum import Enum
class SuperHero(Enum):
    Thor = 1
    Hulk = 2
    Batman = 3
a = 3
print(SuperHero(a))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
SuperHero.Batman
```
### 枚举小结
除了Enum，还有IntEnum，
之前的示例中用的都是数字，也可以用字符串。如果不想用字符串，强制要求用数字，不允许用字符串，则可以用IntEnum；
如果不想用相同的枚举值，需要进行限制，则可以使用unique装饰器，使用示例如下：
```python
from enum import IntEnum,unique
@unique
class SuperHero(IntEnum):
    Thor = 1
    Hulk = 1
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Traceback (most recent call last):
  File "module_1.py", line 104, in <module>
    class SuperHero(IntEnum):
  File "C:\Users\AppData\Local\Programs\Python\Python38\lib\enum.py", line 860, in unique
    raise ValueError('duplicate values found in %r: %s' %
ValueError: duplicate values found in <enum 'SuperHero'>: Hulk -> Thor
```
可以看到报错提示说有重复的values。
