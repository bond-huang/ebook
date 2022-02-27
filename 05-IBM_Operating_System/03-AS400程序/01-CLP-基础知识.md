# CLP-基础知识
&#8195;&#8195;CLP全称Control Language Programming，Control language(CL)允许系统程序员和系统管理员使用IBM i命令和其他IBM提供的命令编写程序。官方参考链接：
- [IBM i 7.3 Control language](https://www.ibm.com/docs/zh/i/7.3?topic=programming-control-language)
- [IBM i 7.3 CL command finder-Alphabetic list](https://www.ibm.com/docs/zh/i/7.3?topic=language-alphabetic-list-cl-commands-by-command-name)
- [IBM i 7.3 CL command finder](https://www.ibm.com/docs/zh/i/7.3?topic=language-cl-command-finder)
- [IBM i 7.3 CL commands by product](https://www.ibm.com/docs/zh/i/7.3?topic=language-cl-commands-by-product)
- [IBM i 7.3 CL concepts](https://www.ibm.com/docs/zh/i/7.3?topic=language-cl-concepts)
- [IBM i 7.3 CL programming](https://www.ibm.com/docs/zh/i/7.3?topic=language-cl-programming)

## 创建CL程序过程
所有程序都是分步骤创建的：源代码创建、模块创建和程序创建:
- Source creation：CL源语句由CL命令组成，源语句按应用程序设计确定的逻辑顺序输入到数据库文件或IFS流文件中
- Module creation：使用`CRTCLMOD`(Create Control Language Module)命令，用于创建系统对象。创建的 CL模块可以绑定到程序中。一个CL模块包含一个CL程序。其他high-level language(HLL)语言可以为每个模块包含多个过程
- Program creation：使用`CRTPGM`(Create Program)命令，该模块（连同其他模块和服务程序）用于创建程序

注意事项：
- 如果想创建一个只包含一个C 模块的程序，可以使用`CRTBNDCL`(Create Bound CL Program)命令，结合了上面第二和第三步
- 如果想从CL源语句创建一个original program model(OPM原始程序模型)CL程序，可以使用`CRTCLPGM`(Create CL Program)命令

### 交互式输入
IBM i操作系统提供了很多菜单和屏幕进行交互式输入：
- 包括编程器的菜单，命令输入显示屏，显示命令提示符和编程开发管理器（PDM）菜单。
- 经常使用的源输入方法是源输入实用程序 (SEU:source entry utility)，它是WebSphere Development Studio的一部分
- 还可以使用`EDTF`(Edit File)命令在数据库源文件中输入或更改CL命令。但是EDTF不提供内置于SEU的集成CL命令提示支持

### 批量输入
&#8195;&#8195;可以在一个批处理输入流中创建CL源、一个CL模块和一个程序。使用`SBMDBJOB`(Submit Data Base Jobs)命令将输入提交到作业队列。输入流应遵循以下格式：
```
// BCHJOB
CRTBNDCL PGM(QGPL/EDUPGM) SRCFILE(PERLIST)
// DATA FILE(PERLIST) FILETYPE(*SRC)
   .
   .              (CL Procedure Source)
   .
//
/*
// ENDINP
```
&#8195;&#8195;示例从inline source创建程序。如果要将源代码保存在文件中，可以使用`CPYF`(Copy File) 命令将源代码复制到数据库文件中，然后可以使用数据库文件创建程序。

&#8195;&#8195;还可以使用设备文件直接从外部介质（例如磁带）上的CL源创建CL模块。IBM提供的磁带源文件是 `QTAPSRC`。例如，假设CL源语句位于名为PGMA的磁带上的源文件中。第一步是通过使用以下带有`LABEL`属性的覆盖命令来识别磁带上源的位置：
```
OVRTAPF FILE(QTAPSRC) LABEL(PGMA)
```
&#8195;&#8195;现在可以将`QTAPSRC`文件视为`CRTCLMOD`(Create CL Module)命令的源文件。根据磁带文件的源输入创建CL 模块：
```
CRTCLMOD MODULE(QGPL/PGMA) SRCFILE(QTAPSRC)
```
&#8195;&#8195;处理`CRTCLMOD`命令时，它会像对待任何数据库源文件一样对待`QTAPSRC`源文件。使用`OVRTAPF`，源位于磁带上，PGMA是在QGPL中创建的，并且该模块的源代码保留在磁带上。
### CL源程序的组成
&#8195;&#8195;尽管作为CL源程序的一部分输入的每个源语句实际上都是一个CL命令，但源可以分为基本部分使用在许多典型的CL源代码程序中。
#### PGM command
PGM PARM(&A)：可选的PGM命令启动源程序并标识收到的任何参数。
#### Declare commands
声明命令：DCL, DCLF, COPYRIGHT, DCLPRCOPT：
- 使用变量时程序或过程变量的强制性声明，以及子程序堆栈大小的可选定义
- `DCLPRCOPT`还提供了覆盖在用于调用CL编译器的CL命令上指定的编译器处理选项的功能
- 声明命令必须在除PGM命令之外的所有其他命令之前

#### INCLUDE command
CL命令在编译时嵌入额外的CL源命令。
#### CL processing commands
CHGVAR, SNDPGMMSG, OVRDBF, DLTF(部分命令)：CL命令用于源语句操作常量或变量。
#### Logic control commands
逻辑控制命令用于控制CL程序或过程中的处理的命令：IF, THEN, ELSE, DO, ENDDO, DOWHILE, DOUNTIL, DOFOR, LEAVE, ITERATE, GOTO, SELECT, ENDSELECT, WHEN, OTHERWISE, CALLSUBR, SUBR, RTNSUBR, ENDSUBR
#### Built-in functions
用于算术、字符串、关系或逻辑表达式的内置函数和运算符：
```
%SUBSTRING (%SST), %SWITCH, %BINARY (%BIN), %ADDRESS (%ADDR), %OFFSET (%OFS), %CHECK, %CHECKR, %SCAN, %TRIM, %TRIML, %TRIMR, %CHAR, %DEC, %INT, %UINT (%UNS), %LEN, %SIZE, %LOWER, %UPPER, %PARMS
```
#### Program control commands
用于将控制权传递给其他程序：CALL, RETURN, TFRCTL
#### Procedure control commands
将控制权传递给另一个procedure：CALLPRC
#### ENDPGM commands
可选的结束程序命令：ENDPGM

&#8195;&#8195;CL程序可以引用在创建程序、处理命令或两者时必须存在的其他对象。在某些情况下，为了让程序或过程成功运行，可能需要以下对象：
- A display file。如果程序使用display，则必须在创建模块之前使用`CRTDSPF`(Create Display File)命令输入和创建显示文件和记录格式。必须使用`DCLF`(Declare File)命令将其声明到声明部分中
- A database file。CL procedure可以读取数据库文件中的记录。如果使用数据库文件，则必须在创建模块之前使用`CRTPF`(Create Physical File)命令或`CRTLF`(Create Logical File)命令创建该文件。可以使用数据描述规范(DDS)、结构化查询语言(SQL)或交互式数据定义实用程序(IDDU)来定义文件中记录的格式。还必须使用`DCLF`(Declare File)命令将文件声明到DCL部分中
- Other programs。如果使用CALL命令，则在运行CALL命令之前，被调用的程序必须存在。在编译调用CL程序或 CL过程时它不必存在
- Other procedures。如果使用CALLPRC命令，则在运行`CRTBNDCL`(Create Bound CL Program)或`CRTPGM`(Create Program)命令时，调用的procedures必须存在

### 简单的程序示例
创建源文件：
```
CRTSRCPF FILE(SAVFLIB/SRCPF3)
```
添加member：
```
ADDPFM FILE(SAVFLIB/SRCPF3) MBR(CLRLOGPHS)
DSPPFM SAVFLIB/SRCPF3
```
使用PDM里面的`Work with members`编辑，或使用SEU编辑：
```
STRSEU SRCFILE(SAVFLIB/SRCPF3) SRCMBR(CLRLOGPHS)
```
输入命令`GO PROGRAM`进入Programmer Menu：
```
PROGRAM                          Programming                        
Select one of the following:                                           
     1. Programmer menu                                             
     2. Programming Development Manager (PDM)                       
     3. Utilities                                                   
     4. Programming language debug                                  
     5. Structured Query Language (SQL) pre-compiler                
     6. Question and answer                                                      
     8. Copy screen image                                           
     9. Cross System Product/Application Execution (CSP/AE)         
    50. System/36 programming                                                             
    70. Related commands                                            
```