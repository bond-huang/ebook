# Python高级语法与应用
### 闭包
在Python中函数是一个对象，在很多其它语言里面函数只是一段可执行的代码，不能进行实例化，Python中可以实例化。
Python中不仅可以把函数赋值给一个变量，还可以把另一个函数的参数，传递到另外的函数里，还可以把一个函数当作另外一个函数的返回结果。

例如下面顶一个函数，输出结果可以看到类型是个函数，并且是属于类：
```python
def x():
    pass
print(type(x))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
<class 'function'>
```
在函数的内部定义一个函数，在函数外是否可以调用：
```python
def super_hero():
    def hero():
        pass
hero()
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Traceback (most recent call last):
  File "module_1.py", line 4, in <module>
    hero()
NameError: name 'hero' is not defined
```
可以看到报错了，找不到hero，内部函数hero的作用域局限于super_hero函数的内部，在外部是不可调用的。
可以在super_hero的内部把hero函数返回回来，需要先调用super_hero，得到返回的hero，a就是hero函数：
```python
def super_hero():
    def hero():
        pass
    return hero
a = super_hero()
print(a)
print(type(a))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
<function super_hero.<locals>.hero at 0x000001EB6062ACA0>
<class 'function'>
```
上面示例验证了可以把函数赋值给一个变量，还可以把一个函数当作另外一个函数的返回结果。
继续示例，传递参数进去：
```python
def super_hero():
    t = 'Thor'
    def hero(x):
        return t*x
    return hero
T = super_hero()
print(T(2))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
ThorThor
```
上面示例修改下，在函数外部对t重新赋值：
```python
def super_hero():
    t = 'Thor'
    def hero(x):
        return t*x
    return hero
t = 'Thaons'
T = super_hero()
print(T(2))
print(T.__closure__)
print(T.__closure__[0].cell_contents)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
ThorThor
(<cell at 0x0000020526A56B80: str object at 0x0000020526A666B0>,)
Thor
```
可以看到值并没有改变，这就涉及到闭包的概念，闭包就是由函数（hero）以及函数在定义时候的外部环境变量（t）构成的一个整体，一旦函数和他的环境变量形成了闭包，在任何地方调用都不会受其它的变量重新赋值影响，还是会引用环境变量。
t不能在hero的内部，也不能定义成全局变量，即super_hero外部，这样都不能形成闭包；
示例中print(T.__closure__)打印出了闭包存放位置；
print(T.__closure__[0].cell_contents) 取出了闭包的环境变量；
闭包 = 函数 + 环境变量（函数定义时候的变量）
### 闭包的意义
闭包的意义：保存的是一个环境，把一个函数调用时候的现场保存起来了。
```python
def super_hero():
    a = 'Thor'
    def hero():
        a = 'Hulk'
        print(a)
    print(a)
    hero()
    print(a)
super_hero()
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
Thor
Hulk
Thor
```
super_hero()调用了函数super_hero，a 赋值后，定义了hero，没有调用或者执行hero，执行到print(a)，所以打印出来就是Thor，继续执行hero()，就调用了hero()，就执行hero()里面的内容，然后a就变成了 Hulk,hero()内部打印出来的a就是Hulk，继续执行super_hero()里的print(a)，可以看到打印出来是Thor，因为在hero的内部的a赋值 在Python中会被认为是一个局部变量，局部变量是不会影响到外部变量，所以打印出来是Thor。

a = 'Hulk'语句在的话，Python不会认为是个外部环境变量的改动，会被认为是一个局部变量，和外面的a就没有关系了，此时闭包就不存在了。
### 闭包示例
需要实现一个里程相加的功能，起始为0，如果走2米输出结果为2，如果继续走6米，里程相加，输出结果为8，先用非闭包的方式实现：
```python
origin = 0 
def travel(x):
    global origin
    distance = origin + x
    origin = distance
    return distance
print(travel(2))
print(travel(6))
print(travel(2))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
2
8
10
```
上面的示例中，注意要在函数内把origin全局化，即 global origin操作，如果不这样操作 ，在函数内部会认为origin是一个局部变量，当然也要去掉origin = distance （不去掉语法有问题），执行后输出结果就会一直是0。

下面用闭包的方法来实现：
```python
origin = 0
def travel(x):
    def distance(y):
        nonlocal x
        new_x = x + y
        x = new_x
        return new_x
    return distance        
traveler = travel(origin)
print(traveler(2))
print(traveler(6))
print(traveler(2))
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\advanced> python module_1.py
2
8
10
```
可以看到成功实现了，但是我们在函数中加入了代码：nonlocal x，如果不加，执行会报错，Python会认为x是一个本地的局部变量，nonlocal x 就是强制声明x不是一个本地的局部变量。

闭包可以记忆住上一次调用的状态，所以闭包可以实现上面的功能。所以的操作都是在函数的内部，并没有改变全局变量origin的值。
### 闭包总结
闭包可以实现在函数外部间接调用函数内部的变量；
闭包中的环境变量是一直常驻在内存中的，容易造成内存泄漏。
