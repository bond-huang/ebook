# CLP-变量基本操作
&#8195;&#8195;变量是一个命名的可变值，可以通过引用其名称来访问或更改，变量不存储在库中，它们也不是对象。官方文档主页：[IBM i 7.3 CL 命令中的变量](https://www.ibm.com/docs/zh/i/7.3?topic=programming-variables-in-cl-commands)
## 定义变量
所有变量必须先向CL程序或过程声明（定义），然后才能被程序或过程使用。
### 声明变量的方法
有两种声明变量的方法：Declare variable及Declare file：
- 声明变量：使用`DCL`(Declare CL Variable)命令完成的，包括定义变量的属性。这些属性包括类型、长度和初始值:
    ```
    DCL  VAR(&AREA)  TYPE(*CHAR)  LEN(4)  VALUE(BOOK)
    DCL  VAR(&TMPDATE) TYPE(*CHAR) LEN(8) 
    DCL  VAR(&DATE) TYPE(*CHAR) LEN(6) 
    ```
- 声明文件：如果CL程序或过程使用文件，则必须在`DCLF`(Declare File)命令的`FILE`参数中指定文件的名称。该文件包含文件中记录的描述（格式）和记录中的字段:
    ```
    DCLF  FILE(MCGANN/GUIDE)
    ```
    - 在编译期间，`DCLF`命令为文件中定义的字段和指标隐式声明CL变量。例如，如果文件的DDS中有一个记录，其中包含两个字段（F1和F2），则程序中会自动声明两个变量`&F1`和`&F2`
    - 如果文件是在没有DDS的情况下创建的物理文件，则为整个记录声明一个变量。变量与文件同名，长度与文件的记录长度相同

声明命令必须在程序或过程中的所有其他命令之前（除了PGM命令），但它们可以以任何顺序混合。
### 使用声明CL变量命令的规则
&#8195;&#8195;`DCL`(Declare CL Variable)命令官方链接：[IBM i 7.3 Declare CL Variable (DCL)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/dcl.htm)。使用`DCL`命令时，必须使用以下规则：
- CL变量名称必须以与号`&`开头，后跟最多10个字符，并且第一个字符必须是字母，其余字符必须是字母数字。例如`&TESTVAR`
- CL变量值必须是以下之一：
    - 长达5000个字符的字符串
    - 一个压缩十进制值，总计最多15位，最多9个小数位
    - 逻辑值`0`或`1`，其中`0`表示`off`、`false`或`no`, `1`表示`on`、`true`或`yes`
    - 两个、四个或八个字节的整数值:如果为`TYPE`参数指定了`*INT`，则该值可以是负数；如果为`TYPE`参数指定了 `*UINT`，则该值必须为正或零。仅当使用`CRTCLMOD`(Create CL Module)命令或`CRTBNDCL`(Create Bound CL Program)命令编译CL源时，才能指定`LEN(8)`
    - 一个指针值，可以保存数据在存储中的位置
- 如果不指定初始值，则默认如下：
    - `0`表示十进制变量
    - 空白的字符变量
    - `0`表示逻辑变量
    - `0`表示整数变量
    - 指针变量为空
    - 对于十进制和字符类型，如果指定了初始值，不指定`LEN`参数，则默认长度与初始值的长度相同
    - 对于`*CHAR`类型，如果不指定`LEN`参数，则字符串可以长达5000个字符
    - 对于`*INT`或`*UINT`类型，如果不指定`LEN`参数，则默认长度为4
- 在程序 DCL 语句中将参数声明为变量

### 基本变量用途
&#8195;&#8195;基于变量可用于映射传递给程序的变量或操作值数组。在使用之前，必须使用`DCL`命令上的`ADDRESS`关键字或使用`%ADDRESS`内置函数来设置基础指针。设置基础指针后，变量将像局部变量一样工作。示例：
```
PGM
    DCL &AUTO  *CHAR 20
    DCL &PTR   *PTR ADDRESS(&AUTO)
    DCL &BASED *CHAR 10 STG(*BASED) BASPTR(&PTR) 
    :
    IF COND(%SST(&AUTO 1 10) *EQ &BASED) +
        THEN(CHGVAR %OFS(&PTR) (%OFS(&PTR) + 10))
    :
ENDPGM
```
示例说明：
- 示例中，基础指针`&PTR`被声明为等于`&AUTO`的地址。然后变量`&BASED`具有指针变量`&PTR`寻址的前10个字节的值
- 在条件语句中，将检查变量`&BASED`的值与变量`&AUTO`的前10个字节是否相等。如果值相同，意味着指针`&PTR`寻址`&AUTO`的第一个字节，则指针偏移量更改为变量`&AUTO`的地址字节11
- 现在变量`&BASED`的值等于变量`&AUTO`的11-20字节

### 使用定义的变量
&#8195;&#8195;定义的变量通过排除从大变量中提取值的需要，使管理控制语言 (CL) 中的复杂数据结构变得容易。已定义变量可用于以不同方式映射已定义变量的不同部分或给定变量的相同部分。官方示例如下：
```
PGM
DCL &OBJECT  *CHAR 20
DCL &OBJNAME *CHAR 10 STG(*DEFINED) DEFVAR(&OBJECT)
DCL &LIBNAME *CHAR 10 STG(*DEFINED) DEFVAR(&OBJECT 11)
:
IF COND(&LIBNAME *EQ '*LIBL     ') +
   THEN(...))
:
ENDPGM
```
示例说明：
- 变量`&OBJNAME`值取变量`&OBJECT`的前 10 个字节，变量`&LIBNAME`值取变量`&OBJECT`的后10个字节
- 使用定义的变量`&OBJNAME`和`&LIBNAME`提高了代码的可读性并使其更易于使用
- 变量`&OBJECT`为`&LIBNAME`和`&OBJNAME`变量提供存储

&#8195;&#8195;还可以使用多个定义创建相同的存储。下面示例中，变量`&BINLEN`和`&CHARLEN`都引用了相同的4字节变量 `&STRUCT`。然后程序可以使用最适合其要求的定义：
```
PGM
DCL &STRUCT  *CHAR 30
DCL &BINLEN  *INT   4 STG(*DEFINED) DEFVAR(&STRUCT)
DCL &CHARLEN *CHAR  4 STG(*DEFINED) DEFVAR(&STRUCT)
:
ENDPGM
```
&#8195;&#8195;下面示例显示了如何使用定义的变量来更改变量中的值。示例中使用`%OFFSET`内置函数和基于变量来导航库列表。这不是进行消息替换的最佳方式，但说明了已定义变量的一些功能：
```
PGM
DCL &MESSAGE  *CHAR  25 VALUE('LIBRARY NNN IS XXXXXXXXXX')
DCL &SEQUENCE *CHAR   3 STG(*DEFINED) DEFVAR(&MESSAGE  9) 
DCL &MSGLIBN  *CHAR  10 STG(*DEFINED) DEFVAR(&MESSAGE 16) 
DCL &COUNTER  *INT    2                                   
DCL &LIBL     *CHAR 165                                   
DCL &PTR      *PTR      ADDRESS(&LIBL)                    
DCL &LIBLNAME *CHAR  10 STG(*BASED) BASPTR(&PTR)          
:
RTVJOBA SYSLIBL(&LIBL)   
CHGVAR &COUNTER 0        
DOFOR &COUNTER FROM(1) TO(15)                 
   IF (&LIBLNAME *EQ '          ') THEN(LEAVE)
   CHGVAR &SEQUENCE &COUNTER                  
   CHGVAR &MSGLIBN &LIBLNAME                  
   SNDPGMMSG  MSGID(CPF9898) MSGF(QSYS/QCPFMSG) MSGDTA(&MESSAGE)
   CHGVAR %OFS(&PTR) (%OFS(&PTR) + 11)        
ENDDO                                         
:
ENDPGM
```
示例说明：
- 变量`&MESSAGE`定义了长度为25字符的文本，程序将对其中数据进行替换，替换后进行输出
- 变量`&SEQUENCE`取变量`&MESSAGE`第9个字符开始后3位，即`NNN`
- 变量`&MSGLIBN`取变量`&MESSAGE`第16个字符开始后10位，即`XXXXXXXXXX`
- 变量`&COUNTER`为两位的数字整型
- 变量`&LIBL`定义长度为165的字符
- 命令`RTVJOBA`(Retrieve Job Attributes)中选项`SYSLIBL`为`System library list`
- 通过`FOR`循环语句进行遍历，过程中进行判断，根据判断结果修改变量`&SEQUENCE`和变量`&MSGLIBN`的取值

参考链接：[Retrieve Job Attributes (RTVJOBA)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/rtvjoba.htm)
### 用于指定列表或限定名称的变量
&#8195;&#8195;变量可用于指定列表或限定名称。参数的值可以是一个列表。例如，`CHGLIBL`(Change Library List)命令需要`LIBL`参数上的库列表，每个库由空格分隔。此列表中的元素可以是变量，如果是变量，必须单独声明每个元素：示例：
```
DCL VAR(&LIB1) TYPE(*CHAR) LEN(10) VALUE(QTEMP)
DCL VAR(&LIB2) TYPE(*CHAR) LEN(10) VALUE(QGPL)
DCL VAR(&LIB3) TYPE(*CHAR) LEN(10) VALUE(HQLIB)
CHGLIBL LIBL(&LIB1 &LIB2 &LIB3)
```
变量元素不能在列表中指定为字符串。错误示例：
```
DCL   VAR(&LIBS) TYPE(*CHAR) LEN(20) +
      VALUE('QTEMP QGPL DISTLIB')
CHGLIBL LIBL(&LIBS)
```
&#8195;&#8195;当呈现为单个字符串时，系统不会将列表视为单独元素的列表，并且会发生错误。如果每个限定符都声明为单独的变量，还可以使用变量来指定限定名。示例如下：
```
DCL VAR(&PGM) TYPE(*CHAR) LEN(10)
DCL VAR(&LIB) TYPE(*CHAR) LEN(10)
CHGVAR VAR(&PGM) VALUE(MYPGM)
CHGVAR VAR(&LIB) VALUE(MYLIB)
...
DLTPGM PGM(&LIB/&PGM)
ENDPGM
```
&#8195;&#8195;上面的示例中，程序名`&PGM`和库名`&LIB`的变量是分开声明的。程序名和库名不能在一个变量中指定，错误示例如下：
```
DCL VAR(&PGM) TYPE(*CHAR) LEN(11)
CHGVAR VAR(&PGM) VALUE('MYLIB/MYPGM')
DLTPGM PGM(&PGM)
```
&#8195;&#8195;上面示例中，系统再次将值视为单个字符串，而不是两个对象（库和对象）。如果必须将限定名称作为具有字符串值的单个变量处理，则可以使用内置函数`%SUBSTRING`和`*TCAT`连接函数将对象和库名称分配给单独的变量。
### 变量中字符的大小写
&#8195;&#8195;`CL`变量中字符的大小写有限制。可用作变量的保留值（例如`*LIBL`）必须始终以大写字母表示，尤其是当它们以单引号括起来的字符串形式表示时。例如，想在命令中用变量替换库名，正确示例如下：
```
DCL VAR(&LIB) TYPE(*CHAR) LEN(10) VALUE('*LIBL')
DLTPGM &LIB/TESTPROG;
```
指定`VALUE`参数错误示例如下：
```
DCL VAR(&LIB) TYPE(*CHAR) LEN(10) VALUE('*libl')
```
注意事项：
- 如果这个`VALUE`参数没有用单引号括起来，那么可以写成小写`*libl`，因为没有单引号它会自动转换为大写
- 错误示例中没有考虑到转换为大写可能是依赖于语言的事实。记住依赖系统将值转换为大写会产生意想不到的结果

### 替换保留或数字参数值的变量
&#8195;&#8195;字符变量可用于某些命令来表示命令参数的值。某些`CL`命令允许在某些参数上使用数字或预定义（保留）值。在这种情况下，可以使用字符变量来表示命令参数的值：
- 命令上的每个参数只能接受某些类型的值。该参数可以允许整数、字符串、保留值、指定类型的变量或这些的某种混合作为值
- 如果参数允许数值（如果该值在命令中定义为 `*INT2`、`*INT4`、`*UINT2`、`*UINT4`或 `*DEC`）并且还允许保留值（前面带有星号的字符串），则可以使用一个变量作为参数的值
- 如果打算使用保留值，则必须将变量声明为`TYPE(*CHAR)`

&#8195;&#8195;`CHGOUTQ`(Change Output Queue) 命令有一个作业分隔符 `JOBSEP`参数，该参数的值可以是数字（0到9）或预定义的默认值`*SAME`。因为数字和预定义的值都是可接受的，可以编写一个`CL`源程序，用字符变量代替 `JOBSEP`值。示例如下：
```
      PGM
      DCL &NRESP *CHAR LEN(6)
      DCL &SEP *CHAR LEN(4)
      DCL &FILNAM *CHAR LEN(10)
      DCL &FILLIB *CHAR LEN(10)
      DCLF.....
      ...
LOOP: SNDRCVF.....
      IF (&SEP *EQ IGNR) GOTO END
      ELSE IF (&SEP *EQ NONE) CHGVAR &NRESP '0'
      ELSE IF (&SEP *EQ NORM) CHGVAR &NRESP '1'
      ELSE IF (&SEP *EQ SAME) CHGVAR &NRESP '*SAME'
      CHGOUTQ OUTQ(&FILLIB/&FILNAM) JOBSEP(&NRESP)
      GOTO LOOP
END:  RETURN
      ENDPGM
```
示例说明：
- 示例中，显示站用户在显示器上输入信息，描述指定输出队列所需的作业分隔符的数量
- 变量`&NRESP`是一个操作数字和预定义值的字符变量（注意使用单引号）
- `CHGOUTQ`(Change Output Queue) 命令上的`JOBSEP`参数将识别这些值，就好像它们已作为数字或预定义值输入一样
- 此程序中使用的显示文件的`DDS`应使用`VALUES`关键字将用户响应限制为`IGNR`、`NONE`、`NORM`或`SAME`
- 如果参数允许数值类型（`*INT2`、`*INT4`、`*UINT2`、`*UINT4`或 `*DEC`）并且不打算输入任何保留值（例如 `*SAME`），则可以使用小数或该参数中的整数变量

### 更改变量的值
&#8195;&#8195;使用`CHGVAR`(Change Variable)命令更改`CL`变量的值。可以更改为数。例如将变量`&INVCMPLT`设置为`0`(两种写法均可)：
```
CHGVAR (&INVCMPLT) VALUE(100)
CHGVAR &INVCMPLT  0
```
更改到另一个变量的值。例如`&VARA`设置为变量`&VARB`的值：
```
CHGVAR VAR(&VARA) VALUE(&VARB)
CHGVAR &VARA &VARB
```
对表达式求值。例如变量`&COUNT`值增加`1`：
```
CHGVAR VAR(&COUNT) VALUE(&COUNT + 1)
CHGVAR &COUNT (&COUNT + 1)
```
更改到内置函数`%SST`产生的值。例如`&VARA`设置为变量`&VARB`值的前六个字符：
```
CHGVAR VAR(&VARA) VALUE(%SST(&VARB 1 6))
```
&#8195;&#8195;更改到内置函数`%SWITCH`产生的值。例如如果作业开关`1`和`8`为`0`，作业开关`4`、`5`和`6`为`1`，则`&VARA`设置为`1`；否则，`&VARA`设置为`0`。
```
CHGVAR VAR(&VARA) VALUE(%SWITCH(0XX111X0))
```
&#8195;&#8195;更改到内置函数`%BIN`产生的值。例如变量`&VARB`的前八个字符被转换为等效的十进制并存储在变量`&VARA`中，如下所示：
```
CHGVAR VAR(&VARA) VALUE(%BIN(&VARB 1 8))
```
&#8195;&#8195;更改到内置函数`%CHECK`产生的值。下面示例中，检查变量`&VARB`中的值，并将最左边不是数字的字符的位置存储在变量 `&VARA`中，如果变量`&VARB`中的所有字符都是数字，则将零值存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%CHECK('0123456789' &VARB))
```
&#8195;&#8195;更改到内置函数`%CHECKR`产生的值。下面示例中，检查变量`&VARB`中的值，并且将不是星号 `*`的最右边字符的位置存储在变量`&VARA`中。如果变量`&VARB`中的所有字符都是星号，则将零值存储在变量`&VARA`中。：
```
CHGVAR VAR(&VARA) VALUE(%CHECKR('*' &VARB))
```
&#8195;&#8195;更改到内置函数`%SCAN`产生的值。下面示例中，扫描变量`&VARB`中的值，并将最左边的句点 `.`字符的位置存储在变量`&VARA`中。如果变量`&VARB`中没有句点字符，则将零值存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%SCAN('.' &VARB))
```
&#8195;&#8195;更改到内置函数`%TRIM`产生的值。下面示例中，变量`&VARB`中的前导星号 `*`和空白字符将被剪掉，结果字符串将存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%TRIM(&VARB '* '))
```
&#8195;&#8195;更改到内置函数`%TRIML`产生的值。下面示例中，变量`&VARB`中的前导空白字符将被剪掉，结果字符串将存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%TRIML(&VARB))
```
&#8195;&#8195;更改到内置函数`%TRIMR`产生的值。下面示例中，变量`&VARB`中的尾随星号 (*) 字符将被剪掉，生成的字符串将存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%TRIMR(&VARB '*'))
```
&#8195;&#8195;更改到内置函数`%CHAR`产生的值。下面示例中，变量`&VARB`将被转换为字符格式，生成的字符串将存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%CHAR(&VARB))
```
&#8195;&#8195;更改到内置函数`%UPPER`产生的值。下面示例中，变量`&VARB`中的小写字母将被转换为大写字母，生成的字符串将存储在变量`&VARA`中：
```
CHGVAR VAR(&VARA) VALUE(%UPPER(&VARB))
```
&#8195;&#8195;更改到内置函数`%SIZE`产生的值。例如变量`&VARB`占用的字节数将存储在变量 `&VARA` 中，如下所示：
```
CHGVAR VAR(&VARA) VALUE(%SIZE(&VARB))
```
&#8195;&#8195;更改到内置函数`%PARMS`产生的值。例如传递给程序的参数数量将存储在变量`&VARA` 中，如下所示：
```
CHGVAR VAR(&VARA) VALUE(%PARMS())
```
&#8195;&#8195;`CHGVAR`命令也可用于检索和更改本地数据区。下面示例中，命令清空本地数据区的10个字节并检索部分本地数据区：
```
CHGVAR %SST(*LDA 1 10) ' '
CHGVAR &A %SST(*LDA 1 10)
```
#### 从值到变量有效赋值

下表显示了从值（文字或变量）对变量的有效赋值：

| 变量               | 逻辑值 | 字符值 | 十进制值 | 有符号整数值 | 无符号整数值 |
| :-----------------:| :----: | :----: | :------: | :----------: | :----------: |
| **逻辑变量**       |   X    |        |          |              |              |
| **字符变量**       |   X    |   X    |    X     |      X       |      X       |
| **十进制变量**     |        |   X    |    X     |      X       |      X       |
| **有符号整数变量** |        |   X    |    X     |      X       |      X       |
| **无符号整数变量** |        |   X    |    X     |      X       |      X       |

注意事项：
- 为字符变量指定数值时，请记住以下几点：
  - 字符变量的值是右对齐的，如有必要，用前导零填充
  - 必要时，字符变量必须足够长以包含小数点和减号`-`
  - 使用时，减号`-`将放置在值的最左侧位置

&#8195;&#8195;例如，`&VARA`是要更改为十进制变量`&VARB`的值的字符变量。`&VARA`的长度为6。`&VARB`的长度为5，小数位为2。`&VARB`的当前值为`123`，`&VARA`的结果值为`123.00`。

为数值变量指定字符值时，请记住以下几点：
- -小数点由字符值中小数点的位置决定。如果字符值不包含小数点，则小数点放在值的最右边
- 字符值的左侧可以包含减号`-`或加号`+`；中间不允许有空格。如果字符值没有符号，则假定该值为正
- 如果字符值包含的小数点右侧的数字多于数字变量中的数字，则如果是十进制变量，则截断数字，如果是整数变量，则四舍五入。如果多余的数字在小数点的左边，它们不会被截断并且会发生错误

&#8195;&#8195;例如，`&VARC`是一个十进制变量，要更改为字符变量`&VARD`的值。`&VARC` 的长度为 5，小数点后 2 位。`&VARD` 的长度为10，其当前值为`+123.1bbbb`（其中 b=空白）。`&VARC` 的结果值为`123.10`。
### 命令参数的尾随空格
&#8195;&#8195;在某些命令参数中，可以定义如何处理尾随空格。一些命令参数使用`VARY(*YES)`的参数值定义。此参数值导致传递的值的长度为单引号之间的字符数：
- 当`CL`变量用于指定以这种方式定义的参数的值时，系统会在确定要传递给命令处理器程序的变量长度之前删除尾随空格
- 如果存在尾随空格并且对参数很重要，则必须采取特殊操作以确保传递的长度包括它们。大多数命令参数的定义和使用方式不会导致这种情况发生。定义可能发生这种情况的参数的一个示例是`OVRDBF`(Override with Database File)命令上的`POSITION`参数的值
- 当这种情况发生时，通过构造一个命令字符串，用单引号将参数值分隔，并将该字符串传递给`QCMDEXC`或`QCAPCMD`进行处理，这些参数就可以得到用户想要的结果

&#8195;&#8195;下面是一个程序示例，该程序可用于运行`OVRDBF`(Override with Database File)命令，以便将尾随空格作为键值的一部分。相同的技术可用于其他命令，这些命令具有使用参数`VARY(*YES)`; 定义的参数；尾随空格必须与参数一起传递：
```
             PGM        PARM(&KEYVAL &LEN)
 /*   PROGRAM TO SHOW HOW TO SPECIFY A KEY VALUE WITH TRAILING        */
 /*      BLANKS AS PART OF THE POSITION PARAMETER ON THE OVRDBF       */
 /*      COMMAND IN A CL PROGRAM.                                     */
 /*   THE KEY VALUE ELEMENT OF THE POSITION PARAMETER OF THE OVRDBF   */
 /*      COMMAND IS DEFINED USING THE VARY(*YES) PARAMETER.           */
 /*      THE DESCRIPTION OF THIS PARAMETER ON THE ELEM COMMAND        */
 /*      DEFINITION STATEMENT SPECIFIES THAT IF A PARAMETER           */
 /*      DEFINED IN THIS WAY IS SPECIFIED AS A CL VARIABLE THE        */
 /*      LENGTH IS PASSED AS THE VARIABLE WITH TRAILING BLANKS        */
 /*      REMOVED. A CALL TO QCMDEXC USING APOSTROPHES TO DELIMIT      */
 /*      THE LENGTH OF THE KEY VALUE CAN BE USED TO CIRCUMVENT        */
 /*      THIS ACTION.                                                 */
 /*    PARAMETERS--                                                   */
            DCL        VAR(&KEYVAL) TYPE(*CHAR) LEN(32)  /*  THE VALUE +
                         OF THE REQUESTED KEY. NOTE IT IS DEFINED AS +
                         32 CHAR.  */
            DCL        VAR(&LEN) TYPE(*INT)              /*  THE LENGTH +
                         OF THE KEY VALUE TO BE USED. ANY VALUE OF  +
                         1 TO 32 CAN BE USED  */
 /*   THE STRING TO BE FINISHED FOR THE OVERRIDE COMMAND TO BE        */
 /*        PASSED TO QCMDEXC (NOTE 2 APOSTROPHES TO GET ONE).         */
            DCL        VAR(&STRING) TYPE(*CHAR) LEN(100) +
                VALUE('OVRDBF FILE(X3) POSITION(*KEY 1 FMT1  '' ')
 /*   POSITION MARKER  123456789 123456789 123456789 123456789       */
            DCL        VAR(&END) TYPE(*DEC) LEN(15 5)  /*  A VARIABLE +
                         TO CALCULATE THE END OF THE KEY IN &STRING  */
 
            CHGVAR     VAR(%SST(&STRING 40 &LEN)) VALUE(&KEYVAL)  /*  +
                         PUT THE KEY VALUE INTO COMMAND STRING FOR +
                         QCMDEXC IMMEDIATELY AFTER THE APOSTROPHE.  */
            CHGVAR     VAR(&END) VALUE(&LEN + 40)  /*  POSITION AFTER +
                         LAST CHARACTER OF KEY VALUE  */
            CHGVAR     VAR(%SST(&STRING &END 2)) VALUE('')')  /*  PUT +
                         A CLOSING APOSTROPHE & PAREN TO END  +
                         PARAMETER  */
            CALL       PGM(QCMDEXC) PARM(&STRING 100)  /*  CALL TO +
                         PROCESS THE COMMAND   */
            ENDPGM
```
注意事项：
- 如果使用`VARY(*YES)`和`RTNVAL(*YES)`并传递`CL`变量，则传递变量的长度不是`CL`变量中数据的长度

## 待补充