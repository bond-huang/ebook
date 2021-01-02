# Python-array&readline
学习记录一些阵列操作方法。
## array高效的数值数组
此模块定义了一种对象类型，可以紧凑地表示基本类型值的数组：字符、整数、浮点数等。
### array.append(x)
添加一个值为x的新项到数组末尾。注意不能添加多个项目。使用示例：
```python
class Solution:
    def shuffle(self, nums: List[int], n: int) -> List[int]: 
        out_list = []
        for i in range(0,n):
            j = i + n
            x = nums[i]
            out_list.append(x)
            y = nums[j]
            out_list.append(y)
        return out_list
```
### array.extend(iterable)
&#8195;&#8195;将来自iterable的项添加到数组末尾。如果iterable是另一个数组，它必须具有完全相同的类型码；否则将引发 TypeError 如果iterable不是一个数组，则它必须为可迭代对象并且其元素必须为可添加到数组的适当类型。

如果指定多个不是列表的元素，会报错,必须以列表的形式添加进去：
```
extend() takes exactly one argument (2 given)
```
使用示例：
```python
item_list = []
    mtype = {'title':'Machine Typy','value':self.get_type()}
    sn = {'title':'Serial Number','value':self.get_sn()}
    fw = {'title':'Platform Firmware Level','value':self.get_fw()}
    item_list.extend([mtype,sn,fw])
```
### array.insert(i, x)
将值x作为新项插入数组的i位置之前。 负值将被视为相对于数组末尾的位置。

## array常用操作
### 列表取第一列数据
示例如下：
```python
>>> array = [[1,3,5,7,9],[2,4,6,8,10],[31,23,45,67,2]]
>>> newarray = [x[0] for x in array]
>>> print(newarray)
[1, 2, 31]
```
如果全是字符，第一列比较规则的话，可以采用切片方位方式:
```python
>>> import os
>>> a = os.popen('errpt -dS -s 1120154520')
>>> b = [x[0:8] for x in a]
>>> b
['IDENTIFI', '1BA7DF4E', 'CB4A951F', 'CB4A951F', '1BA7DF4E', 'CB4A951F', 'CB4A951F', '1BA7
DF4E', 'CB4A951F', 'CB4A951F', '1BA7DF4E', 'CB4A951F', 'CB4A951F', '1BA7DF4E', 'CB4A951F', 'CB4A951F', '1BA7DF4E', 'CB4A951F', 'CB4A951F']
```
### 根据第一列数据去重
&#8195;&#8195;有时候需要根据第一列数据去重，shell中sort和uniq都可以去重，排序后其它列的顺序也改变了，又不想改变其他列原有顺序，可以用python来实现。此示例不一定适用所有场合：
```python
import re
a = [[1,3,5,],[2,4,5],[1,23,45],[3,13,5],[2,13,5]]
list1 = []
for i in a:
    j = str(i[0])
    k = [x[0] for x in list1]
    if not re.findall(j,str(k)):
        list1.append(i)
print(list1)
```
运行示例：
```
bash-5.0# python3 test.py
[[1, 3, 5], [2, 4, 5], [3, 13, 5]]
```
## NumPY
&#8195;&#8195;Numeric Python扩展(NumPy)定义了另一种数组类型，是一个第三方库，需要安装，可以用于在数组提取某一列元素，或者矩阵按列进行重新组成新矩阵。官方网站：[https://numpy.org/](https://numpy.org/)

### 待补充
