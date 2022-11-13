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

## 待补充