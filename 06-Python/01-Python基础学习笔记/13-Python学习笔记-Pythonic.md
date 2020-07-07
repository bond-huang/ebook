# Python学习笔记-Pythonic
### 用字典映射代替switch case语句
在其它语言中，switch是一个条件分支语句，swithch后面可以接一个变量，当变量取某个值的时候，就执行什么代码；
例如C#里面的switch语句：
```c#
switch (hero)
{
    hero 0 :
      heroname = "Thor";
      break;
    hero 1 :
      heroname = "THanos";
      break;
    ...
    default :
      heroname = "Batman";
      break
}
```
在Python中可以用if else来代替，也可以用字典映射来代替：
```python
hero = 'A'
superheros = {
    'T':'Thor',
    'A':'Captain America',
    'H':'Hulk',
    'I':'Iron Man',
}
superhero = superheros[hero]
print(superhero)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
Captain America
```
可以发现初步替代了，但是switch里面有default的选项，而字典里面如果传入的参数是不存在的，那么会报错，可以用Python内置get方法来实现，第一个参数仍然是需要传入的参数，第二个参数就是指定第一个参数对应的不存在的时候返回的结果：
```python
hero = 'C'
superheros = {
    'T':'Thor',
    'A':'Captain America',
    'H':'Hulk',
    'I':'Iron Man',
}
superhero = superheros.get(hero,'Batman')
print(superhero)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
Batman
```
在字典中，value不仅可以是字符串，还可以是函数，如同把函数赋值给变量一个道理：
```python
hero = 'C'
def get_ca():
    return 'Captain America'
def get_default():
    return 'Batman'
superheros = {
    'T':'Thor',
    'A':get_ca,
    'H':'Hulk',
    'I':'Iron Man',
}
superhero = superheros.get(hero,get_default)()
print(superhero)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
Batman
```
示例中调用了函数传入到字典中，在调用的时候返回也是函数，需要在后面加上()去调用函数的值，同样对于默认参数，如果有调入函数的，默认的返回结果也要传入一个函数，要不然会报错。
### 列表推导式
如果想根据一个已经存在的列表，创建一个新的列表，列表推导式很有用

例如一个列表，里面是数字，需要得到一个新的列表，值是原来列表数值的3次方，可以用之前学过的for循环来实现，也可以用map来实现；
```python
a = [2,4,6,8,10,12,14]
b = [i**3 for i in a]
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
[8, 64, 216, 512, 1000, 1728, 2744]
```
也可以进行条件筛选：
```python
a = [2,4,6,8,10,12,14]
b = [i**3 for i in a if i <=8]
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
[8, 64, 216, 512]
```
除了列表，用集合也行，列表推导式不一定是要中括号，改成啥就就输出什么类型的数据：
```python
a = [2,4,6,8,10,12,14]
b = {i**3 for i in a if i <=8}
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
{8, 512, 64, 216}
```
### 字典符合编写列表推导式
例如下面示例提取key，需要调用items去遍历字典：
```python
superhero = {
    'Thor':9,
    'Captain America':8,
    'Hulk':7
}
hero = [key for key,value in superhero.items()]
print(hero)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
['Thor', 'Captain America', 'Hulk']
```
如果需要讲value和key颠倒输出，将列表推导式输出结果颠倒下就行了，并且表达式也用字典的格式：
```python
superhero = {
    'Thor':9,
    'Captain America':8,
    'Hulk':7
}
hero = {value:key for key,value in superhero.items()}
print(hero)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
{9: 'Thor', 8: 'Captain America', 7: 'Hulk'}
```
如果想输出元组类型：
```python
superhero = {
    'Thor':9,
    'Captain America':8,
    'Hulk':7
}
hero = (key for key,value in superhero.items())
print(hero)
```
但是这样会报错，如下所示：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
<generator object <genexpr> at 0x00000205475B1200>
```
需要遍历下数据类型，因为元组是不可变的：
```python
superhero = {
    'Thor':9,
    'Captain America':8,
    'Hulk':7
}
hero = (key for key,value in superhero.items())
for i in hero:
    print(i)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
Thor
Captain America
Hulk
```
### None
在Python表示空，从类型和值上来看，不等于空字符串，也不等于空的列表，也不是0，也不是Flase；
```python
a = []
b = False
c =''
print(a==None)
print(b==None)
print(c==None)
print(a is None)
print(type(None))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
False
False
False
False
<class 'NoneType'>
```
继续进行示例：
```python
def function():
    return None
a = function()
if not a:
    print('T')
else:
    print('F')
if a is None:
    print('T')
else:
    print('F')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
T
T
```
可以看到两个输出都是T，但是这不能说明 not a和is None是一样的，修改下继续示例：
```python
a = []
if not a:
    print('T')
else:
    print('F')
if a is None:
    print('T')
else:
    print('F')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
T
F
```
用列表后，not a 输出值是Ture，所以返回'T'，空列表不等同于None，所以a is None 结果是False，返回值就是'F'。
一般判空操作可以用 fi a:  或者用if not a来判断。

None 和Flase不是用一个数据类型，None是NoneType类型，Flase是bool类型；
None表示不存在概念，Flase表示真假
有时候得到的结果相同，但是并不代表意义相同。
### 对象并不一定是True
当做if逻辑判断时候，Python每一个对象都和bool值有着对应关系，空字符串表示False，空列表表示False，非空字符串表示True；
None永远对应这Flase，
里面不定义方法，实例化后进行判断：
```python
class Hero():
    pass
hero = Hero()
if hero:
    print('T')
else:
    print('F')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
T
```
修改下，类中定义一个方法：
```python
class Hero():
    def __len__(self):
        return 0
hero = Hero()
if hero:
    print('T')
else:
    print('F')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
F
```
对类型进行bool判断：
```python
class Hero():
    def __len__(self):
        return 0
hero = Hero()
print(bool(hero))
print(bool(None))
print(bool([]))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
False
False
False
```
对于自定义类，判断是否是True还是Flase和内置的两个方法有关。
### __len__与__bool__内置方法
类下不存在任何方法，默认输出是True：
```python
class Hero():
    pass
print(bool(Hero()))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
True
```
加上__len__方法：
```python
class Hero():
    def __len__(self):
        return 0
print(bool(Hero()))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
False
```
如果是其它数字（需要是整形），就是True。

加入__bool__方法后，不再受__len__影响：
```python
class Hero():
    def __bool__(self):
        return False
    def __len__(self):
        return True
print(bool(Hero()))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\Pythonic> python module_1.py
False
```
__bool__方法中不能用0替代False，执行会报错，等同于不代表类型是相同的；
有了__bool__方法后就不会去调用__len__方法，所以不受__len__影响；
在Python2中__nonzero__来确定对象最终bool值，在Python3中用__bool__方法替代了。
