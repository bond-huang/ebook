# AS400-Spooled文件管理
&#8195;&#8195;Spooled文件管理包括诸如保留Spooled文件、释放Spooled文件和移动Spooled文件等任务。官方相关参考链接如下：
- [IBM i 7.3 Spooled file](https://www.ibm.com/docs/zh/i/7.3?topic=sfoq-spooled-file#rzalusplf)
- [IBM i 7.3 Managing printing](https://www.ibm.com/docs/zh/i/7.3?topic=printing-managing#rzalumanage)
- [IBM i 7.3 Spooled files and output queues](https://www.ibm.com/docs/zh/i/7.3?topic=concepts-spooled-files-output-queues)
- [IBM i 7.3 Managing spooled files](https://www.ibm.com/docs/zh/i/7.3?topic=printing-managing-spooled-files)

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

## 基于字符界面Spooled文件管理操作