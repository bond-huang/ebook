# Python学习笔记-循环&分支&条件

### 表达式
##### 表达式定义
表达式（Expression）是运算符（operator）和操作数（operand）所构成的序列
```python
>>> >>> a=('Thor')+('Hulk')*2;print(a)
ThorHulkHulk
>>> a=1;b=2;c=3;print(a+b*c);print(a or b and c)
7
1
```
在上面的示例中，a+b*c值是7，数学运算的优先级和基本数学运算一致；
a or b and c 结果是2，说明系统先运算的b and c然后再运算的or，所以得到了1，
这就涉及到表达式中的运算符的优先级，网上随便找了个表格，优先级如下所示：

|运算符说明|Python运算符|优先级|结合性|
---|:---:|---:|---:
|小括号|( )|19|无|
|索引运算符|x[i] 或 x[i1: i2 [:i3]]|18|左|
|属性访问|x.attribute|17|左|
|乘方|**|16|左|
|按位取反|~|15|右|
|符号运算符|+（正号）、-（负号）|14|右|
|乘除|*、/、//、%|13|左|
|加减|+、-|12|左|
|位移|>>、<<|11|左|
|按位与|&|10|右|
|按位异或|^|9|左|
|按位或|&#124;|8|左|
|比较运算符|==、!=、>、>=、<、<= |7|左|
|is 运算符|is、is not|6|左|
|in 运算符|in、not in|5|左|
|逻辑非|not|4|右|
|逻辑与|and|3|左|
|逻辑或|or|2|左|
|逗号运算符|exp1, exp2|1|左|

解析器在解析表达式的时候，如果优先级是同级的，解析器会从左到右去解析。在编程中叫左结合，如果有赋值运算’=‘，则变成右结合，先从’=‘右边开始运算，然后再赋值：
```python
>>> a=1;b=1;c=a+b;print(c)
2
>>> a='Thanos';b=3;c=3;not a or b+2==c
False
>>> a='Thanos';b=3;c=3;(not a) or((b+2)==c)
False
```
### 条件控制语句
##### Python中一些基础知识
单行注释，在行前面加上#即可
多行注释，在多行前后加上'''
python中用缩进来决定代码块，代码尾部也不需要用分号，通过换行来区分。
##### 条件控制语句
即if else语句，解决选择性问题，示例如下：
```python
world=False
if world:
    print('The world is True')
else:
    print('The world is Flase')
```
执行结果：
```shell
PS D:\Python\codefile> python test.py
The world is Flase
```
当然使用表达式也可以，if后面是个bool值即可：
```python
T='Thanos';I='Iron man'
if T>I:
    print('The world will end')
else:
    print('The world will be saved')
```
执行结果：
```shell
PS D:\Python\codefile> python test.py
The world will end
```
if else比较常用的就是密码判断，例如如下代码：
```python
account = 'root'
password = 'abc123'
print('Please input account')
user_account = input()
print('Please input password')
user_password = input()
if account == user_account:
    if password == user_password:
        print('You are login successful!')
    else:
        print('The password is wrong! Please try again!')
else:
    print('The account is wrong! Please try again!')
```
上面代码中，用到input函数，input是python中在命令行中接受用户输入的函数；
在python中，习惯用下划线“_”来隔离变量中的多个单词；
习惯用大写来定义变量，当然小写也可以；
各种运算符前后都建议空格。
正确输入账号密码后执行结果：
```shell
PS D:\Python\codefile> python test.py
Please input account
root
Please input password
abc123
You are login successful!
```
输错密码后执行结果：
```shell
PS D:\Python\codefile> python test.py
Please input account
root
Please input password
123456
The password is wrong! Please try again!
```
输错用户后执行结果：
```shell
PS D:\Python\codefile> python test.py
Please input account
python
Please input password
abc123
The account is wrong! Please try again!
```
##### if else其它注意事项
上面代码中使用了条件语句的嵌套使用，在实际应用中
if可以单独使用不需要else，但是else不能单独使用；
pass是空语句，或者叫占位语句，作用是保持代码结构的完整性，如果没有pass没任何代码，if语句是不能成功执行，当我们确实没有代码写入或者预留后期写入，可以写入pass保持结果完整性：
```python
if expression:
    pass
```
##### elif函数
elif是if else的简写，不能单独使用，只能在嵌套中代替if else使用，例如下面语句，用到了很多if else，代码行数很多：
```python
a = input()
if a == 'T':
    print('Thanos')
else:
    if a == 'C':
        print('Captain America')
    else:
        if a== 'H':
            print('Hulk')
        else:
            print('Iron Man')
```
我们可以用elif替代，就简洁不少：
```python
a = input()
if a == 'T':
    print('Thanos')
elif a == 'C':
    print('Captain America')
elif a== 'H':
    print('Hulk')
else:
    print('Iron Man')
```
随便输入个C 查看输出结果：
```shell
PS D:\Python\codefile> python test.py
C
Captain America
```
input输入数字注意，示例下面代码：
```python
a = input()
if a == 1:
    print('Thor')
elif a == 2:
    print('Thanos')
else:
    print('Captain America')
```
输入1，然后查看结果，可以看到并不是预期的结果，怎么输入都是结果都是一样：
```shell
PS D:\Python\codefile> python test.py
1
Captain America
```
在代码中加入打印a和打印a的类型：
```python
a = input()
if a == 1:
    print('Thor')
elif a == 2:
    print('Thanos')
else:
    print('Captain America')
print(a)
print(type(a))
```
然后输入1，查看结果：
```shell
1
Captain America
1
<class 'str'>
```
可以看到是个字符，所以我们在命令行输入的1会被仍为是个字符，所以不等于数字1，
转换一下类型就可以了：
```python
a = input()
a = int (a)
if a == 1:
    print('Thor')
elif a == 2:
    print('Thanos')
else:
    print('Captain America')
print(type(a))
```
输入1再查看结果，得到了预期的几个：
```shell
PS D:\Python\codefile> python test.py
1
Thor
<class 'int'>
```
### 循环语句
##### while循环
主要用户递归的情况下
结构如下：
```python
while expression:
    pass
else:
    pass
```
while会不停重复判断，如果判断条件是False，则不会执行后面代码，条件一直是True，会出现死循环，例如如下代码会不停输出’I am Thanos‘：
```python
THANOS = True
while THANOS:
    print('I am Thanos')
```
如果不想出现死循环，运行次数是有限的，在while内部代码段中需要有影响条件的语句：
```python
counter = 1
while  counter <=3:
    counter +=1
    print(counter)
```
执行输出结果，可以看到到4就结束了：
```shell
PS D:\Python\codefile> python while_for.py
2
3
4
```
while 除了单独使用，可以和else一起使用，当while后面条件语句返回值是False时候，就会执行else后面的代码.
```python
counter = 1
while  counter <=3:
    counter +=1
    print(counter)
else:
    print('End')
```
执行输出结果，可以看到输出到4结束后就执行了打印出End：
```shell
PS D:\Python\codefile> python while_for.py
2
3
4
End
```
##### for循环
主要用来遍历/循环 序列、集合或者字典
基本结构如下：
```python
for target_list in expression_list:
    pass
```
示例如下：
```python
super_hero = ['Iron Man','Captain America','Hulk']
for x in super_hero:
    print(x)
```
运行后可以看到依次打印出了列表中的元素：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man
Captain America
Hulk
```
当然也可以采用嵌套，
在下面示例中在打印中使用了end函数，默认不设置是end="/n" 这里设置为空字符，可以讲输出结构横向打印出来，之前的都是列排布：
```python
super_hero = [['Iron Man ','Captain America '],('Wonder Woman ','Batman')]
for a in super_hero:
    for b in a:
        print(b,end='')
```
执行输出结果如下：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man Captain America Wonder Woman Batman
```
for循环也是可以和else搭配使用；
当列表里面所有元素都被遍历完之后，就会执行else：
```python
super_hero = [['Iron Man ','Captain America '],('Wonder Woman ','Batman')]
for a in super_hero:
    for b in a:
        print(b,end='')
else:
    print('Super hero is gone!')
```
执行输出结果如下：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man Captain America Wonder Woman Batman Super hero is gone!
```
跳出循环的方法：
采用break和continue
采用break后，if后面判断条件成立，就立即终止代码，不会继续执行后续循环，包括后面的else内容，例如如下代码：
```python
super_hero = ['Iron Man','Captain America','Thor']
for a in super_hero:
    if a == 'Captain America':
        break
    print(a,end='')
else:
    print('End')
```
执行输出结果如下，可以看到if判断成立后，直接终止整个循环了：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man
```
如果想排除某个元素，但是想继续执行，则可以用continue：
```python
super_hero = ['Iron Man','Captain America','Thor']
for a in super_hero:
    if a == 'Captain America':
        continue
    print(a,end='')
else:
    print('End')
```
执行结果如下，可以看到排除了if判断成立的元素，但是代码继续在执行：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man Thor End
```
在嵌套中，加入break，如果是在内部循环，只会终止内部循环，外部循环继续执行，示例如下：
```python
super_hero = [['Iron Man ','Captain America ','Hulk '],('Wonder Woman ','Batman ')]
for a in super_hero:
    for b in a:
        if b == 'Captain America ':
            break
        print(b,end='')
else:
    print('Super hero is gone!')
```
执行输出结果如下：
```shell
PS D:\Python\codefile> python while_for.py
Iron Man Wonder Woman Batman Super hero is gone!
```
如何在for循环中指定循环次数，例如十次：
```python
for a in range(0,10):
    print(a,end='')
```
执行输出结果如下：
```shell
PS D:\Python\codefile> python while_for.py
0123456789
```
可以看到从0开始循环了十次，range函数就是指定一个范围，第一个元素是起始位，第二个元素不是表示结束位，表示是偏移量，就是从零开始有多少个元素。

如果希望生成的数字有间隔，可以加入第三个元素，及步长，设置成2:
```python
for a in range(0,10,2):
    print(a,end='|')
```
执行输出结果：
```shell
PS D:\Python\codefile> python while_for.py
0|2|4|6|8|
```
在输出end函数中使用了end='|',表示输出结果以行输出，并且用字符 | 隔开；
如果想递减，同样用range函数，第一个参数大于第二参数即可，第三个参数可以为负数，range（10，0，-2），即可输出10  8 6 4 2。

如果想打印出一大串数字中的奇数，例如1-20：
```python
a = range(1,20)
for b in range(0,len(a),2):
    c = a[b]
    print(c,end=',')
```
执行输出结果：
```shell
PS D:\Python\codefile> python while_for.py
1,3,5,7,9,11,13,15,17,19,
```
上面我用for循环方式实现，其实可以直接用切片方式实现：
```python
a = range(1,20)
b = a[0:len(a):2]
print(list(b))
```
执行输出结果：
```shell
PS D:\Python\codefile> python while_for.py
[1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
```
