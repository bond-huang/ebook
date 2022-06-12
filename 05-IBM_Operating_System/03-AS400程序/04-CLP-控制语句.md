# CLP-控制语句
官方文档主页：[Controlling processing within a CL program or CL procedure](https://www.ibm.com/docs/en/i/7.4?topic=programming-controlling-processing-within-cl-program-cl-procedure)
## GOTO命令
GOTO命令处理无条件分支，官方文档：
- [GOTO command and command labels in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=cpwcpcp-goto-command-command-labels-in-cl-program-procedure)
- [IBM i 7.4 Go To (GOTO)](https://www.ibm.com/docs/en/i/7.4?topic=ssw_ibm_i_74/cl/goto.htm)

### GOTO命令说明
标准格式：
```
GOTO CMDLBL(label)
```
命令说明：
- 每当遇到`GOTO`命令时，程序处理过程将被定向到程序或过程的另一部分（由label标识）
- 在分支到标记语句之后，处理从该语句开始并按连续顺序继续
- 它不会返回到`GOTO`命令，除非由另一条指令特别指示返回。也可以定向到向前或向后的分支
- 不能使用`GOTO`命令转到程序或过程之外的标签，也不能使用`GOTO`命令分支进入或退出程序中定义的子例程

### GOTO命令示例
A的值小于30就回到`LOOP`这个Label中更改A的值：
```
LOOP: CHGVAR &A (&A + 1)
      IF (&A *LT 30) THEN(GOTO LOOP)
```
满足IF条件定向到`END`这个label，然后结束程序：
```
PGM                                                                   
     DCL        VAR(&DATE) TYPE(*CHAR) LEN(6)                             
     RTVSYSVAL  SYSVAL(QDATE) RTNVAR(&DATE)                   
     IF   (%SST(&DATE 1 2) *NE '01') THEN(GOTO END)  
     ...
END: ENDPGM  
```
回到`GOTO`所在的Label进行循环：
```
PGM
        ...
START:  SNDRCVF RCDFMT(MENU)
        IF (&RESP=1) THEN(CALL CUS210)
        ...
        GOTO START
        ...
ENDPGM
```
## IF及ELSE命令
`IF`命令用于声明一个条件，如果该条件为真，则指定要运行的程序或过程中的一个语句或一组语句。     
`ELSE`命令可与`IF`命令一起使用，以指定在`IF`命令表示的条件为假时要运行的语句或语句组。
官方参考链接：
- [IF command in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=procedure-if-command-in-cl-program)
- [Embedded IF commands in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=procedure-embedded-if-commands-in-cl-program)
- [If (IF)](https://www.ibm.com/docs/en/i/7.4?topic=ssw_ibm_i_74/cl/if.htm)
- [ELSE command in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=procedure-else-command-in-cl-program)
- [Else (ELSE)](https://www.ibm.com/docs/en/i/7.4?topic=ssw_ibm_i_74/cl/else.htm)

### IF命令
IF命令格式：
```
IF COND(logical-expression) THEN(CL-command)
```
说明：
- `COND`参数上的逻辑表达式可以是单个逻辑变量或常量，或者必须描述两个或多个操作数之间的关系；然后表达式判断`true`或`false`：
    - 如果逻辑表达式描述的条件被评估为`true`，则处理`THEN`参数上的CL命令。可以是单个或一组命令
    - 如果条件不成立，则运行下一个顺序命令
- 命令名称`IF`和关键字`COND`或值之间需要空格。关键字（如果指定）和包含值的左括号之间不允许有空格
- `COND`和`THEN`都是命令中的关键字，位置输入可以省略

基本格式示例如下：
```
IF COND(&RESP=1) THEN(CALL CUS210)
IF (&A *EQ &B) THEN(GOTO LABEL)
IF (&A=&B) GOTO LABEL
```
嵌入`CHGVAR`命令和`DO`命令：
```
IF (&A *EQ &B) THEN(CHGVAR &A (&A+1))
IF (&B *EQ &C) THEN(DO)
                ...
                ENDDO
```
多个条件判断示例：
```
PGM                                                                    
    DCL      VAR(&DATE) TYPE(*CHAR) LEN(6)                               
    RTVSYSVAL  SYSVAL(QDATE) RTNVAR(&DATE)    
    IF ((%SST(&DATE 1 2) *EQ '03') +
        *OR (%SST(&DATE 1 2) *EQ '06') +
        *OR (%SST(&DATE 1 2) *EQ '09') +
        *OR (%SST(&DATE 1 2) *EQ '12'))  THEN(DO)
        CHGVAR  VAR(&TMPDATE) VALUE(%SST(&DATE 1 +             
            2)||'/'||'21'||'/'||%SST(&DATE 5 2))         
    ENDDO
```
### ELSE命令
&#8195;&#8195;如果关联的`IF`命令的条件为`False`，则`ELSE`命令是一种指定替代处理的方法。如果使用`ELSE`命令，则处理逻辑会发生变化，示例如下：
```
IF (&A=&B) THEN(CALLPRC PROCA)
ELSE CMD(CALLPRC PROCB)
CHGVAR &C 8
```
示例说明：
- 如果`&A=&B`，则调用`PROCA`，而不调用`PROCB`
- 如果表达式`&A=&B`不为`Ture`，则调用`PROCB`

&#8195;&#8195;每个`ELSE`命令前面必须有一个关联的`IF`命令。如果存在嵌套级别的`IF`命令，则每个`ELSE`命令都与尚未与另一个`ELSE`命令匹配的最内层`IF`命令匹配。示例如下：
```
IF ... THEN ...
IF ...THEN(DO)
        IF ...THEN(DO)
                ...
        ENDDO
        ELSE DO
                IF ...THEN(DO)
                    ...
                        ENDDO
                ELSE DO
                    ...
                        ENDDO
        ENDDO
ELSE IF ... THEN ...
IF ... THEN ...
IF ... THEN ...
```
&#8195;&#8195;`ELSE`命令可用于测试一系列互斥选项。在下面的示例中，在第一次成功的`IF`测试之后，处理嵌入式命令并且过程处理`RCLRSC`命令：
```
IF  COND(&OPTION=1) THEN(CALLPRC PRC(ADDREC))
 ELSE    CMD(IF COND(&OPTION=2) THEN(CALLPRC PRC(DSPFILE)))
  ELSE   CMD(IF COND(&OPTION=3) THEN(CALLPRC PRC(PRINTFILE)))
    ELSE CMD(IF COND(&OPTION=4) THEN(CALLPRC PRC(DUMP)))
RCLRSC
RETURN
```
### 嵌入式IF命令
&#8195;&#8195;`IF`命令可以嵌入到另一个`IF`命令中。当要在真实评估下处理的命令（放置在THEN参数上的CL命令）本身是另一个`IF`命令时，可能会嵌入`IF`命令：
```
IF (&A=&B) THEN(IF (&C=&D) THEN(GOTO END))
GOTO START
```
在运行某个命令或命令组之前必须满足几个条件时，嵌入式`IF`可能很有用：
- 上面例子中，如果第一个表达式为真，则系统读取第一个`THEN`参数；
- 如果`&C=&D`表达式被评估为真，系统将处理第二个`THEN`参数`GOTO END`中的命令
- 两个表达式都必须为真才能处理`GOTO END`命令
- 如果其中一个为假，则运行`GOTO START`命令
- 请注意使用括号来组织表达式和命令

&#8195;&#8195;在`CL`编程中最多允许25级这样的嵌入。随着嵌入级别的增加和逻辑变得更加复杂，可能希望以自由形式设计输入代码以阐明关系。示例：
```
PGM
DCL &A *DEC 1
DCL &B *CHAR 2
DCL &RESP *DEC 1
IF (&RESP=1) +
    IF (&A=5) +
          IF (&B=NO) THEN(DO)
                       ...
                       ENDDO
CHGVAR &A VALUE(8)
CALL PGM(DAILY)
ENDPGM
```
示例说明：
- 前面的`IF`系列作为一个嵌入式命令处理
- 每当任何一个`IF`条件失败时，处理都会分支到代码的其余部分（CHGVAR)和后续命令）
- 如果此代码的目的是累积一系列条件，所有这些条件都必须为真，`Do`组才能处理，那么使用`*AND`在一个命令中使用多个表达式可以更容易地对其进行编码

&#8195;&#8195;在某些情况下，分支必须根据哪个条件失败而有所不同。可以通过为每个嵌入式`IF`命令添加一个`ELSE`命令来完成此操作：
```
PGM
DCL &A ...
DCL &B ...
DCL &RESP ...
IF (&RESP=1) +
      IF (&A=5) +
             IF (&B=NO) THEN(DO)
                           ...
                           SNDPGMMSG ...
                           ...
                          ENDDO
             ELSE CALLPRC PROCA
      ELSE CALLPRC PROCB
CHGVAR &A 8
CALLPRC PROC(DAILY)
ENDPGM
```
示例中：
- 如果所有条件都为真，则处理`SNDPGMMSG`命令，然后处理`CHGVAR`命令
- 如果第一个条件`&RESP=1`和第二个条件`&A=5`为真，但第三个`&B=NO`为假，则调用`PROCA`；当 PROCA 返回时，处理`CHGVAR`命令
- 如果第二个条件失败，则调用`PROCB`（`&B=NO`未测试），然后调用`CHGVAR`命令
- 如果`&RESP`不等于`1`，则立即处理`CHGVAR`命令
- `ELSE`命令已用于为每个测试提供不同的分支

以下三个示例是前面示例中嵌入的`IF`命令的正确等价语法：
```
IF (&RESP=1) THEN(IF (&A=5) THEN(IF (&B=NO) THEN(DO)))
IF (&RESP=1) THEN +
           (IF (&A=5) THEN +
                         (IF (&B=NO) THEN(DO)))
IF (&RESP=1) +
        (IF (&A=5) +
              (IF (&B=NO) THEN(DO)))
```
## DO命令和组
官方参考链接：[DO command and DO groups in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=cpwcpcp-do-command-do-groups-in-cl-program-procedure)
### DO命令和组
&#8195;&#8195;`DO`命令允许用户一起处理一组命令。组定义为`DO`命令和相应的结束执行组`ENDDO`命令之间的所有命令。`DO`组最常与`IF`、`ELSE`或`MONMSG`命令相关联。这是`DO`组的示例：
```
IF (&A=&B) THEN(DO)
            ...
            ENDDO
...
ENDPGM
```
示例说明：
- 如果逻辑表达式`&A=&B`为真，则处理`DO`组
- 如果表达式不为真，则处理在`ENDDO`命令之后的命令；`DO`组被跳过

&#8195;&#8195;下面示例中，如果`&A`不等于`&B`，则系统调用`PROCB`。不会调用 `PROCA`，也不会处理`DO`组中的任何其他命令：
```
IF (&A=&B) THEN(DO)
            CALLPRC PROCA
            CHGVAR &A &B
            SNDPGMMSG...
            ENDDO
CALLPRC PROCB
CHVAR &ACCTS &B
```
&#8195;&#8195;`DO`组可以嵌套在其他`DO`组中，最多嵌套25级。下面示例有三层嵌套。注意每个`DO`组是如何由`ENDDO`命令完成的：
```
PGM
IF (&A=&B) DO
           CALL PGMA
           IF (&A=6) DO
                    CHGVAR &A 33
                    CALL PGMB
                    IF (&AREA=YES) DO
                                   CHGVAR &AREA NO
                                   CHGVAR &P (&P+1)
                                   ENDDO
                    CALLPRC ACCTSPAY
                    ENDDO
           ENDDO
CALL PGMC
ENDPGM
```
示例说明：
- 示例中，如果第一个嵌套中的`&A`不等于`6`，则调用`PGMC`
- 如果`&A`等于`6`，则处理第二个`DO`组中的语句
- 如果第二个`DO`组中的`&AREA`不等于`YES`，则调用过程`ACCTSPAY`，因为处理移动到`DO`组之后的下一个命令

### 显示DO和SELECT嵌套级别
&#8195;&#8195;一些CL源程序包含`DO`命令或`SELECT`命令，其中这些命令嵌套了好几层。例如，一个`DO`命令和相应的`ENDDO`命令之间可以是一个`DOFOR`和另一个`ENDDO`命令：
- CL编译器最多支持`DO`命令和`SELECT`命令的25级嵌套
- 使用多级`DO`命令或`SELECT`命令嵌套更改CL源代码可能很困难
- 必须将每种类型的`DO`命令与匹配的`ENDDO`命令正确匹配
- 还必须将每个`SELECT`命令与匹配的`ENDSELECT`命令匹配

&#8195;&#8195;CL编译器提供了一个选项来显示CL编译器列表中`DO`、`DOFOR`、`DOUNTIL`、`DOWHILE`和`SELECT`命令的嵌套级别：
- 为创建CL程序`CRTCLPGM`命令、创建CL模块`CRTCLMOD`命令或创建绑定CL程序`CRTBNDCL`命令的`OPTION`参数指定`*DOSLTLVL`以使嵌套级别出现在编译器列表中
- 如果不想看到此嵌套级别信息，可以为`OPTION`参数指定`*NODOSLTLVL`

## DOUNTIL命令
&#8195;&#8195;`DOUNTIL`命令处理一组CL命令一次或多次。命令组定义为`DOUNTIL`和匹配的`ENDDO`命令之间的命令。在处理完这组命令之后，评估所述条件：
- 如果条件为真，则退出`DOUNTIL`组，并使用相关`ENDDO`之后的下一个命令继续处理
- 如果条件为假，组将继续处理组中的第一个命令

&#8195;&#8195;`COND`参数上的逻辑表达式可以是单个逻辑变量或常量，或者必须描述两个或多个操作数之间的关系；然后表达式被评估为真或假。示例关于使用`DOUNTIL`命令的条件处理：
```
DOUNTIL (&LGL)
  ...
  CHGVAR &INT (&INT + 1)
  IF (&INT *GT 5) (CHGVAR &LGL '1')
ENDDO
```
示例说明：
- `DOUNTIL`组的主体将至少运行一次
- 如果`&INT`变量的初始值为`5`或更大，则`&LGL`将在第一次设置为真，并且在组末尾计算表达式时将继续处理 `ENDDO`
- 如果初始值小于`5`，则组的主体会不断重复，直到`&INT`的值大于`5`并且`&LGL`的值变为`true`

`DOUNTIL`命令更多用法说明：
- `LEAVE`命令可用于退出`DOUNTIL`组并在`ENDDO`之后恢复处理
- `ITERATE`命令可用于跳过组中的剩余命令并立即评估所述条件

## DOWHILE命令
&#8195;&#8195;`DOWHILE`命令允许在逻辑表达式的值为真时处理一组命令零次或多次。`DOWHILE`命令用于说明一个条件，如果该条件为真，则指定程序或过程中要运行的一个命令或一组命令。命令组定义为`DOWHILE`和匹配的`ENDDO`命令之间的那些命令。在处理命令组之前判断所述条件：
- 如果所述条件最初为假，则永远不会处理该组命令
- 如果条件为假，则退出`DOWHILE`组，并使用相关`ENDDO`之后的下一个命令继续处理
- 如果条件为真，该组将继续处理该组中的第一个命令
- 当到达`ENDDO`命令时，控制返回到`DOWHILE`命令以再次判断条件

&#8195;&#8195;`COND`参数上的逻辑表达式可以是单个逻辑变量或常量，或者必须描述两个或多个操作数之间的关系；然后表达式被判断为真或假。示例关于使用`DOWHILE`命令的条件处理：
```
DOWHILE (&LGL)
  ...
  IF (&INT *EQ 2) (CHGVAR &LGL '0')
ENDDO
```
示例说明：
- 处理`DOWHILE`组时，将评估所述条件：
    - 如果条件为真，则处理`DOWHILE`组中的命令组
    - 如果条件为假，则继续处理相关`ENDDO`命令之后的命令
- 如果`&LGL`的值为真，则`DOWHILE`组中的命令将一直运行，直到`&INT`等于2，从而导致`&LGL`变量值设置为假

&#8195;&#8195;`LEAVE`命令可用于退出`DOWHILE`组并在`ENDDO`之后恢复处理。`ITERATE`命令可用于跳过组中的剩余命令并立即评估所述条件。
## DOFOR命令
&#8195;&#8195;`DOFOR`命令允许您处理一组命令指定的次数。`DOFOR`命令指定一个变量、它的初始值、增量或减量量以及终值条件。`DOFOR`命令的格式为：
```
DOFOR VAR(integer-variable) FROM(initial-value) TO(end-value) BY(integer-constant)
```
命令说明：
- 当`DOFOR`组的处理开始时，`VAR`参数指定的整数变量被初始化为`FROM`参数指定的初始值
- 整数变量的值与`TO`参数中指定的最终值进行比较
- 当`BY`参数上的整数常量为正时，比较会检查大于最终值的整数变量
- 如果`BY`参数上的整数常量为负数，则比较检查整数变量是否小于最终值
- 如果条件不成立，则处理`DOFOR`组的主体
- 当达到`ENDDO`时，将`BY`参数中的整数常量添加到整数值，并再次评估条件

示例关于使用`DOFOR`命令的条件处理：
```
CHGVAR &INT2 0
DOFOR VAR(&INT) FROM(2) TO(4) BY(1)
  ...
  CHGVAR &INT2 (&INT2 + &INT)
ENDDO
/* &INT2 = 9 after running the DOFOR group 3 times */
```
示例说明：
- 处理`DOFOR`组时，`&INT`初始化为2，检查`&INT`的值是否大于`4`
- 不是，所以处理组体
- 在该组的第二次迭代中，将一个添加到`&INT`并重复检查
- 它小于`4`，因此再次处理`DOFOR`组
- 第二次到达`ENDDO`时，`&INT`的值再次增加`1`。`&INT`现在的值为`4`
- 因为`&INT`仍然小于或等于`4`，所以再次处理`DOFOR`组
- 第三次到达`ENDDO`时，`&INT`的值再次增加`1`
- 这一次，值为`5`，并继续处理`ENDDO`之后的命令

&#8195;&#8195;`LEAVE`命令可用于退出`DOFOR`组并在`ENDDO`之后恢复处理。`ITERATE`命令可用于跳过组中的剩余命令，增加控制变量，并立即评估最终值条件。

## ITERATE命令