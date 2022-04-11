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
- [DO command and DO groups in a CL program or procedure](https://www.ibm.com/docs/en/i/7.4?topic=cpwcpcp-do-command-do-groups-in-cl-program-procedure)

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