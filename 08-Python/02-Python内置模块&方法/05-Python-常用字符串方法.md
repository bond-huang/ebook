# Python-常用字符串方法
&#8195;&#8195;字符串实现了所有一般序列的操作，还额外提供了以下列出的一些附加方法。官方文档:[内置类型](https://docs.python.org/zh-cn/3/library/stdtypes.html?highlight=join#str.join)

### str.join()
&#8195;&#8195;返回一个由iterable中的字符串拼接而成的字符串。如果iterable中存在任何非字符串值包括bytes对象则会引发 TypeError。调用该方法的字符串将作为元素之间的分隔。标准格式：`str.join(iterable)`,示例如下：
```python
x = ('l', 'e', 'e', 't', 'c', 'o', 'd', 'e')
y = ','.join(x)
print(y)
y = ''.join(x)
print(y)
```
运行后输出结果如下：
```shell
PS C:\Users\vscode\codefile\Leetcode> python 1528.py
l,e,e,t,c,o,d,e
leetcode
```
应用实例：在解答leetcode题目1528. Shuffle String时候用到此方法。

### str.strip()
&#8195;&#8195;标准格式：` str.strip([chars])`   
&#8195;&#8195;返回原字符串的副本，移除其中的前导和末尾字符。chars参数为指定要移除字符的字符串,如果省略或为 None，则chars参数默认移除空格符。实际上chars参数并非指定单个前缀或后缀,而是会移除参数值的所有组合,示例如下：
```python
>>> import os
>>> cmd_type = 'uname -M |awk -F, \'{print $2}\''
>>> mtype = os.popen(cmd_type)
>>> mtype = mtype.read()
>>> print(mtype)
9117-570

>>> print(mtype.strip())
9117-570
>>> ' Captain America   '.strip()
'Captain America'
>>> 'www.google.com'.strip('cmowz.')
'google'
```
### str.split()
标准格式：`str.split(sep=None, maxsplit=-1)`   
说明：
- 返回一个由字符串内单词组成的列表，使用sep作为分隔字符串
- 如果给出了maxsplit，则最多进行maxsplit次拆分；如果maxsplit未指定或为-1，则不限制拆分次数
- 如果给出了sep，则连续的分隔符不会被组合在一起而是被视为分隔空字符串

示例如下：
```
>>> import os
>>> user_rate_cmd = 'iostat -t 1 5|awk \'{print $3,$4,$5}\'|sed -n \'/^[0-9]/p\''
>>> user_rate = os.popen(user_rate_cmd)
>>> for i in user_rate:
...     print(i)
... 
1.0 2.9 96.1
0.5 2.2 97.3
0.7 2.5 96.8
0.2 1.8 98.0
0.2 1.3 98.5
>>> user_rate = os.popen(user_rate_cmd)
>>> for i in user_rate:
...     j = i.split(' ')
...     print(j)
... 
['2.9', '5.4', '91.8\n']
['0.3', '1.8', '98.0\n']
['0.2', '1.8', '98.0\n']
['0.2', '1.8', '97.9\n']
['0.4', '1.6', '98.0\n']
```
如果不进行分割，如果去遍历j，就会一个字符一个字符去遍历。
### 待补充
