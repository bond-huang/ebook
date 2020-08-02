# Shell-退出码_test_expr查询表
整理Linux退出状态码、test命令功能表、双括号命令符号以及运算expr的表格，方便查阅。   
基本上是摘自书本《Linux命令行与shell脚本编程大全》中的表格。

### Linux中退出状态码
Linux中退出状态码：     
状态码|描述
:---|:---
0|命令成功结束
1|一般性未知错误
2|不适合的shell命令
126|命令不可执行
127|没找到命令
128|无效的退出参数
128+x|与Linux信号x相关的严重作为
130|通过Ctrl+C终止的命令
255|正常范围之外的退出状态码

### test命令比较查询表
##### 数值比较功能对照表
比较方式|描述
:---|:---
n1 -eq n2|检查n1是否与n2相等
n1 -ge n2|检查n1是否大于或等于n2
n1 -gt n2|检查n1是否大于n2
n1 -le n2|检查n1是否小于或等于n2
n1 -lt n2|检查n1是否小于n2
n1 -ne n2|检查n1是否不能于n2

##### 字符串比较测试表
比较方式|描述
:---|:---
str1 = str2|检查str1是否和str2相同
str1 != str2|检查str1是否和str2不同
str1 < str2|检查str1是否比str2小
str1 > str2|检查str1是否比str2大
-n str1|检查str1长度是否非0
-z str1|检查str1长度是否为0

##### test命令文件比较功能
比较方式|描述
:---|:---
-d file|检查file是否存在并是一个目录
-e file|检查file是否存在
-f file|检查file是否存在并是一个文件
-r file|检查file是否存在并可读
-s file|检查file是否存在并非空
-w file|检查file是否存在并可写
-x file|检查file是否存在并可执行
-O file|检查file是否存在并属当前用户所有
-G file|检查file是否存在并且默认组于当前用户相同
file1 -nt file2|检查file1是否比file2新
file1 -ot file2|检查file1是否比file2旧

### 双括号命令符号
除了test命令提供的标准数学运算符，双括号允许用户再比较过程中使用高级数学表达式。     
符号|描述
:---|:---
val++|后增
val--|后减
++val|先增
--val|先减
!|逻辑求反
~|位求反
\*\*|幂运算
<<|左位移
\>>|右位移
&|位布尔和
&#124;|位布尔或
&&|逻辑和
&#124;&#124;|逻辑或

### expr运算查询表
操作符|描述
:---|:---
ARG1 &#124; ARG2|如果ARG1既不是null也不是零值，返回ARG1；否则返回ARG2
ARG1 & ARG2|如果没有参数是null或零值，返回ARG1；否则返回0
ARG1 < ARG2|如果ARG1小于ARG2，返回1；否则返回0
ARG1 <= ARG2|如果ARG1小于或等于ARG2，返回1；否则返回0
ARG1 = ARG2|如果ARG1等于ARG2，返回1；否则返回0
ARG1 != ARG2|如果ARG1不等于ARG2，返回1；否则返回0
ARG1 >= ARG2|如果ARG1大于或等于ARG2，返回1；否则返回0
ARG1 > ARG2|如果ARG1大于ARG2，返回1；否则返回0
ARG1 + ARG2|返回ARG1和ARG2的算术运算和
ARG1 - ARG2|返回ARG1和ARG2的算术运算差
ARG1 &#42; ARG2|返回ARG1和ARG2的算术运算乘积
ARG1 / ARG2|返回ARG1和ARG2的算术运算商
ARG1 % ARG2|返回ARG1和ARG2的算术运算余数
STRING : REGEXP|如果REGEXP匹配到STRING中某个模式，返回该模式匹配
match STRING REGEXP|如果REGEXP匹配到STRING中某个模式，返回该模式匹配
substr STRING POS LENGTH|返回起始位置为POS(1开始计数)及长度为LENGTH个字符的子字符串
index STRING CHARS|返回在STRING中找到CHARS字符串的位置；否则返回0
length STRING|返回字符串STRING的数值长度
&#43; TOKEN|将TOKEN解析成字符串，即使是个关键字
(EXPRESSION)|返回EXPRESSION的值
