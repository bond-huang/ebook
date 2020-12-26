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

### str.strip
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
### 待补充
