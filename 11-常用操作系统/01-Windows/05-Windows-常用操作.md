# Windows-常用操作
## 系统常用操作
### JAVA环境变量配置
添加JAVA环境变量（windows10为例）：
- 右键此电脑，点击属性
- 在相关设置中找到高级系统设置
- 点击环境变量
    - 在系统变量中新建`JAVA_HOME`变量，值为JAVA安装路径，例如`C:\Program Files\Java\jdk1.8.0_112`
    - 在系统变量中新建CLASSPATH变量，变量值：`.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`
    - 在系统变量里面编辑Path变量，新建两个，分别为：`%JAVA_HOME%\bin`  `%JAVA_HOME%\jre\bin`
    - 再添加一个安装JDK的目录：`C:\Program Files\Java\jdk1.8.0_112\bin`
- 点击确认完成编辑

进行验证测试：
```
C:\Users\admin>java -version
java version "1.8.0_311"
Java(TM) SE Runtime Environment (build 1.8.0_311-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.311-b11, mixed mode)
```
## 防火墙
### 入站规则
#### 开启ping权限
步骤如下：
- 打开Windows防火墙
- 点击`高级设置`
- 点击`入站规则`
- 找到`文件和打印机共享（回显请求-ICMPv4-In）`
- 右键，点击`启用规则`

## 常用软件操作
### Notepad++
#### Notepad++文件比对
安装Compare插件：
- 打开Notepad++软件
- 主菜单选择选项`插件`，然后点击`插件管理`
- 在`插件管理`界面列表中找到并选中`Compare`
- 然后点击右上角`安装`按钮进行安装，安装会重启Notepad++

使用Compare插件：
- 打开两份需要比对的文档，分别放置在左右不同的视图中
- 主菜单选择选项`插件`，然后点击`Compare`
- 在`Compare`展开列表中点击`Compare`，比对结果显现
- 或直接`Ctrl+Alt+C`

## 常用命令
### tree命令
打开命令行，使用`tree /?`命令可以查看使用方法及参数：
```
C:\Windows\system32>tree /?
以图形显示驱动器或路径的文件夹结构。

TREE [drive:][path] [/F] [/A]

   /F   显示每个文件夹中文件的名称。
   /A   使用 ASCII 字符，而不使用扩展字符。
```
示例查看D盘所有目录及文件树：
```
C:\Windows\system32>tree d:/ /F
卷 新加卷 的文件夹 PATH 列表
卷序列号为 000000E2 74F5:C51B
D:\
├─AS400学习资料
│      AS400系统基本操作.docx
│
├─huang
│  │  AIX_TEST.bat
│  │  AIX_TEST.bat - 快捷方式.lnk
...output omitted...
```
## Microsoft Office常用操作
### Word
#### 页眉横线删除
有时候页眉有个横线想删除，方法：
- 在页眉处双击进入页眉页脚编辑状态
- 按快捷键`Crtl+Shift+N`可以直接删除页眉横线

### Excel
#### 修改列显示格式
修改成纯数字：文件--选项--公示--勾选`R1C1引用样式`
## 待补充