# Python学习笔记-正则表达式
正则表达式是一个特殊的字符序列，一个字符串是否与我们所设定的字符序列相匹配
可以实现快速检索文本，实现一些替换文本的操作。
例如：
- 检查一串数字是否是电话号码；
- 检测一个字符串是否符合email；
- 把一个文件里指定的单词替换为另外一个单词。

新的学习内容，在vccode中新建一个包expression，在包下新建一个模块module_1.py开始新代码的学习和编写。
### 元字符
Python有个内置函数，index可以去查找字符串并判断，也可以用之前学过的in，例如在下面字符中找出神奇女侠：
```python
a= 'Thor|Hulk|Captain America|Wonder Woman|Iron Man'
print(a.index('Wonder Woman')>-1)
print('Wonder Woman' in a)
```
运行后输出结果如下，可以看到都返回了一个True，说明a中有神奇女侠：
```shell
PS D:\Python\codefile\expression> python module_1.py
True
True
```
用正式表达式也可以解决上面的问题。
Python有一个模块：re，模块中有很多方法，其中有一个findall，格式：findall(pattern: AnyStr, string: AnyStr, flags: _FlagsType=...) -> List[Any]
示例如下：
```python
import re
a = 'Thor|Hulk|Captain America|Wonder Woman|Iron Man'
b = re.findall('Wonder Woman',a)
if len(b) > 0:
    print('DC superhero in the team')
print(b)
```
运行后输出结果如下，可以看到都返回了一个list：
```shell
PS D:\Python\codefile\expression> python module_1.py
DC superhero in the team
['Wonder Woman']
```
上面示例中只是查找了一串连续的字符，如果不是连续的，正则表达式也可以实现，例如找出a中所有的数字，示例如下：
```python
import re
a = '1Thor2Hulk3Captain America4Wonder Woman5Iron Man'
b = re.findall('\d',a)
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['1', '2', '3', '4', '5']
```
在正则表达式中，\d表示数字0-9。在正则表达式中，‘Wonder Woman’是普通字符，\d是元字符，可以混合在一起使用，根据匹配需求来选择。

学习正则表达式其实就是学习元字符的使用，从百度百科上截取了正则表达式元字符列表：

元字符|描述
---|:---
\\ | 将下一个字符标记符、或一个向后引用、或一个八进制转义符。例如，“\\n”匹配\n。“\n”匹配换行符。序列“\\\”匹配“\”而“\(”则匹配“(”。即相当于多种编程语言中都有的“转义字符”的概念。
^|匹配输入字行首。如果设置了RegExp对象的Multiline属性，^也匹配“\n”或“\r”之后的位置。
$|匹配输入行尾。如果设置了RegExp对象的Multiline属性，$也匹配“\n”或“\r”之前的位置。
\*|匹配前面的子表达式任意次。例如，zo*能匹配“z”，也能匹配“zo”以及“zoo”。*等价于{0,}。
\+|匹配前面的子表达式一次或多次(大于等于1次）。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”。+等价于{1,}。
?|匹配前面的子表达式零次或一次。例如，“do(es)?”可以匹配“do”或“does”。?等价于{0,1}。
{n}|n是一个非负整数。匹配确定的n次。例如，“o{2}”不能匹配“Bob”中的“o”，但是能匹配“food”中的两个o。
{n,}|n是一个非负整数。至少匹配n次。例如，“o{2,}”不能匹配“Bob”中的“o”，但能匹配“foooood”中的所有o。“o{1,}”等价于“o+”。“o{0,}”则等价于“o*”。
{n,m}|m和n均为非负整数，其中n<=m。最少匹配n次且最多匹配m次。例如，“o{1,3}”将匹配“fooooood”中的前三个o为一组，后三个o为一组。“o{0,1}”等价于“o?”。请注意在逗号和两个数之间不能有空格。
?|当该字符紧跟在任何一个其他限制符（*,+,?，{n}，{n,}，{n,m}）后面时，匹配模式是非贪婪的。非贪婪模式尽可能少地匹配所搜索的字符串，而默认的贪婪模式则尽可能多地匹配所搜索的字符串。例如，对于字符串“oooo”，“o+”将尽可能多地匹配“o”，得到结果[“oooo”]，而“o+?”将尽可能少地匹配“o”，得到结果 ['o', 'o', 'o', 'o']
.点|匹配除“\n”和"\r"之外的任何单个字符。要匹配包括“\n”和"\r"在内的任何字符，请使用像“[\s\S]”的模式。
(pattern)|匹配pattern并获取这一匹配。所获取的匹配可以从产生的Matches集合得到，在VBScript中使用SubMatches集合，在JScript中则使用$0…$9属性。要匹配圆括号字符，请使用“\\(”或“\\)”。
(?:pattern)|非获取匹配，匹配pattern但不获取匹配结果，不进行存储供以后使用。这在使用或字符“(&#124;)”来组合一个模式的各个部分时很有用。例如“industr(?:y&#124;ies)”就是一个比“industry&#124;industries”更简略的表达式。
(?=pattern)|非获取匹配，正向肯定预查，在任何匹配pattern的字符串开始处匹配查找字符串，该匹配不需要获取供以后使用。例如，“Windows(?=95&#124;98&#124;NT&#124;2000)”能匹配“Windows2000”中的“Windows”，但不能匹配“Windows3.1”中的“Windows”。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。
(?!pattern)|非获取匹配，正向否定预查，在任何不匹配pattern的字符串开始处匹配查找字符串，该匹配不需要获取供以后使用。例如“Windows(?!95&#124;98&#124;NT&#124;2000)”能匹配“Windows3.1”中的“Windows”，但不能匹配“Windows2000”中的“Windows”。
(?<=pattern)|非获取匹配，反向肯定预查，与正向肯定预查类似，只是方向相反。例如，“(?<=95&#124;98&#124;NT&#124;2000)Windows”能匹配“2000Windows”中的“Windows”，但不能匹配“3.1Windows”中的“Windows”。*python的正则表达式没有完全按照正则表达式规范实现，所以一些高级特性建议使用其他语言如java、scala等
(?<!patte_n)|非获取匹配，反向否定预查，与正向否定预查类似，只是方向相反。例如“(?<!95&#124;98\&#124;NT&#124;2000)Windows”能匹配“3.1Windows”中的“Windows”，但不能匹配“2000Windows”中的“Windows”。*python的正则表达式没有完全按照正则表达式规范实现，所以一些高级特性建议使用其他语言如java、scala等
x&#124;y|匹配x或y。例如，“z&#124;food”能匹配“z”或“food”(此处请谨慎)。“[z&#124;f]ood”则匹配“zood”或“food”。
[xyz]|字符集合。匹配所包含的任意一个字符。例如，“[abc]”可以匹配“plain”中的“a”。
[^xyz]|负值字符集合。匹配未包含的任意字符。例如，“[^abc]”可以匹配“plain”中的“plin”任一字符。
[a-z]|字符范围。匹配指定范围内的任意字符。例如，“[a-z]”可以匹配“a”到“z”范围内的任意小写字母字符。注意:只有连字符在字符组内部时,并且出现在两个字符之间时,才能表示字符的范围; 如果出字符组的开头,则只能表示连字符本身.
[^a-z]|负值字符范围。匹配任何不在指定范围内的任意字符。例如，“[^a-z]”可以匹配任何不在“a”到“z”范围内的任意字符。
\b|匹配一个单词的边界，也就是指单词和空格间的位置（即正则表达式的“匹配”有两种概念，一种是匹配字符，一种是匹配位置，这里的\\b就是匹配位置的）。例如，“er\b”可以匹配“never”中的“er”，但不能匹配“verb”中的“er”；“\b1_”可以匹配“1_23”中的“1_”，但不能匹配“21_3”中的“1_”。
\B|匹配非单词边界。“er\B”能匹配“verb”中的“er”，但不能匹配“never”中的“er”。
\cx|匹配由x指明的控制字符。例如，\cM匹配一个Control-M或回车符。x的值必须为A-Z或a-z之一。否则，将c视为一个原义的“c”字符。
\d|匹配一个数字字符。等价于[0-9]。grep 要加上-P，perl正则支持
\D|匹配一个非数字字符。等价于[^0-9]。grep要加上-P，perl正则支持
\f|匹配一个换页符。等价于\x0c和\cL。
\n|匹配一个换行符。等价于\x0a和\cJ。
\r|匹配一个回车符。等价于\x0d和\cM。
\s|匹配任何不可见字符，包括空格、制表符、换页符等等。等价于[ \f\n\r\t\v]。
\S|匹配任何可见字符。等价于[^ \f\n\r\t\v]。
\t|匹配一个制表符。等价于\x09和\cI。
\v|匹配一个垂直制表符。等价于\x0b和\cK。
\w|匹配包括下划线的任何单词字符。类似但不等价于“[A-Za-z0-9_]”，这里的"单词"字符使用Unicode字符集。
\W|匹配任何非单词字符。等价于“[^A-Za-z0-9_]”。
\xn|匹配n，其中n为十六进制转义值。十六进制转义值必须为确定的两个数字长。例如，“\x41”匹配“A”。“\x041”则等价于“\x04&1”。正则表达式中可以使用ASCII编码。
\num|匹配num，其中num是一个正整数。对所获取的匹配的引用。例如，“(.)\1”匹配两个连续的相同字符。
\n|标识一个八进制转义值或一个向后引用。如果\n之前至少n个获取的子表达式，则n为向后引用。否则，如果n为八进制数字（0-7），则n为一个八进制转义值。
\nm|标识一个八进制转义值或一个向后引用。如果\nm之前至少有nm个获得子表达式，则nm为向后引用。如果\nm之前至少有n个获取，则n为一个后跟文字m的向后引用。如果前面的条件都不满足，若n和m均为八进制数字（0-7），则\nm将匹配八进制转义值nm。
\nml|如果n为八进制数字（0-7），且m和l均为八进制数字（0-7），则匹配八进制转义值nml。
\un|匹配n，其中n是一个用四个十六进制数字表示的Unicode字符。例如，\u00A9匹配版权符号（&copy;）。
\p{P}|小写 p 是 property 的意思，表示 Unicode 属性，用于 Unicode 正表达式的前缀。中括号内的“P”表示Unicode 字符集七个字符属性之一：标点字符。其他六个属性：L：字母；M：标记符号（一般不会单独出现）；Z：分隔符（比如空格、换行等）；S：符号（比如数学符号、货币符号等）；N：数字（比如阿拉伯数字、罗马数字等）；C：其他字符。*注：此语法部分语言不支持，例：javascript。
\\<   \\>|匹配词（word）的开始（\\<）和结束（\>）。例如正则表达式\\<the\\>能够匹配字符串"for the wise"中的"the"，但是不能匹配字符串"otherwise"中的"the"。注意：这个元字符不是所有的软件都支持的。
( )|将( 和 ) 之间的表达式定义为“组”（group），并且将匹配这个表达式的字符保存到一个临时区域（一个正则表达式中最多可以保存9个），它们可以用 \1 到\9 的符号来引用。
&#124;|将两个匹配条件进行逻辑“或”（or）运算。例如正则表达式(him&#124;her) 匹配"it belongs to him"和"it belongs to her"，但是不能匹配"it belongs to them."。注意：这个元字符不是所有的软件都支持的。
### 字符集
如果需要匹配字符串中的某一个字符，例如下面示例中需要匹配字符中单词第二个字母为c或者d的单词：
```python
import re
a = 'abc,adc,aec,ajc,acc,aic,alc'
b = re.findall('a[cd]c',a)
c = re.findall('a[^cd]c',a)
print(b)
print(c)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['adc', 'acc']
['abc', 'aec', 'ajc', 'aic', 'alc']
```
上面方法用到了普通字符和元字符的混合用法，在[cd]中c、d是或关系，加上^就是取反，[c-f]格式就是c到f字母都匹配。
### 概括字符集
在上面示例中，\d可以用[0-9]实现，\D可以用[^0-9]实现，像\d这种可以叫概括字符集，就代表一类或者一串字符了，
\w可以代表所有单词字符，\W代表非单词字符，制表符都属于非单词字符；
不敢是字符集，还是概括字符集，都只可以匹配单一的字符，例如下面示例，输出结果都是拆分后的字符：
```python
import re
a = 'Thor1Hulk2Batman9'
b = re.findall('\w',a)
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['T', 'h', 'o', 'r', '1', 'H', 'u', 'l', 'k', '2', 'B', 'a', 't', 'm', 'a', 'n', '9']
```
\s可以代表任何不可见字符，包括空格、制表符、换页符等等，或者叫空白字符。等价于[ \f\n\r\t\v]：
```python
import re
a = 'Thor &1Hulk2\nBatman\t9'
b = re.findall('\s',a)
print(b)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
[' ', '\n', '\t'] 
```
### 数量词
之前的示例中，可以看到字符匹配都是单个字符，输出结果也是单个字符，如果需要是多个，则需要用到数量词，示例中的{4}就表示四个字符，{4，7}表示4到7个字符的匹配：
```python
import re
a = 'thor342captain 342batman 2thanos 423hulk'
b = re.findall('[a-z]{4}',a)
c = re.findall('[a-z]{4,7}',a)
print(b)
print(c)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['thor', 'capt', 'batm', 'than', 'hulk']
['thor', 'captain', 'batman', 'thanos', 'hulk']
```
### 贪婪和非贪婪
默认情况下Python倾向于贪婪匹配方式，例如{4,7}，Python匹配到第三个的时候，会继续匹配直到不满足条件。
在数量词后面加上问号，就可以改成非贪婪模式：
```python
import re
a = 'thor342captain 342batman 2thanos 423hulk'
b = re.findall('[a-z]{4}',a)
c = re.findall('[a-z]{4,7}?',a)
print(b)
print(c)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['thor', 'capt', 'batm', 'than', 'hulk']
['thor', 'capt', 'batm', 'than', 'hulk']
```
可以看到c和b的输出结果是一样的。
### 匹配次数
星号：*，匹配0次或无限多次；
加号：+，匹配1次或者无限次；
问好：？，匹配0次或者1次。一般用于去重，去除重复的字母。和非贪婪的用法和意义不一样；
点：. ，匹配除换行符\n之外的其它所有字符。
示例如下：
```python
import re
a =  'Thano0Thanos1Thanoss2'
b = re.findall('Thanos*',a)
c = re.findall('Thanos+',a)
d = re.findall('Thanos?',a)
print(b)
print(c)
print(d)
```
运行后输出结果如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
['Thano', 'Thanos', 'Thanoss']
['Thanos', 'Thanoss']
['Thano', 'Thanos', 'Thanos']
```
### 边界匹配
例如下面示例，需要密码是4到8位，用下面方法可以匹配下：
```python
import re
password1 = '1234567'
password2 = '123'
password3 = '123456789'
a = re.findall('\d{4,8}',password1)
b = re.findall('\d{4,8}',password2)
c = re.findall('\d{4,8}',password3)
print(a)
print(b)
print(c)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
['1234567']
[]
['12345678']
```
上面示例中只是个寻找匹配，不是完全匹配，这里就需要用到边界匹配；
^表示从字符串开头匹配，$表示从字符串末尾匹配，一前一后就可以匹配完整的字符串：
```python
import re
password1 = '1234567'
password2 = '123'
password3 = '123456789'
a = re.findall('^\d{4,8}$',password1)
b = re.findall('^\d{4,8}&',password2)
c = re.findall('^\d{4,8}$',password3)
print(a)
print(b)
print(c)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
['1234567']
[]
[]
```
### 组
下面字符串中包括多个Thanos，需要判断字符串中是否包含三个Thanos，把Thanos用括号括起来，括起来的字符就在正则表达式中称作组，在组的后面加上数量词，表示把组的字符重复多少次：
```python
import re
a =  'ThanosThanosThanosThanos'
b = re.findall('(Thanos){3}',a)
print(b)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
['Thanos']
```
注意，之前用的[ ]，里面字符是或关系，这里的( )里面的字符是且关系。
### 匹配模式参数
之前提到过re.findall的格式：findall(pattern: AnyStr, string: AnyStr, flags: _FlagsType=...) -> List[Any]，还有一个参数flags就是匹配模式参数；
如何匹配忽略大小写，可以下模式参数下设置re.I，就可以了：
```python
import re
hero =  'Thor,Hulk\n,Iron Man,Batman'
a = re.findall('hulk.{1}',hero,re.I | re.S)
print(a)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
['Hulk\n']
```
对于模式参数，可以接收好几种模式，多种模式通过 |  来隔开，参数间是且关系。re.I表示可以不区分大小写，re.S参数加入后就可以匹配到换行符了，不加的话，匹配字符里面的.{1}就会无效。
### re.sub正则替换
re模块下有很多函数，sub用于字符串的替换，格式如下：
def sub(pattern: AnyStr, repl: AnyStr, string: AnyStr, count: int=..., flags: _FlagsType=...)
例如下面示例尝试把Thanos换成Thor：
```python
import re
hero = 'Iron Man,Captain America,Thanos,Hulk,Thanos'
real_hero = re.sub('Thanos','Thor',hero,0)
print(real_hero)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
Iron Man,Captain America,Thor,Hulk,Thor
```
可以看到成功替换了，count参数默认值是0，表示无限替换下去，就是有多少替换多少，如果设置1，那么就只替换一次了。

也可以使用下面方法去替换，调用replace函数，但是必须将值赋值给一个新的变量，因为字符串是不可变的，如果不赋值，打印出来的hero是不会改变的：
```python
import re
hero = 'Iron Man,Captain America,Thanos,Hulk,Thanos'
real_hero = hero.replace('Thanos','Thor')
print(real_hero)
```
简单的建议用replace，复杂点的可以用re.sub。

sub还有个强大的功能，就是第二个参数可以是一个函数，例如定义一个函数superhero，先空着：
```python
import re
hero = 'Iron Man,Captain America,Thanos,Hulk,Thanos'
def superhero(value):
    print(value)
real_hero = re.sub('Thanos',superhero,hero,0)
print(real_hero)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
<re.Match object; span=(25, 31), match='Thanos'>
<re.Match object; span=(37, 43), match='Thanos'>
Iron Man,Captain America,,Hulk,
```
可以看到Thanos全没了。如果把一个函数作为sub第二个参数传到参数列表，运行流程是，当sub的一个参数Thanos匹配到一个结果后，会把匹配结果传到函数里面，函数返回结果会是一个新的结果，因为我们函数没有返回值，所以Thanos就会被空字符替换。
打印出来的value是个对象，可以调用value的group方法拿到匹配的字符：
```python
import re
hero = 'Iron Man,Captain America,Thanos,Hulk,Thanos'
def superhero(value):
    matched = value.group()
    return matched + ' is not a hero'
real_hero = re.sub('Thanos',superhero,hero,0)
print(real_hero)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
Iron Man,Captain America,Thanos is not a hero,Hulk,Thanos is not a hero
```
需要根据匹配结果，做不同的操作的时候，使用到函数就比较方便了，
下面的字符串中，我们要把字符中数字大于等于5的数值改成9，小于5的改成0：
```python
import re
a = 'AF2582GELG834534'
def convert(value):
    matched = value.group()
    if int(matched) >= 5:
        return '9'
    else:
        return '0'
b = re.sub('\d',convert,a)
print(b)
```
运行输出结果：
```shell
PS D:\Python\codefile\expression> python module_1.py
AF0990GELG900900
```
matched是个字符，所以要传换成一个整形，返回值也是一样，直接数字是不行的，要是一个字符串。
### re.match和re.search
re.match格式：def match(pattern: AnyStr, string: AnyStr, flags: _FlagsType=...)
re.search格式：def match(pattern: AnyStr, string: AnyStr, flags: _FlagsType=...)
二者共同点就是只匹配一次，findall会匹配所有符合的元素。
下面示例中想匹配字符串a中的数字：
```python
import re
a = 'tdfaf48tr48qhff4j'
b = re.match('\d',a)
c = re.search('\d',a)
print(b)
print(c)
```
运行后输出如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
None
<re.Match object; span=(5, 6), match='4'>
```
match输出结果是None，说明没有匹配到，serch有匹配到，但是返回的一个match对象。
match匹配特征是从字符串开始来匹配，a字符串开头是字母，所以直接返回了一个空值；如果把字符串第一个改成数字，返回和search一样都是一个对象了；
search匹配是在字符串中搜索，找到第一个符合的条件后，就返回出结果。
上面示例中把第一个改成数字，并且用group来获取返回结果：
```python
import re
a = '7dfaf48tr48qhff4j'
b = re.match('\d',a)
c = re.search('\d',a)
print(b.group())
print(c.group())
```
运行后输出如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
7
7
```
### group分组
下面示例中，需要匹配出am和！中间的字符，包括空格：
```python
import re
a = 'I am Iron Man !'
b = re.search('am.*!',a)
print(b.group())
```
运行后输出如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
am Iron Man !
```
之前有学习过，点 . 就是匹配除换行符\n之外的其它所有字符，*就是进行多次匹配，可以看到有匹配中间结果，但是根据需要，应该只是需要中间的名字，这样的匹配方式显然不满足需求，

group可以传递参数，指定获取的组号，默认是0，之前的示例中只有一个组：am.*! ，加上括号也可以（am.*!），输出结果一样。
在上面的示例中改动下，把模糊匹配括起来作为一个组：（.*）
```python
import re
a = 'I am Iron Man !'
b = re.search('am(.*)!',a)
print(b.group(1))
print(b)
```
运行后输出如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
Iron Man
```
可以看到是预期的结果，如果group参数不设置或者设置为0，输出都是完整的匹配结果，所有设置为1。
用findall也可以实现，更加简洁:
```python
import re
a = 'I am Iron Man !'
b = re.findall('am(.*)!',a)
print(b)
```
运行后输出如下：
```shell
PS D:\Python\codefile\expression> python module_1.py
[' Iron Man ']
```
group也可以用group(0,1,2)这样用，可以用groups()这种就输出组里面的内容。
### 正则表达式学习总结
正则表达式可以完成一些字符串内置函数无法完成的功能。
#### 常用正则表达式
下面内容截取自链接：[https://www.cnblogs.com/zxin/archive/2013/01/26/2877765.html](https://www.cnblogs.com/zxin/archive/2013/01/26/2877765.html)
##### 校验数字的表达式
- 数字：\^[0-9]*$
- n位的数字：\^\d{n}$
- 至少n位的数字：\^\d{n,}$
- m-n位的数字：\^\d{m,n}$
- 零和非零开头的数字：\^(0|[1-9][0-9]*)$
- 非零开头的最多带两位小数的数字：\^([1-9][0-9]*)+(.[0-9]{1,2})?$
- 带1-2位小数的正数或负数：\^(\\-)?\d+(\\.\d{1,2})?$
- 正数、负数、和小数：\^(\\-|\\+)?\d+(\\.\d+)?$
- 有两位小数的正实数：\^[0-9]+(.[0-9]{2})?$
- 有1~3位小数的正实数：\^[0-9]+(.[0-9]{1,3})?$
- 非零的正整数：\^[1-9]\d*\$ 或者
  \^([1-9][0-9]\*){1,3}\$ 或 \^\\+?[1-9][0-9]*$
- 非零的负整数：\^\\-[1-9][]0-9"\*\$ 或 ^-[1-9]\d*\$
- 非负整数：^\d+\$ 或 \^[1-9]\d*|0$
- 非正整数：\^-[1-9]\d*|0\$ 或 ^((-\d+)|(0+))\$
- 非负浮点数：^\d+(\\.\d+)?\$ 或 \^[1-9]\d*\\.\d*|0\\.\d*[1-9]\d*|0?\\.0+|0\$
- 非正浮点数：^((-\d+(\\.\d+)?)|(0+(\\.0+)?))\$ 或 ^(-([1-9]\d*\\.\d*|0\\.\d*[1-9]\d*))|0?\\.0+|0\$
- 正浮点数：\^[1-9]\d*\\.\d*|0\\.\d*[1-9]\d*\$ 或 ^(([0-9]+\\.[0-9]\*[1-9][0-9]\*)|([0-9]\*[1-9][0-9]\*\\.[0-9]+)|([0-9]\*[1-9][0-9]\*))\$
- 负浮点数：^-([1-9]\d*\\.\d*|0\\.\d*[1-9]\d*)\$ 或 ^(-(([0-9]+\.[0-9]\*[1-9][0-9]\*)|([0-9]\*[1-9][0-9]*\\.[0-9]+)|([0-9]\*[1-9][0-9]\*)))\$
- 浮点数：^(-?\d+)(\\.\d+)?\$ 或 ^-?([1-9]\d*\\.\d*|0\\.\d*[1-9]\d*|0?\\.0+|0)$

##### 校验字符的表达式
- 汉字：\^[\u4e00-\u9fa5]{0,}$
- 英文和数字：\^[A-Za-z0-9]+\$ 或 \^[A-Za-z0-9]{4,40}$
- 长度为3-20的所有字符：^.{3,20}$
- 由26个英文字母组成的字符串：\^[A-Za-z]+$
- 由26个大写英文字母组成的字符串：\^[A-Z]+$
- 由26个小写英文字母组成的字符串：\^[a-z]+$
-  由数字和26个英文字母组成的字符串：\^[A-Za-z0-9]+$
- 由数字、26个英文字母或者下划线组成的字符串：\^\w+\$ 或 ^\w{3,20}$
- 中文、英文、数字包括下划线：\^[\u4E00-\u9FA5A-Za-z0-9_]+\$
- 中文、英文、数字但不包括下划线等符号：\^[\u4E00-\u9FA5A-Za-z0-9]+\$ 或 \^[\u4E00-\u9FA5A-Za-z0-9]{2,20}$
- 可以输入含有^%&',;=?$\"等字符：[\^\%\&'\,\;\=\?\$\\x22]+
- 禁止输入含有~的字符：[\^\~\\x22]+

##### 特殊需求表达式
- Email地址：^\w+([-+.]\w+)\*@\w+([-.]\w+)\*\\.\w+([-.]\w+)\*$
- 域名：[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(/.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+/.?
- InternetURL：[a-zA-z]+://[^\s]* 或 ^http://([\w-]+\\.)+[\w-]+(/[\w-./?%&=]*)?$
- 手机号码：^(13[0-9]|14[0-9]|15[0-9]|16[0-9]|17[0-9]|18[0-9]|19[0-9])\d{8}\$ (由于工信部放号段不定时，所以建议使用泛解析 ^([1][3,4,5,6,7,8,9])\d{9}$)
- 电话号码("XXX-XXXXXXX"、"XXXX-XXXXXXXX"、"XXX-XXXXXXX"、"XXX-XXXXXXXX"、"XXXXXXX"和"XXXXXXXX)：^(\\(\d{3,4}-)|\d{3.4}-)?\d{7,8}$ 6 国内电话号码(0511-4405222、021-87888822)：\d{3}-\d{8}|\d{4}-\d{7}
- 18位身份证号码(数字、字母x结尾)：^((\d{18})|([0-9x]{18})|([0-9X]{18}))$
- 帐号是否合法(字母开头，允许5-16字节，允许字母数字下划线)：\^[a-zA-Z][a-zA-Z0-9_]{4,15}$
- 密码(以字母开头，长度在6~18之间，只能包含字母、数字和下划线)：\^[a-zA-Z]\w{5,17}$
- 强密码(必须包含大小写字母和数字的组合，不能使用特殊字符，长度在8-10之间)：^(?=.\*\d)(?=.\*[a-z])(?=.\*[A-Z]).{8,10}\$
- 日期格式：^\d{4}-\d{1,2}-\d{1,2}
- 一年的12个月(01～09和1～12)：^(0?[1-9]|1[0-2])$
- 一个月的31天(01～09和1～31)：^((0?[1-9])|((1|2)[0-9])|30|31)$
- 钱的输入格式：有四种钱的表示形式我们可以接受:"10000.00" 和 "10,000.00", 和没有 "分" 的 "10000" 和 "10,000"：\^[1-9][0-9]\*\$
- 这表示任意一个不以0开头的数字,但是,这也意味着一个字符"0"不通过,所以我们采用下面的形式：^(0|[1-9][0-9]\*)\$
- 一个0或者一个不以0开头的数字.我们还可以允许开头有一个负号：^(0|-?[1-9][0-9]\*)\$
- 这表示一个0或者一个可能为负的开头不为0的数字.让用户以0开头好了.把负号的也去掉,因为钱总不能是负的吧.下面我们要加的是说明可能的小数部分：\^[0-9]+(.[0-9]+)?\$
- 必须说明的是,小数点后面至少应该有1位数,所以"10."是不通过的,但是 "10" 和 "10.2" 是通过的：\^[0-9]+(.[0-9]{2})?\$
- 这样我们规定小数点后面必须有两位,如果你认为太苛刻了,可以这样：\^[0-9]+(.[0-9]{1,2})?\$
- 这样就允许用户只写一位小数.下面我们该考虑数字中的逗号了,我们可以这样：\^[0-9]{1,3}(,[0-9]{3})\*(.[0-9]{1,2})?\$
- 1到3个数字,后面跟着任意个 逗号+3个数字,逗号成为可选,而不是必须：^([0-9]+|[0-9]{1,3}(,[0-9]{3})\*)(.[0-9]{1,2})?\$
-  备注：这就是最终结果了,别忘了"+"可以用"\*"替代如果你觉得空字符串也可以接受的话(奇怪,为什么?)最后,别忘了在用函数时去掉去掉那个反斜杠,一般的错误都在这里
- xml文件：^([a-zA-Z]+-?)+[a-zA-Z0-9]+\\\.[x|X][m|M][l|L]\$
- 中文字符的正则表达式：[\u4e00-\u9fa5]
- 双字节字符：[^\x00-\xff] (包括汉字在内，可以用来计算字符串的长度(一个双字节字符长度计2，ASCII字符计1))
- 空白行的正则表达式：\n\s*\r (可以用来删除空白行)
-  HTML标记的正则表达式：<(\S*?)[^>]\*>.\*?</\1>|<.\*? /> (网上流传的版本太糟糕，上面这个也仅仅能部分，对于复杂的嵌套标记依旧无能为力)
-  首尾空白字符的正则表达式：\^\s*|\s*\$或(^\s*)|(\s*$) (可以用来删除行首行尾的空白字符(包括空格、制表符、换页符等等)，非常有用的表达式)
- 腾讯QQ号：[1-9][0-9]{4,} (腾讯QQ号从10000开始)
- 中国邮政编码：[1-9]\d{5}(?!\d) (中国邮政编码为6位数字)
- IP地址：\d+\.\d+\.\d+\.\d+ (提取IP地址时有用)
