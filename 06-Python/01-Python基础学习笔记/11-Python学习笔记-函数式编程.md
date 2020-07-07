# Python学习笔记-函数式编程与装饰器
### 匿名函数
定义一个函数的时候，不需要去定义函数名
要定义一个匿名函数，需要借助lambda关键字，格式如下：
```python
lambda parameter_list: expression
```
例如，下面的普通函数：
```python
def add(x,y):
    return x + y
print(add(2,3))
```
利用匿名函数实现如下：
```python
z= lambda x,y:x+y
print(z(2,3))
```
调用匿名函数的话，可以将函数赋值给一个变量，然后再调用传递参数。
示例中还可以看到匿名函数的定义非常简洁。

但是在匿名函数中，后面expression是个表达式，不能是个代码块；在普通函数中，在函数里面可以写入条件语句或者定义函数，在匿名函数中，是不可以写入的，例如下面代码：
```python
z= lambda x,y:a = x+y
print(z(2,3))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
  File "module_1.py", line 5
    z= lambda x,y:a = x+y
       ^
SyntaxError: cannot assign to lambda
```
可以看到报错了，赋值操作是一个完整的代码语句，不是一个表达式，所以认为语法错误。
### 三元表达式
判断两个数字大小，可以用if语句来实现：
```python
a = 2
b = 5
if a > b:
    print(a)
else:
    print(b)
```
对于上面条件判断语句，可以用一个三元表达式来替代，格式如下：
条件为真是返回的结果 if 条件判断 else 条件为假时的返回结果
```python
a = 2
b = 5
c = a if a > b else b
print(c)
```
a if a > b else b 是三元表达式，因为只是个表达式，所以用变量c来接收三元表达式的结果，运行输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
5
```
三元表达式适合用在lambda表达式上。
### map
格式如下：
```python
map(func: Callable[[_T1], _S], iter1: Iterable[_T1])
map(func, *iterables) --> map object
```
有一个列表，列表下面有多个数字，求列表下面每个数字的平方，并且把结果组成一个新的列表打印出来。
可以用for循环来实现：
```python
list_x = [1,2,3,4,5,6,7]
def square(x):
    return x*x
for x in list_x:
    y = square(x)
    print(y)
```
用map来实现：
```python
list_x = [1,2,3,4,5,6,7]
def square(x):
    return x*x
y = map(square,list_x)
print(y)
print(list(y))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
<map object at 0x0000019BAAE06BB0>
[1, 4, 9, 16, 25, 36, 49]
```
y的值打印出来是个map的对象，转换成列表就达到了满足的需求了，转换成了平方的形式，并且输出还是个列表。

调用map后，map会对传入列表里面每一项元素都会传入到square函数里面去执行，并且接收函数的返回结果。
### map与lambda
lambda是一个函数，可以把square直接写在lambda里面，上面示例用lambda来实现：
```python
list_x = [1,2,3,4,5,6,7]
y = map(lambda x: x*x,list_x)
print(y)
print(list(y))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
<map object at 0x000002116DD36BB0>
[1, 4, 9, 16, 25, 36, 49]
```
上面示例也是函数式编程的一个示例。

如果有多个参数需要传入，示例如下：
```python
list_x = [1,2,3,4,5,6,7]
list_y = [7,6,5,4,3,2,1]
z = map(lambda x,y: x*y,list_x,list_y)
print(list(z))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
[7, 12, 15, 16, 15, 12, 7]
```
如果传入的参数列表中的元素数量不是相等的，输出结果元素个数取决于元素数量较小的参数列表，示例如下：
```python
list_x = [1,2,3,4,5,6,7]
list_y = [7,6,5,4,3,2]
z = map(lambda x,y: x*y,list_x,list_y)
print(list(z))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
[7, 12, 15, 16, 15, 12]
```
### reduce
使用reduce需要先从functools模块下导入，格式如下：
```python
from functools import reduce
def reduce(function, sequence, initial=None)
reduce(function, sequence[, initial]) -> value
```
reduce是一个函数，第一个参数是接受一个函数，reduce下的函数必须要有两个参数，示例如下：
```python
from functools import reduce
list_x = [1,2,3,4,5,6,7]
z = reduce(lambda x,y:x+y,list_x)
print(z)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
28
```
在示例中，并没有传入参数y，但是可以正常输出没有报错；
可以看到输出结果是28，是将x元素全部相加了；
reduce是做一个连续计算，连续调用lambda表达式，在第一次执行的适合，lambda会取前两个元素，也就是1 和2，然后进行表达式运算，结果为3，reduce函数特点，会把一次计算结果作为x传入到lambda表达式中，第二次调用lambda 函数y就是第三个参数3，计算结果就是6，依次类推，最后得到上面的结果，即计算流程是：(((((1+2)+3)+4)+5)+6)+7
就是每一次调用lambda的计算结果会作为下一次调用的参数，直到列表被遍历完成。

reduce 中还有一个参数，initial初始值，默认是None，把上面示例设置成20后：
```python
from functools import reduce
list_x = [1,2,3,4,5,6,7]
z = reduce(lambda x,y:x+y,list_x,20)
print(z)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
48
```
可以看到结果加了20，并不是在之前运算完成后再加了20，而是把20作为一个初始值去进行计算，也就是从10+1开始。
### filter
就是筛选，可以过滤掉一些不需要的元素或者不符合规则的元素，格式如下：
```python
def filter(function: None, iterable: Iterable[Optional[_T]])
filter(function or None, iterable) --> filter object
```
例如下面示例把list_a中的1筛选出来：
```python
list_a = [3,1,0,5,6,1,9,6,1,4,1]
b = filter(lambda a:True if a == 1 else False,list_a )
print(b)
print(list(b))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
<filter object at 0x00000172EFC46BB0>
[1, 1, 1, 1]
```
可以看到成功筛选出了，一样适合lambda函数和三元表达式一起使用。

上面示例中都是采用函数式编程的方式进行的，之前的学习章节中学习的是命令式编程方式进行的；
函数式编程常用的三个函数：map reduce filter ，以及算子：lambda
### 装饰器
如果想知道函数执行的时间，可以导入time模块下的time函数：
```python
import time
def super_hero():
    print(time.time())
    print('Batman is a superhero')
super_hero()
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592315521.594548
Batman is a superhero
```
但是时间不是我们常见的时间格式，显示的是一个unix时间戳，

如果有很多个函数，所有的函数都需要添加打印时间的功能，每个函数一个个去加就很麻烦了。
编程的一个基本原则：对修改是封闭的，对扩展是开放的
一个个去修改函数不符合上面原则，可以专门定义一个函数去实现：
```python
import time
def super_hero():
    print('Batman is a superhero!')

def hostile():
    print('Thanos is hostile!')
    
def print_time(func):
    print(time.time())
    func()
print_time(super_hero)
print_time(hostile)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592316546.351815
Batman is a superhero!
1592316546.3528154
Thanos is hostile!
```
但是这种方式也比较复杂，可以用装饰器来实现，下面定义的嵌套函数就是装饰器：
```python
import time
def decotator(func):
    def wrapper():
        print(time.time())
        func()
    return wrapper
def super_hero():
    print('Batman is a superhero!')
a = decotator(super_hero)
a()
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592323161.3373797
Batman is a superhero!
```
当时上面的示例还是改变了调用规则，可以直接在super_hero前面@装饰器的名字，就不需要改变调用了：
```python
import time
def decotator(func):
    def wrapper():
        print(time.time())
        func()
    return wrapper
@decotator
def super_hero():
    print('Batman is a superhero!')
super_hero()
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592323524.7907574
Batman is a superhero!
```
如果给函数传递一个参数呢，如何修改装饰器，示例如下：
```python
import time
def decotator(func):
    def wrapper():
        print(time.time())
        func()
    return wrapper
@decotator
def super_hero(hero):
    print(hero + 'is a superhero!')
super_hero('Batman')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
Traceback (most recent call last):
  File "module_1.py", line 113, in <module>
    super_hero('Batman')
TypeError: wrapper() takes 0 positional arguments but 1 was given
```
可以看到报错了，装饰器内部也需要修改：
```python
import time
def decotator(func):
    def wrapper(hero):
        print(time.time())
        func(hero)
    return wrapper
@decotator
def super_hero(hero):
    print(hero + 'is a superhero!')
super_hero('Batman')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592401954.4871726
Batman is a superhero!
```
如果有多个函数呢，装饰器只有一个，可以在装饰器中用可变参数，例如下面示例中的*args，args是很多编程语言里面代表一组参数，是一个比较通用的用法，当然用其它也行，通用就建议采用这种命名：
```python
import time
def decotator(func):
    def wrapper(*args):
        print(time.time())
        func(*args)
    return wrapper
@decotator
def super_hero(hero):
    print(hero + ' is a superhero!')
@decotator
def hostile(scut):
    print(scut +' is hostile!')
super_hero('Batman')
hostile('Thanos')
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592402396.377453
Batman is a superhero!
1592402396.3779383
Thanos is hostile!
```
如果函数中有关键字参数，例如下面示例函数中有关键字参数\*\*kw。在装饰器中\*\*kw是形参，名字可以随便命名：
```python
import time
def decotator(func):
    def wrapper(*args,**kw):
        print(time.time())
        func(*args,**kw)
    return wrapper
@decotator
def super_hero(hero):
    print(hero + ' is a superhero!')
@decotator
def hostile(scut,**kw):
    print(scut +' is hostile!')
    print(kw)
super_hero('Batman')
hostile('Thanos',a=1,b='59',c=1)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\function_code> python module_1.py
1592406125.517447
Batman is a superhero!
1592406125.5184083
{'a': 1, 'b': '59', 'c': 1}
```
#### 装饰器
如果想对一个函数增加一个功能，但是又不想取改变函数内部的实现，普通的函数是实现不了的，无论如何都会改变函数或者改变函数的调用，装饰器的优势就体现了。
