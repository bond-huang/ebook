# Python学习笔记-函数
### 函数简介
函数特性：功能性、避免编写重复的代码

之前用到过很多函数，例如print等
Python中定义了很多函数，例如round，保留小数位，并且四舍五入。
新的学习内容，为了区分，建立一个新的包：function。并在包下面建立新的模块module1.py
示例下round函数：
```python
a = 1.2328364
b = round(a,3)
print(b)
```
执行后输出结果：
```shell
PS D:\Python\codefile\function> python module1.py
1.233
```
可以看到结果中保留了三位有效小数位，并且进行了四舍五入运算。

如何快速查看Python中各类函数的功能
首先在windows下进入phthon的idle，使用help命令去查看详细介绍，格式：help(function)：
```shell
PS D:\Python\codefile\function> python
Python 3.8.3 (tags/v3.8.3:6f8c832, May 13 2020, 22:37:02) [MSC v.1924 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> help(round)
Help on built-in function round in module builtins:

round(number, ndigits=None)
    Round a number to a given precision in decimal digits.

    The return value is an integer if ndigits is omitted or None.  Otherwise
    the return value has the same type as the number.  ndigits may be negative.

>>>
```
### 函数的结构
在Python中，用def定义函数，格式如下：
```python
def funcname(parameter_list):
    pass
```
funcname：自动义函数名
parameter_list ：参数列表，需要用括号括起来
pass：空语句，占位语句，可以编写任意代码

参数列表可以没有
在函数体中可以使用return返回结果，结果用value表示，如果函数体没有return，那么Python默认返回的是None
### 函数的编写
定义一个自定义函数后，调用必须在定义后面
例如实现如下两个功能：两个数字或字符相加
```python
def add(a,b):
    result = a + b
    return result
a = add ('Thanos','Loki')
print(a)
```
执行输出结果如下：
```shell
PS D:\Python\codefile\function> python module1.py
ThanosLoki
```
打印输入的参数：
定义如下函数，函数中调用了系统print函数，在print函数不需要使用return，因为已经打印了，不需要返回结果：
```python
def print(code):
    print(code) 
```
可以看到我们定义的和系统print函数名字一样，这样会不停自己调用自己，执行会报错。
原因在于我们定义了一个和Python内置函数同名，所以一般命令不要和内置函数冲突，换个名字然后调用定义的函数：
```python
def print_input(code):
    print(code) 
print_input('Captain America')
```
执行输出结果：
```shell
PS D:\Python\codefile\function> python module1.py
Captain America
```
把刚才两个示例合在一起：
```python
def add(a,b):
    result = a + b
    return result
def print_input(code):
    print(code) 
a = add ('Thanos','Loki')
b = print_input('Captain America')
print(a,b)
```
执行输出结果：
```shell
PS D:\Python\codefile\function> python module1.py
Captain America
ThanosLoki None
```
可以看到先打印出了b，然后打印a，并且没有换行打印出None，这和Python执行顺序有关，
执行到给a赋值的时候，把结算结果赋值给a，但是目前并没有执行打印操作，继续执行给b赋值，在给b赋值的表达式中，调用了print_input函数，而print_input函数中调用了print函数，有打印操作，所以就先打印出了b的值，然后执行print(a,b)，所以接着就换行打印出了a的值，print(a,b)打印出结果不会换行，所以不换行接着打印b，由于print_code内部是没有return，会把函数的结果设置成None，所以print(a,b)打印出来是：ThanosLoki None
#### 关于return
一般函数内部代码执行到reture，后面的代码是不会执行。return返回结果没有限制，可以返回字符串，也可以返回元组等，可以返回一个函数。
返回值也可以是多个，用逗号隔开即可
```python
def damage(Thanos,Thor):
    damage1 = Thanos * 2
    damage2 = Thor *3 + 2
    return damage1,damage2
damages = damage(1,2)
print(type(damages))
print(damages)
damages1,damages2 = damage(1,2)
print(damages1,damages2)
```
执行输出结果
```shell
PS D:\Python\codefile\function> python module1.py
<class 'tuple'>
(2, 8)
2 8
```
可以看到途中打印了两种输出结果，第一种方法damages输出类型是个元组，打印出来也是个元组；
第二种用两个变量接收了函数的两个返回结果，用返回结果实际意义来表示，这种方法叫序列解包。
一般建议用第二种方法。
#### 序列解包
之前经常需要定义多个变量，例如a=1;b=2;c=3，可以简写成a,b,c=1,2,3。这样就比较简洁直观。
如果是a=1;b=1;c=1，则可以携程a=b=c=1;
可以用一个变量接收多个数值：例如d = 1,2,3
```python
d = 1,2,3
print(type(d))
a,b,c = d
print(a)
```
执行结果如下：
```shell
PS D:\Python\codefile\function> python module1.py
<class 'tuple'>
1
```
示例中可以给d赋值多个数值，类型输出位tuple，是个序列，代码中用a,b,c桑格值对d进行了序列解包，所以打印出a的值就是1。
序列解包的元素要相等,例如刚才的示例，d有三个元素，我们也需要定义三个变量去序列解包。
#### 函数的参数
以下面代码为例，解释参数类型。
必须参数（形式参数，实际参数）：在函数参数列表定义的参数（Thanos，Thor）是必须传递的，也就是调用函数的时候，必须给Thanos，Thor进行赋值；
形式参数：Thanos和Thor即形式参数；
实际参数：在函数调用过程中往函数的参数列表中给参数传递的实际取值，即下面代码中的1和2；
```python
def damage(Thanos,Thor):
    damage1 = Thanos * 2
    damage2 = Thor *3 + 2
    return damage1,damage2
damages = damage(1,2)
print(type(damages))
print(damages)
damages1,damages2 = damage(1,2)
print(damages1,damages2)
```
关键字参数：就是调用的时候，明确指定给的参数值，不用在乎序列，指定赋值，例如在上面的代码进行如下调用：
```python
damages = damage(Thor=1,Thanos=2)
```
默认参数：
例如如下代码：
```python
def print_self_introduction(name,home,want,dream):
    print('I am '+ name)
    print('I am come form ' + home)
    print('I want ' + want)
    print('I will '+ dream)
print_self_introduction('Thanos','Titan','Power','Balance the universe')
```
执行输出结果如下：
```shell
PS D:\Python\codefile\function> python module1.py
I am Thanos
I am come form Titan
I want Power
I will Balance the universe
```
可以看到在调用定义函数后依次打印出来了，但是如果我要打印很多人的信息，他们就name和home不一样，其它都一样，如果特别多，每个这样调用肯定很复杂。
可以通过修改函数的定义，在形式参数那里预先进行定义，没有默认参数的必须传递是个实际参数；当然如果不想用默认的，只需要更改下就行了：
```python
def print_self_introduction(name,home='Earth',want='Power',dream='save the world'):
    print('I am '+ name)
    print('I am come form ' + home)
    print('I want ' + want)
    print('I will '+ dream)
print_self_introduction('Iron Man','Earth','Power','save the world')
print('~~~~~~~~~~~~~~~~~~~~~~~~~~~')
print_self_introduction('Batman')
print('~~~~~~~~~~~~~~~~~~~~~~~~~~~')
print_self_introduction('Thor','Asgard')
```
执行输出结果如下：
```shell
PS D:\Python\codefile\function> python module1.py
I am Iron Man
I am come form Earth
I want Power
I will save the world
~~~~~~~~~~~~~~~~~~~~~~~~~~~
I am Batman
I am come form Earth
I want Power
I will save the world
~~~~~~~~~~~~~~~~~~~~~~~~~~~
I am Thor
I am come form Asgard
I want Power
I will save the world
```
非默认参数不能放到默认参数后面，也就是必须传递的形式参数不能在默认的形式参数后面。
如果有一个name相同，但是其中一个形式参数不想用默认的，则可以在实际参数中使用关键字参数，关键字参数不在乎顺序：
```python
def print_self_introduction(name,home='Earth',want='Power',dream='save the world'):
    print('I am '+ name)
    print('I am come form ' + home)
    print('I want ' + want)
    print('I will '+ dream)
print_self_introduction('Ultron',dream='destroy the world')
```
执行输出结果如下：
```shell
PS D:\Python\codefile\function> python module1.py
I am Ultron
I am come form Earth
I want Power
I will destroy the world
```
