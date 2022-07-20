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