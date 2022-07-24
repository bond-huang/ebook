# CLP-检索可用作变量的值
官方文档链接：[IBM i 7.4 Retrieving values that can be used as variables](https://www.ibm.com/docs/en/i/7.4?topic=cp-retrieving-values-that-can-be-used-as-variables)
## 检索系统值
&#8195;&#8195;IBM提供了几种类型的系统值。例如，`QDATE`和`QTIME`是日期和时间系统值，在操作系统启动时设置它们。可以将系统值带入用户的程序或过程，并使用`RTVSYSVAL`(Retrieve System Value)命令将它们作为变量进行操作，示例：
```
RTVSYSVAL  SYSVAL(system-value-name)  RTNVAR(CL-variable-name)
```
示例说明：
- `RTNVAR`参数指定CL程序或过程中要接收系统值的变量名称
- 变量的类型必须与系统值的类型相匹配
    - 对于字符和逻辑系统值，CL变量的长度必须等于值的长度
    - 对于十进制值，变量的长度必须大于或等于系统值的长度

### 检索QTIME系统值
官方示例如下：
```
PGM
DCL  VAR(&PWRDNTME)  TYPE(*CHAR)  LEN(6)  VALUE('183030')
DCL  VAR(&TIME)  TYPE(*CHAR)  LEN(6)
RTVSYSVAL  SYSVAL(QTIME)  RTNVAR(&TIME)
IF  (&TIME *GT &PWRDNTME)  THEN(DO)
SNDBRKMSG('Powering down in 5 minutes.  Please sign off.')
PWRDWNSYS  OPTION(*CNTRLD)  DELAY(300)  RESTART(*NO)  +
    IPLSRC(*PANEL)
ENDDO
ENDPGM
```
示例说明：
- 定义了变量`&PWRDNTME`，值为`183030`，即`18:30:30`，格式为`HH:MM:SS`
- 定义了变量`&TIME`，然后检索系统值`QTIME`，并将值赋给变量`&TIME`
- 比较两个变量值，如果`&TIME`大于`&PWRDNTME`，向系统发出将要关机消息，然后执行关机命令

### 将QDATE检索到CL变量中
&#8195;&#8195;在许多应用程序中，可能希望通过检索系统值`QDATE`并将其放入变量中，从而在程序或过程中使用当前日期。可能还想更改该日期的格式以在程序或过程中使用。
#### CVTDAT命令
使用`CVTDAT`(Convert Date)命令来转换CL程序或过程中的日期格式：
- 系统日期的格式是系统值`QDATFMT`：
    - `QDATFMT`的包含值因国家或地区而异。例如，`062489`是1989年6月24日的`MDY`(月日年)格式
    - 可以将此格式更改为`YMD`、`DMY`或`JUL`(Julian)格式
    - 对于`Julian`，`QDAY`值是一个从`001`到`366`的三字符值。它用于确定两个日期之间的天数
- 还可以`CVTDAT`命令删除日期分隔符或更改日期分隔符

`CVTDAT`命令格式如下：
```
CVTDAT  DATE(date-to-be-converted) TOVAR(CL-variable) +
        FROMFMT(old-format) TOFMT(new-format) +
        TOSEP(new-separators)
```
格式说明：
- `DATE`参数可以指定要转换的常量或变量
- 日期转换后，它被放置在以`TOVAR`参数命名的变量中

&#8195;&#8195;在下面示例中，变量`&DATE`中的日期格式为`MDY`，被更改为`DMY`格式并放置在变量`&CVTDAT`中，使用的日期分隔符与系统值`QDATSEP`中指定的保持一致：
```
CVTDAT  DATE(&DATE) TOVAR(&CVTDAT) FROMFMT(*MDY) TOFMT(*DMY)
        TOSEP(*SYSVAL)
```
&#8195;&#8195;在创建对象或添加使用日期作为其名称一部分的成员时，`CVTDAT`命令非常有用。例如，假设必须使用当前系统日期将成员添加到文件中。此外，假设当前日期为`MDY`格式，并将转换为`Julian`格式。如果当前日期是1988年1月5日，则添加的成员将命名为`MBR88005`：
```
PGM
DCL &DATE6 *CHAR  LEN(6)
DCL &DATE5  *CHAR  LEN(5)
RTVSYSVAL  QDATE  RTNVAR(&DATE6)
CVTDAT  DATE(&DATE6)  TOVAR(&DATE5)  TOFMT(*JUL)  TOSEP(*NONE)
ADDPFM  LIB1/FILEX  MBR('MBR'  *CAT  &DATE5)
. . .
ENDPGM
```
转换时间格式时注意事项：
- `DATE`参数中值的长度和`TOVAR`参数中变量的长度必须与日期格式兼容。如果转换后的日期比变量短，则在右侧用空格填充。`TOVAR`参数上的变量长度必须至少为：
    - 对于两位数年份的`Non-Julian`日期：
        - 不使用分隔符时使用六个字符，例如`July 28, 1978`为`072878`
        - 使用分隔符时使用八个字符，例如`July 28, 1978`为`07-28-78`
    - 对于四位数年份的`Non-Julian`日期：
        - 不使用分隔符时使用八个字符，例如`July 28, 1978`为`07281978`
        - 使用分隔符时使用十个字符，例如`July 28, 1978`为`07-28-1978`
    - 对于两位数年份的`Julian`日期：
        - 不使用分隔符时使用五个字符，例如`December 31, 1996`为`96365`
        - 使用分隔符时使用六个字符，例如`December 31, 1996`为`96-365`
    - 对于四位数年份的`Julian`日期：
        - 不使用分隔符时使用七个字符，例如`February 4, 1997`为`1997035`
        - 使用分隔符时使用八个字符，例如`February 4, 1997`为`1997-035`
- 在除`Julian`之外的所有日期格式中，月和日都是`2`字节字段，无论它们包含什么值。年份可以是`2`字节或 `4`字节字段。所有转换后的值都右对齐，并在必要时用前导零填充
- 在`Julian`格式中，`day`是`3`字节字段，`year`是`2`字节或`4`字节字段。所有转换后的值都右对齐，并在必要时用前导零填充

官方替代程序示例如下：
```
PGM
DCL  &LILDATE  *INT   LEN(4)
DCL  &PICTSTR  *CHAR  LEN(5)  VALUE(YYDDD)
DCL  &JULDATE  *CHAR  LEN(5)
DCL  &SECONDS  *CHAR  8       /*  Seconds from CEELOCT            */
DCL  &GREG     *CHAR  23      /*  Gregorian date from CEELOCT     */
                              /*                                  */
CALLPRC  PRC(CEELOCT)         /* Get current date and time        */ + 
         PARM(&LILDATE        /* Date in Lilian format            */ + 
              &SECONDS        /* Seconds field will not be used   */ + 
              &GREG           /* Gregorian field will not be used */ + 
              *OMIT)          /* Omit feedback parameter          */ + 
                              /*  so exceptions are signalled     */   
CALLPRC  PRC(CEEDATE) +                                               
         PARM(&LILDATE        /* Today's date     */ +                  
              &PICTSTR        /* How to format    */ +                  
              &JULDATE        /* Julian date      */ +                  
              *OMIT)                                
ADDPFM   LIB1/FILEX  MBR('MBR'  *CAT   &JULDATE)
ENDPGM
```
示例说明：
- 示例是一个`CVTDAT`命令的替代程序。使用`ILE`绑定API，`CEELOCT`(Get Current Local Time)，将日期转换为`Julian`格式
- 要创建此程序，必须单独使用`CRTBNDCL`(Create Bound Control Language Program)命令，或同时使用`CRTCLMOD`(Create Control Language Module)命令和`CRTPGM`(Create Program)命令

## 检索配置源
&#8195;&#8195;使用`RTVCFGSRC`(Retrieve Configuration Source)命令，可以生成CL命令源以创建现有配置对象并将源放在源文件成员中。生成的CL命令源可用于以下目的：
- 在系统之间迁移
- 维护现场配置
- 保存配置（不使用`SAVSYS`）

### RTVCFGSRC命令
命令`RTVCFGSRC`示例如下：
```
RTVCFGSRC   CFGD(CTL*)  CFGTYPE(*CTLD)
            SRCMBR(CTLS)  RTVOPT(*OBJ)
```
示例说明：
- 将CL源语句放在源文件`QCLSRC`的文件成员`CTLS`中
- 这些源语句可用于为名称以`CTL`开头的所有现有控制器重新创建对象描述

命令参考链接：[Retrieve Configuration Source(RTVCFGSRC)](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/rtvcfgsrc.htm)
## 检索配置状态
&#8195;&#8195;使用`RTVCFGSTS`(Retrieve Configuration Status)命令，可以让应用程序能够从三个配置对象中检索配置状态：`line`、`controller`和`device`。
### RTVCFGSTS命令
示例检索行配置描述`ND01`的配置状态，以在CL变量`&STSCODE`中使用：
```
RTVCFGSTS   CFGD(ND01)  CFGTYPE(*LIN)  STSCDE(&STSCODE)
```
命令参考链接：[Retrieve Configuration Status(RTVCFGSTS)](https://www.ibm.com/docs/zh/i/7.5?topic=ssw_ibm_i_75/cl/rtvcfgsts.htm)
## 检索网络属性