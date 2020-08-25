# Python-常见内置函数
Python中提供了一些内置函数可用直接调用，最常用的例如print()函数。官方文档:[内置函数](https://docs.python.org/zh-cn/3/library/functions.html#int)
### 基本数据处理
主要有三种：
- int()：返回一个基于数字或字符串 x 构造的整数对象，或者在未给出参数时返回 0
- oct()：将一个整数转变为一个前缀为“0o”的八进制字符串
- hex()：将整数转换为以“0x”为前缀的小写十六进制字符串

还有`tuple()`、`list()`、`str()`、`dict()` 、`set()` ·和`round()`使用方法和示例在学习笔记中有:[Python学习笔记-基本数据类型](https://bond-huang.github.io/huang/08-Python/01-Python%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/01-Python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.html)

### print()
标准格式：`print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)`

目前用到的也是常用方法示例：
```python
for a in range(0,10,2):
    print(a,end='|')
b = 'Iron Man!'
print('I am '+b)
```
执行输出结果：
```shell
PS D:\Python\codefile> python while_for.py
0|2|4|6|8|
I am Iron Man!
```
### input()
标准格式：` input([prompt])`

如果存在prompt实参，则将其写入标准输出，末尾不带换行符。接下来，该函数从输入中读取一行，将其转换为字符串（除了末尾的换行符）并返回。当读取到 EOF 时，则触发 EOFError。示例如下：
```python
>>> input('Please input hdisk name:')
Please input hdisk name:hdisk2
'hdisk2'
```
知识点：
- 可以直接将结果赋值给变量示例：`hero = input('Please input hero name:')`
- 返回结果是字符串，所有输入的数字也是字符串类型，如果需要转换注意转换成对应数据类型

### quit()
在代码中输入`quit()`可以直接终止代码执行并退出到shell，就是后面的代码都不执行了。
### range()
range 类型表示不可变的数字序列，通常用于在 for 循环中循环指定的次数:`for i in range(0,10)`,基本示例如下：
```python
>>> list(range(0, 10, 3))
[0, 3, 6, 9]
```
### len()
返回对象的长度（元素个数）。实参可以是序列（如 string、bytes、tuple、list 或 range 等）或集合（如 dictionary、set 或 frozen set 等）。

在for循环中经常和range一起使用：`for i in range(0,len(hex_ids))`。
### eval()
标准格式： `eval(expression, [globals,[locals]])`

实参是一个字符串，以及可选的 globals 和 locals。globals 实参必须是一个字典。locals 可以是任何映射对象。

用到过的示例：`oct_id = oct(eval('0x'+hex_ids[i]))`，对eval里面表达式计算成16进制格式，然后再用oct()转换成8进制。
### open()
标准格式：`open(file, mode='r',buffering=-1,encoding=None,errors=None,newline=None,closefd=True,opener=None)`

打开 file 并返回对应的 file object。如果该文件不能打开，则触发 OSError。
### format()
标准格式： `format(value,[format_spec])`

目前还没使用过，看起来很有用，可以参考：[格式规格迷你语言](https://docs.python.org/zh-cn/3/library/string.html#formatspec)
### max() & min()
标准格式：`max(arg1, arg2, *args,[key])`

返回可迭代对象中最大的元素，或者返回两个及以上实参中最大的。
###  filter()
标准格式：`filter(function, iterable)`

用 iterable 中函数 function 返回真的那些元素，构建一个新的迭代器。经常和lambda表达式结合使用，
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
###  map()和reduce()
和filter()一样，结合使用的话实用性很大，使用方法在学习笔记函数式编程中有介绍，[Python学习笔记-函数式编程](https://bond-huang.github.io/huang/08-Python/01-Python%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/11-Python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B.html?h=filter)

### sort()
&#8195;&#8195;此方法会对列表进行原地排序，只使用 `< `来进行各项间比较。异常不会被屏蔽:如果有任何比较操作失败，整个排序操作将失败（而列表可能会处于被部分修改的状态）,标准格式：`sort(*, key=None, reverse=False)`,参数说明：
- key指定带有一个参数的函数，用于从每个列表元素中提取比较键 (例如 key=str.lower)。 对应于列表中每一项的键会被计算一次，然后在整个排序过程中使用。默认值None表示直接对列表项排序而不计算一个单独的键值
- reverse为一个布尔值。如果设为True，则每个列表元素将按反向顺序比较进行排序

示例如下：
```python
list1 = [4,5,6,7,0,1,2,3]
list1.sort()
print(list1)
list1.sort(key=None)
print(list1)
list1.sort(key=None,reverse=False)
print(list1)
list1.sort(reverse=True)
print(list1)
list1.sort(key=None,reverse=True)
print(list1)
```
运行后输出结果如下：
```shell
PS C:\Users\big1000\vscode\codefile\Leetcode> python 1528.py
[0, 1, 2, 3, 4, 5, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7]
[7, 6, 5, 4, 3, 2, 1, 0]
[7, 6, 5, 4, 3, 2, 1, 0]
```
### 其它常用内置函数待使用
