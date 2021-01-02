# Python-常见问题
记录一些学习和实践过程中遇到的问题。
## 运行常见报错
### 缩进问题
预期缩进的块：
```
expected an indented block
```
unindent与任何外部缩进级别都不匹配
```
unindent does not match any outer indentation level
```
以上两个检查缩进。    

### 数据类型问题
示例一：
```
TypeError
expected string or bytes-like object
```
在取字典里面的值进行字符串相加时候报错，把格式转换成字符串即可，例如：dict['name'][0]

## 编码报错
### 报错示例一
报错：'utf-8' codec can't decode byte 0xc8 in position 334: invalid continuation byte    
原因：编码模式不对str(web_content,encoding='utf-8') gb18030,或者使用GBK

修改后发现正则表达式匹配不到数据，单独做测试发现如下报错：
```
SyntaxError: Non-UTF-8 code starting with '\xe9' in file test.py on line 15
```
需要在脚本开头加上：`# -*- coding:gb18030 -*-`或者`#coding:utf-8`

### 报错示例二
报错示例：
```
bash-5.0# python3 setup.py
System check in progress!
Get system information,please waiting......
Traceback (most recent call last):
  File "setup.py", line 16, in <module>
    print(content)
UnicodeEncodeError: 'latin-1' codec can't encode character '\uff1a' in position 443: ordin
al not in range(256)
```
在写html中，一个冒号使用了中文的字符，因为没有指定UTF-8，但是我使用的全是英文，所以改过来即可。
## 正则表达式注意
正则表达式中使用变量作为参数：
```python
import re
a = [[1,3,5,],[2,4,5],[1,23,45],[3,13,5],[2,13,5]]
list1 = []
for i in a:
    j = i[0]
    if not re.findall(j,list1):
        list1.append(i)
print(list1)
```
会报错
```
TypeError: first argument must be string or compiled pattern
```
查看正则表达式findall格式：
```python
re.findall(pattern, string, flags=0)
```
应该有两个问题：
- pattern在示例中是整型，改成字符串
- string在示例中是列表，同样改成字符串

修改后代码如下：
```python
import re
a = [[1,3,5,],[2,4,5],[1,23,45],[3,13,5],[2,13,5]]
list1 = []
for i in a:
    j = str(i[0])
    if not re.findall(j,str(list1)):
        list1.append(i)
print(list1)
```
运行示例：
```
[[1, 3, 5], [2, 4, 5]]
```
## 内置函数使用问题
### len()使用报错
报错内容：
```
TypeError: object of type '_wrap_close' has no len()
```
原因是我在使用os.popen()运行系统命令赋值给变量后得到的是一个对象：
```python
cpu_type = os.popen(type_cmd)
if len(cpu_type) == 0:
```
使用read函数读取对象内容即可：
```python
cpu_type = os.popen(type_cmd)
cpu_type = cpu_type.read()
if len(cpu_type) == 0:
```
## 待补充
