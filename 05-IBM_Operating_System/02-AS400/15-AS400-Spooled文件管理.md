# AS400-Spooled文件管理
&#8195;&#8195;Spooled文件管理包括诸如保留Spooled文件、释放Spooled文件和移动Spooled文件等任务。官方相关参考链接如下：
- [IBM i 7.3 Spooled file](https://www.ibm.com/docs/zh/i/7.3?topic=sfoq-spooled-file#rzalusplf)
- [IBM i 7.3 Managing printing](https://www.ibm.com/docs/zh/i/7.3?topic=printing-managing#rzalumanage)
- [IBM i 7.3 Spooled files and output queues](https://www.ibm.com/docs/zh/i/7.3?topic=concepts-spooled-files-output-queues)
- [IBM i 7.3 Managing spooled files](https://www.ibm.com/docs/zh/i/7.3?topic=printing-managing-spooled-files)
- [IBM i 7.3 保存和恢复假脱机文件](https://www.ibm.com/docs/zh/i/7.3?topic=system-saving-restoring-spooled-files)

## Spooled文件概念
&#8195;&#8195;假脱机是将数据保存在数据库文件中以供以后处理或打印的系统功能。保存并最终打印的此数据称为假脱机文件（或打印机输出文件）。相关详细说明：
- Spooled文件是从应用程序、系统程序或通过按打印键创建的，这些文件放在称为输出队列的地方
- 几乎所有生成打印输出的应用程序都使用IBM i操作系统提供的假脱机支持
- 打印文件的`SPOOL`参数上的值`SPOOL(*YES)`和`SPOOL(*NO)`确定是否请求Spooled支持
- 使用`Print`键捕获显示屏的图像一般会创建假脱机文件(必须在工作站设备描述中指定的打印文件中指定`SPOOL =*YES`)。当按下`Print`键时，系统会查看`QSYSPRT`打印文件中的`OUTQ`参数，以确定将Spooled文件发送到哪个输出队列
- `QSYSPRT`打印文件中`SPOOL`属性的默认值为`*YES`

Spooling`SPOOL=*YES`比直接输出（打印机文件中的`SPOOL=*NO`）有几个优点：
- 用户的显示站仍可用于工作
- 其他用户无需等待打印机可用即可请求打印工作
- 如果需要特殊表格，用户可以将Spooled文件发送到特殊输出队列并在打印机不忙时打印
- 由于磁盘操作比打印机快得多，因此可以有效地使用系统

## 显示Spooled文件列表
使用IBM Navigator for i显示：
- 展开左侧`Basic Operations`菜单
- 单击选项`Printer Output`

默认设置是显示与当前用户关联的所有打印机输出，显示其它输出：
- 右键单击`Printer Output`
- 鼠标指向`Customize this view`
- 单击`Include`即可进入定制窗口

使用命令`WRKSPLF`可以查看当前用户的Printer Output：
```
                         Work with All Spooled Files                                     
Type options, press Enter.                                                    
  1=Send   2=Change   3=Hold   4=Delete   5=Display   6=Release   7=Messages  
  8=Attributes        9=Work with printing status                             
                             Device or                     Total     Cur      
Opt  File        User        Queue       User Data   Sts   Pages    Page  Copy
     QSYSPRT     BONDHUANG   BONDHUANG   WRKIT       RDY       1             1
     QPJOBLOG    BONDHUANG   QEZJOBLOG   QPAD142931  RDY      12             1
```
## 显示Spooled文件内容
使用IBM Navigator for i显示(可以显示ASCII Spooled文件)：
- 展开左侧`Basic Operations`菜单
- 单击选项`Printer Output`
- 右键单击要显示的打印机输出文件，点击`Open`

&#8195;&#8195;使用`WRKSPLF`命令，然后选择选项`5=Display`即可查看。基于字符的界面具有能够显示`*LINE`和`*IPDS`Spooled文件的附加功能。
## 显示与Spooled文件关联的消息
&#8195;&#8195;使用`WRKSPLF`命令，然后选择选项`7=Messages`即可查看。使用IBM Navigator for i查看方法步骤：
- 展开左侧`Basic Operations`菜单
- 单击选项`Printer Output`
- 右键单击带有消息的Printer Output文件，点击`Reply`

## Navigator中Spooled文件管理操作
### Navigator中Spooled文件管理选项
使用IBM Navigator for i中右键打印机输出文件相关操作：
- Open(O)：显示Spooled文件内容
- Reply(Y)：如果没有消息是灰色的，有消息可以显示消息并回复
- Hold(L)：暂时阻止用户选择打印的Spooled文件（打印机输出）
- Release(A)：释放hold的Spooled文件（打印机输出）
- Print Next(I)：打印下一个
- Send(N)：将Spooled文件发送给另一个用户或系统：
     - Send via TCP/IP：通过TCP/IP发送
     - Send via SNA：通过SNA发送
- Move(M)：将Spooled文件（打印机输出）从一个输出队列移动到另一个输出队列
- Delete(D)：删除Spooled文件（打印机输出）
- Convert to PDF：将Spooled转换为PDF文件：
     - 存储为打印机输出
     - 存储为流文件
     - 作为电子邮件发送
- Export(X)：将Spooled文件（打印机输出）导出到用户PC文件系统
- Cut(T))：剪切Spooled文件（打印机输出）
- Copy(C)：拷贝Spooled文件（打印机输出）
- Advanced(V)：
     - Restart Printing(R)：重新启动打印(命令官方没有介绍，应该是没有)
     - Manage Output Queue(O)：管理输出队列
     - Manage Printer(P)：管理打印机
- Properties(R)：查看及修改Spooled文件（打印机输出）的属性

### 启用Spooled文件通知消息
&#8195;&#8195;要在Spooled文件(打印机输出)完成打印或被打印机写入程序保留时收到通知，需要启用假脱机文件通知功能。在导航器中开启方法如下(官方没有介绍基于字符界面的方法)：
- 展开`Users and groups`，单击`Users`
- 右键单击要更改的用户名，然后选择`Properties`
- 在`General`面板中，单击用户设置下的`Jobs`
- 选择选项`Display Session`
- 勾选`Send message to spooled file owner`

## 基于字符界面Spooled文件管理操作
### 字符界面中Spooled文件管理选项
使用命令`WRKSPLF`进入`Work with All Spooled Files`，示例：
```
                         Work with All Spooled Files                          
Type options, press Enter.                                                    
  1=Send   2=Change   3=Hold   4=Delete   5=Display   6=Release   7=Messages  
  8=Attributes        9=Work with printing status                             
                             Device or                     Total     Cur      
Opt  File        User        Queue       User Data   Sts   Pages    Page  Copy
     QSYSPRT     BONDHUANG   BONDHUANG   WRKIT       RDY       1             1
     QPRTSPLQ    BONDHUANG   BONDHUANG               RDY       2             1
     QPJOBLOG    BONDHUANG   QEZJOBLOG   QZDASOINIT  RDY      91             1
```
选项说明如下：
- `1=Send`：将Spooled文件发送给另一个用户或系统，方式描述同上
- `2=Change`：更改Spooled文件的属性
- `3=Hold`：暂时阻止用户选择打印的Spooled文件
- `4=Delete`：删除Spooled文件（打印机输出）
- `5=Display`：显示Spooled文件内容
- `6=Release`：释放hold的Spooled文件（打印机输出）
- `7=Messages`：查看并回复Spooled文件的消息
- `8=Attributes`：查看Spooled文件（打印机输出）的属性
- `9=Work with printing status`：
     - `2=Change status`：更改状态
     - `5=Display detailed description`：显示详细描述

### 将Spooled文件转换为PDF
使用命令`CPYSPLF`(Copy Spooled File)：
```
CPYSPLF FILE(QPRTSPLQ) TOFILE(*TOSTMF) WSCST(*PDF)
```   
示例说明：
- 使用`TOFILE(*TOSTMF)`参数指示用户要将Spooled文件复制到流文件
- 使用`TOSTMF`和`WSCST`参数指定流文件中输出的位置和格式(PDF)

## 控制Spooled文件数量
&#8195;&#8195;作业完成后，会保留Spooled文件和内部作业控制信息，直到打印或取消Spooled文件。系统上的作业数和系统已知的Spooled文件数增加了执行IPL和内部搜索所需的时间，并增加了所需的临时存储量。
### 控制Spooled文件数量方法
&#8195;&#8195;用户可以使用`CRTJOBD`(Create Job Description)或`CHGJOB`(Change Job)命令上的`LOG`和`LOGOUTPUT`参数：
- `LOG`(Message logging)：
     - `Level`,`Severity`,`Text`
- `LOGOUTPUT`(Job log output)：
     - `*SAME`,`*SYSVAL`,`*JOBLOGSVR`,`*JOBEND`及`*PND`

&#8195;&#8195;或使用`QLOGOUTPUT`(Job log output)系统值来控制生成的作业日志的数量，可以设置的值有：`*JOBEND`,`*JOBLOGSVR`及`*PND`，示例如下：
```
                             Display System Value                  
System value . . . . . :   QLOGOUTPUT                              
Description  . . . . . :   Job log output                          
Job log output . . . . :   *JOBEND        *JOBEND, *JOBLOGSVR, *PND
```
&#8195;&#8195;用户还可以通过使用系统清理功能来控制作业日志和其他系统输出在系统上停留的天数。有关详细信息，参考命令[Change Cleanup(CHGCLNUP)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/chgclnup.htm)。示例：
```
CHGCLNUP ALWCLNUP(*YES) USRMSG(*KEEP) STRTIME(0700)
```
示例说明：
- 示例更改了清理选项，以便在执行清理时保留用户消息而不将其删除
- 示例中将清理开始时间设置为上午7:00

## 删除过期Spooled文件
&#8195;&#8195;在`CHGPRTF`(Change Printer File)、`CRTPRTF`(Create Printer File)、`CHGSPLFA`(Change Spooled File Attributes)或`OVRPRTF`(Override with Printer File)命令上使用`EXPDATE`或`DAYS`参数，以使Spooled文件符合命令`DLTEXPSPLF`(Delete Expired Spooled files)的删除条件。示例如下：
```
ADDJOBSCDE JOB(DLTEXPSPLF) CMD(DLTEXPSPLF ASPDEV(*ALL)) FRQ(*WEEKLY) SCDDATE(*NONE) SCDDAY(*ALL) SCDTIME(010000) JOBQ(QSYS/QSYSNOMAX) TEXT('DELETE EXPIRED SPOOLED FILES SCHEDULE ENTRY')
```
示例会创建一个作业计划任务条目，改任务会使用`DLTEXPSPLF`命令每天删除系统上所有过期的Spooled文件。

## 回收Spooled文件存储
&#8195;&#8195;使用`RCLSPLSTG`(Reclaim Spool Storage)命令或自动清理未使用的打印机输出存储 `QRCLSPLSTG`(Reclaim spool storage)系统值来回收Spooled文件存储。官方参考链接：[IBM i 7.3 Reclaiming spooled file storage](https://www.ibm.com/docs/zh/i/7.3?topic=msf-reclaiming-spooled-file-storage)。

注意事项：这些是从`QSPL`或`QSPLxxxx`库中删除Spooled数据库成员的唯一允许方法，任何其他方式都可能导致严重的问题。
## 保存及恢复Spooled文件
&#8195;&#8195;使用`SAVLIB`、`SAVOBJ`、`STLIB`和`RSTOBJ`CL命令上的`SPLFDTA`参数来保存和恢复Spooled文件，并且不会丢失Spooled文件的打印保真度、属性或标识。官方参考链接：
- [Saving and restoring spooled files](https://www.ibm.com/docs/zh/i/7.3?topic=msf-saving-restoring-spooled-files)
- [IBM i 7.3 保存和恢复假脱机文件](https://www.ibm.com/docs/zh/i/7.3?topic=system-saving-restoring-spooled-files)

## 通过Spooled文件大小控制打印
&#8195;&#8195;使用`CRTOUTQ`(Create Output Queue)或`CHGOUTQ`(Change Output Queue)命令上的`MAXPAGES`参数按大小控制Spooled文件的打印。示例：
```
CHGOUTQ OUTQ(MYOUTQ) MAXPAGES((40 0800 1600) (10 1200 1300))
```
示例说明：
- 示例要限制在上午8点到下午4点之间在输出队列`MYOUTQ`上打印超过40页的Spooled文件
- 在中午12点和下午1点之间，打印不超过10页的Spooled文件

## 修复输出队列和Spooled文件
&#8195;&#8195;使用命令`STRSPLRCL`(Start Spool Reclaim)修复在不可恢复状态的输出队列和spooled文件。示例回收输出队列`BONDHUANG`和驻留在输出队列上的所有Spooled文件：
```
STRSPLRCL OUTQ(QUSRSYS/BONDHUANG)
```
&#8195;&#8195;修复系统和基本用户ASP中的所有输出队列和Spooled文件。示例将回收在系统辅助存储池(ASP1)和所有定义的基本用户ASP(ASP 2-32)中找到的所有库中的所有输出队列，驻留在输出队列上的Spooled文件也将被回收：
```
STRSPLRCL OUTQ(*ALL/*ALL) ASPGRP(*SYSBAS)
```
&#8195;&#8195;修复当前用户的ASP组中的输出队列。示例将回收当前用户的ASP组中所有库中名为`PRT01`的输出队列。驻留在选定输出队列上的Spooled文件也将被回收：
```
STRSPLRCL OUTQ(*ALL/PRT01) ASPGRP(*CURASPGRP)
```
## Spool Performance
### 可能遇到锁争用的Spool对象
#### Output queue (OUTQ)
&#8195;&#8195;此对象是Spooled文件的存储库。在内部，这个对象被实现为一个独立的索引（类型0E子类型 02）。输出队列上的Spooled文件都表示为`Output queue`索引上的一个条目：
- 当前的内部设计不允许在不影响数据完整性的情况下共享对输出队列对象的访问。对于在Spooled文件上执行的每个操作，都会在Spooled文件所在的输出队列上获得一个排他锁
- 这些操作包括在输出队列上添加(CRTSPLF)、删除(DLTSPLF)、保留(HLDSPLF)、释放(RLSPLF)、更改(CHGSPLFA)或列出(WRKOUTQ)Spooled文件
- 这意味着使用`WRKOUTQ`(Work with Output Queue)命令将与创建Spooled文件冲突，与删除Spooled文件等冲突

#### 内部打印队列(PRTQ)
&#8195;&#8195;内部打印队列包括：QSPUSRQ、QSPALLQ、QSPDEVQ、QSPFRMQ、QSPUDTQ、QSPASPQ、QSPQJBQ。这些内部对象使用`WRKSPLF`命令有效地列出Spooled文件：
- 在内部，这些对象被实现为独立的索引（类型0E子类型C7）
- 系统上的每个Spooled文件都表示为每个索引上的一个条目
- 每个索引都有一个不同的键，用于对Spooled文件列表进行子集化。例如：
     - 如果交互式用户执行`WRKSPLF USER(QSYS)`，将锁定`QSPUSRQ`索引，该索引与用户名无关。仅列出用户`QSYS`拥有的那些Spooled文件
     - 如果用户执行`WRKSPLF DEV(PRT01)`，将锁定`QSPDEVQ`索引，该索引与设备名称无关。仅列出设备输出队列`PRT01`上的那些Spooled文件
     - 这样就无需检查系统上的所有Spooled文件以确定它是否与`WRKSPLF`命令上的过滤条件匹配
- 与输出队列一样，内部打印队列设计的主要缺点是在系统上添加(CRTSPLF)、删除(DLTSPLF)、保留(HLDSPLF)、释放(RLSPLF)、更改(CHGSPLFA)或读取(WRKSPLF)Spooled文件时需要独占锁
- 这意味着，`WRKSPLF USER(QSYS)`命令将与创建Spooled文件冲突，与删除Spooled文件等冲突

#### Output file control block(OFCB)
&#8195;&#8195;`OFCB`或Spooled数据库成员是为Spooled文件存储数据和属性的位置Spooled文件属性存储在与数据库成员关联的空间中：
- 在数据库成员上获得一个独占空间位置锁，以同步对Spooled文件属性的访问
- 此对象上的锁争用通常不是问题，因为多个作业通常不会尝试同时访问同一个Spooled文件

#### Spool control block(SCB)
&#8195;&#8195;系统上的每个作业都存在一个`spool control block`（类型19子类型C2）。此对象挂在工作控制块表条目之外，主要用于保存作业的Spooled文件数量和某些作业属性的计数器：
- 在为特定作业添加(CRTSPLF)或删除(DLTSPLF)Spooled文件时，或者如果作业的属性正在更改而影响Spooled文件，则需要在SCB上使用排他锁
- 从在V5R1M0中将Spooled文件的属性从SCB移到数据库成员中后，该对象就不再是争用的主要来源
- 但是，由于交换或网络Spooled文件活动(SNDNETSPLF或SNDTCPSPLF)，代表其他用户创建许多Spooled文件的客户会遇到争用问题。通常会在附加到`QPRTJOB`作业的SCB上看到这一点

#### Reader writer control block(RWCB)
&#8195;&#8195;每个系统上存在一个`RWCB`对象（类型19子类型C5）。此对象包含系统上每个活动写入器的一个条目。 列出(WRKWTR)、启动(STRPRTWTR)、结束(ENDWTR)或更改(CHGWTR)写入器时，`RWCB`需要排他锁。在启动或结束所有或多个写入器时，更有可能发生对该对象的争用。

### 可指示Spooled锁争用的消息
消息如下：
- MCH5802-对象&1的锁定操作不满足
- MCH5804-在指定的时间间隔内不满足锁定空间定位操作
- CPF3330-没有必要的资源
- CPF4218-&7中的输出队列&6不可用
- CPF2528-由于&1，作业日志未写入输出队列
- CPF4235-无法打开假脱机文件。原因代码是&6

### 一般Spool性能建议
建议如下：
- 减少系统上Spooled文件的数量。 在V5R4M0中添加的过期日期(EXPDATE)Spooled文件属性可用于自动从系统中删除假脱机文件。参考`DLTEXPSPLF`(Delete Expired Spooled files)命令
- 将Spooled文件分散到尽可能多的输出队列、用户和作业中
- 确保系统资源调配正确。对于`WRKOUTQ`和`WRKSPLF`之类的操作，分页吞吐量是一个重要的门控因素。增加分配给作业处理Spooled文件列表的内存等资源
- 使用`SAVOBJ/RSTOBJ`命令保存Spooled文件数据。保存后，将它们从系统中删除
- 将作业日志输出系统值`QLOGOUTPUT`或作业属性`LOGOUTPUT`更改为`*PND`以减少在系统上创建的作业日志的数量
- 对于系统上的每个活动写入器，将至少1兆字节的主存储专用于`*SPOOL`池
- 确保`QRCLSPLSTG`系统值未设置为`*NONE`。使用`*NONE`会对创建Spooled文件的性能产生不利影响
- 在高度活跃的Spooled环境中的高峰活动期间避免以下长时间运行的操作：
     - 使用格式`SPLF0100`或`SPLF0200`调用`QUSLSPL API`以列出系统上的所有Spooled文件
     - 使用格式`OSPL0100`或`OSPL0200`调用`QUSLSPL API`以列出系统上的所有Spooled文件
     - 许多基本辅助级别的Spooled和打印操作
     - `CHGJOB OUTPTY(X)`针对具有数百或数千个Spooled文件的作业
     - `CHGJOB SPLFACN(*DETACH)`针对具有数百或数千个Spooled文件的作业
     - `HLDJOB SPLFILE(*YES)`针对具有数百或数千个Spooled文件的作业
     - `RLSJOB`针对以前使用`HLDJOB SPLFILE(*YES)`保存的具有数百或数千个Spooled文件的作业
     - 运行`CLROUTQ`
     - 运行`CALL PGM(QSYS/QSPFIXUP)`

### WRKSPLF性能注意事项
&#8195;&#8195;使用`WRKSPLF`获取Spooled文件列表时，某些选项可能会对具有高度活跃Spooled环境的系统造成性能影响。建议用户选择特定的用户名、打印设备、表单类型、用户数据或ASP来过滤列表。还建议在此环境中避免使用以下选项：
- 运行命令`WRKSPLF SELECT(*ALL)`
- 使用通用的用户名或用户数据来过滤没有推荐选项之一的列表
- 使用Spooled文件名、限定的作业名或创建日期和时间来过滤列表，而没有推荐的选项之一
- 使用基本辅助级别：`ASTLVL(*BASIC)`

### Spool锁争用场景
#### 输出队列锁争用
自`S/38`以来，输出队列锁争用一直是个问题。最近的罪魁祸首是输出队列`QEZJOBLOG`：
- 当许多作业同时结束并尝试削减作业日志时，可能会出现瓶颈
- 如果瓶颈足够严重，则会出现消息`CPF2528`、`CPF4218`和`MCH5802/CPF3330`。输出队列争用不限于`QEZJOBLOG`
- 一些客户将全部或大部分Spooled输出集中到一个或几个输出队列。这可能导致比`QEZJOBLOG`更糟糕的瓶颈

##### 场景一
数千个作业同时结束，并尝试将作业日志切到输出队列QEZJOBLOG。

结果：
- 在这种类型的环境中，可能会导致`QEZJOBLOG`出现严重的瓶颈
- 再加上执行`WRKOUTQ OUTQ(QEZJOBLOG)`的用户，可能会在输出队列上发生严重争用
- `WRKOUTQ OUTQ(QEZJOBLOG)`将在持有排他锁的同时拍摄输出队列的快照，这可能需要几秒钟或几分钟，具体取决于专用于该作业的资源

症状：
- 尝试在`QEZJOBLOG`上创建作业日志或访问Spooled文件的作业将进入`LCKW`或挂在`END`状态，直到可以剪切作业的作业日志或执行`WRKOUTQ`的用户已获得快照
- 可能会产生消息`CPF2528`、`MCH5802/CPF3330`和`CPF4218`
- `WRKOBJLCK OBJ(QEZJOBLOG) OBJTYPE(*OUTQ)`命令可能会显示等待`*EXCL`锁的作业。随着瓶颈的消退，锁的持有者将从一个作业转移到下一个作业

建议：
- 将作业日志输出系统值`QLOGOUTPUT`或作业属性`LOGOUTPUT`更改为`*PND`以减少在系统上创建的作业日志的数量。`*PND`选项将允许“按需”创建作业日志
- 如果需要Spooled作业日志，请在作业日志输出系统值`QLOGOUTPUT`或作业属性`LOGOUTPUT`上指定 `*JOBLOGSVR`。这将减少系统上创建作业日志的作业数量
- 将作业描述的日志记录级别更改为不剪切作业日志或仅在需要时剪切作业日志：
     - 注意：如果作业异常结束，无论作业的日志级别如何，作业日志都将可用
     - 减少`QEZJOBLOG`上的Spooled文件数将会使`WRKOUTQ OUTQ(QEZJOBLOG)`命令持有排他锁的时间更短
- 调用`ENDSBS`命令时，使用`ENDSBSOPT(*NOJOBLOG)`参数减少创建的作业日志量
- 将作业日志Spooled文件分散到多个输出队列中。如何将不同子系统创建的作业日志路由到不同的作业日志输出队列的示例如下：
     - 为系统上定义的每个需要单独的作业日志输出队列的子系统创建一个库
     - 创建`QUSRSYS/QEZJOBLOG`输出队列的副本，并将副本放入创建的每个库中
     - 创建`QSYS/QPJOBLOG`打印机文件的副本，并将副本放入创建的每个库中
     - 将每个重复的`QPJOBLOG`打印机文件的`OUTQ`属性从`QUSRSYS/QEZJOBLOG`更改为`*LIBL/QEZJOBLOG`
     - 为每个子系统创建一个路由条目以调用执行`CHGSYSLIBL LIB(x)`的单独程序，库列表的系统部分被修改为在`QSYS`上面的第一步中创建的新库
     - 注意：使用此技术时，请注意不会对这些重复的输出队列进行系统输出的自动清理。结合使用`EXPDATE`Spooled文件属性和`DLTEXPSPLF`命令来自动清理Spooled文件
- 确保系统资源调配正确。对于`WRKOUTQ`和`WRKSPLF`之类的操作，分页吞吐量是一个重要的门控因素。增加分配给作业处理Spooled文件列表的内存等资源
- 在不使用输出队列的非高峰时间，将Spooled文件移动到备用输出队列。`DLTOLDSPLF`之类的应用程序使用`CHGSPLFA`命令(非DLTSPLF)修改以移动超过`X`天的Spooled文件
- 使用OA清理功能`GO CLEANUP`减少作业日志的保留期
- 要减少对`QEZJOBLOG`的争用，使用`WRKJOBLOG`命令而不是`WRKOUTQ`来访问作业日志Spooled文件。此命令不会获得`QEZJOBLOG`输出队列上的锁定
- 确保`QEZJOBLOG`仅用于保存作业日志Spooled文件
- 确保`QRCLSPLSTG`系统值未设置为`*NONE`。使用`*NONE`会对创建Spooled文件的性能产生不利影响

##### 场景二
&#8195;&#8195;客户每天创建`30000`个Spooled文件到输出队列(OUTQA)。客户的Spooled文件保留政策是将Spooled文件在`OUTQA`中保留`7`天。输出队列平均有`200000-210000`个Spooled文件。在高峰操作期间，客户有10个应用程序同时在输出队列上创建和更改Spooled文件。客户在系统上还有25个唯一用户试图访问相同的Spooled文件以显示、更改、保留或释放它们。

结果：
- 在这种类型的环境中，可能会导致输出队列出现严重的瓶颈
- 对于`OUTQA`中的Spooled文件执行的每个操作，都会在输出队列上获得一个排他锁
- 再加上执行`WRKOUTQ OUTQ(OUTQA)`的用户，可能会在输出队列上发生严重争用
- 这可能需要几秒钟或几分钟，具体取决于专用于该作业的资源
- 随着Spooled文件的数量、创建Spooled文件的应用程序或访问输出队列上的Spooled文件的用户增加，争用也会增加

症状：
- 尝试在输出队列上创建或访问Spooled文件的作业将进入`LCKW`，从而导致作业吞吐量降低
- 可能会产生消息`MCH5802/CPF3330`和`CPF4218`
- `WRKOBJLCK OBJ(OUTQA) OBJTYPE(*OUTQ)`命令可能会显示等待`*EXCL`锁的作业。随着瓶颈的消退，锁的持有者将从一个作业转移到下一个作业

建议：
- 将Spooled文件分散到尽可能多的输出队列中。减少输出队列上的Spooled文件数将会使`WRKOUTQ OUTQ(OUTQA)`命令在更短的时间内保持独占锁
- 使用`SAVOBJ/RSTOBJ`命令保存Spooled文件数据。保存Spooled文件后，将其从系统中删除
- Spooled文件到期日期`EXPDATE`属性与`DLTEXPSPLF`命令一起可用于自动从系统中删除Spooled文件
- 确保系统资源调配正确。对于`WRKOUTQ`和`WRKSPLF`之类的操作，分页吞吐量是一个重要的门控因素。增加分配给作业处理Spooled文件列表的内存等资源
- 验证是否确实需要输出队列上的所有Spooled文件。许多客户和业务合作伙伴应用程序创建临时Spooled文件并立即删除它们的案例。建议尽可能避免使用Spooled文件作为存储临时数据的工具。Spooled文件审计可以帮助确定这是否是一个问题，因为大多数客户都没有意识到这种情况正在发生
- 在不使用输出队列的非高峰时间，将Spooled文件移动到备用输出队列。`DLTOLDSPLF`之类的应用程序使用`CHGSPLFA`命令(非DLTSPLF)修改以移动超过`X`天的Spooled文件
- 要减少具有数千个Spooled文件的输出队列的争用，使用`WRKSPLF`或`WRKJOB OPTION(*SPLF)`来访问Spooled文件，而不是`WRKOUTQ`
- 确保将作业日志和系统转储生成到专用输出队列
- 确保`QRCLSPLSTG`系统值未设置为`*NONE`。使用`*NONE`会对创建Spooled文件的性能产生不利影响

#### 内部打印队列锁争用
&#8195;&#8195;内部打印队列对象争用的罪魁祸首是`WRKSPLF`命令，该命令列出了系统上所有或大部分Spooled文件。更多说明如下：
- `WRKSPLF USER(*ALL)`将在持有索引排他锁的同时拍摄`QSPALLQ`内部打印队列的快照
- 此索引包含系统上每个Spooled文件的条目，因此生成快照可能需要几秒钟或几分钟
- 当索引被锁定时，系统上的Spooled文件不能被创建、保留、释放、删除或更改
- 如果瓶颈足够严重，可能会出现消息`CPF4235 RC1`和`MCH5802/CPF3330`

##### 场景一
在高峰运行时间，在具有`300000`个Spooled文件的系统上，用户执行`WRKSPLF USER(*ALL)`。

结果：
- `WRKSPLF USER(*ALL)`将在持有排他锁的同时拍摄`QSPALLQ`索引的快照
- 这可能需要几秒钟或几分钟，具体取决于专用于该作业的资源
- 在快照完成之前，不能在系统上创建、保留、释放、删除或更改任何Spooled文件

症状：
- 尝试创建、保留、释放、删除、更改或列出Spooled文件的作业将在`LCKW`状态下挂起
- 可能会产生消息`MCH5802/CPF3330`和`CPF4235 RC1`
- 由于这是一个内部对象，因此不能使用`WRKOBJLCK`命令来确定持有锁的作业
- 在`QSPTLIB`库(可通过PTF获得)中提供的`DSPLCKSTS`命令，可用于确定锁持有者
- 一旦`WRKSPLF USER(*ALL)`的用户完成了索引的快照，瓶颈就会消失

建议：
- 减少系统上Spooled文件的数量。使用`SAVOBJ/RSTOBJ`命令保存Spooled文件数据。 保存后将它们从系统中删除。Spooled文件`EXPDATE`属性与`DLTEXPSPLF`命令一起可用于自动从系统中删除Spooled文件
- 确保系统资源调配正确。对于`WRKOUTQ`和`WRKSPLF`之类的操作，分页吞吐量是一个重要的门控因素。增加分配给作业处理Spooled文件列表的内存等资源
- 使用`WRKSPLF USER(*ALL)`以外的其他内容来对Spooled文件列表进行子集化。过滤用户、表单类型或用户数据可能更合适。使用`WRKJOB OPTION(*SPLF)`或`WRKOUTQ`获取Spooled文件列表
- V5R3M0中添加支持将Spooled文件存储在`IASP`中。这种新设计用键控数据库逻辑文件替换了内部打印队列对象。这种方法允许共享访问数据库逻辑文件

##### 场景二

## 待补充