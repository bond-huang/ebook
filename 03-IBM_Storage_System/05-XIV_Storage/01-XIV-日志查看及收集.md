# XIV-日志查看及收集
XIV System Data类似于DS8000的PEpackage，在XIV里面称作XRAY。
## XIV GUI日志查看
通过XIV GUI查看很方便，在左侧“系统”菜单中可以查看：
- 所有系统警报
- 所有系统事件

在左侧“监视”菜单中可以查看：
- 系统：菜单点击后可以看到整个AIV存储3D视图，鼠标指向某个设备（例如硬盘）就会显示对应的信息，包括状态和事件
- 统计信息：菜单可以看到系统性能概况
- 警报：系统警报信息
- 事件：系统事件
- Qos性能类：性能限制组信息

## 数据收集
XIV System Data类似于DS8000的PEpackage，在XIV里面称作XRAY。
### 老版本收集XRAY
&#8195;&#8195;老版本GUI中没有收集XRAY的选项，需要通过终端（需安装XCLI，在IBM fix central中有下载）连上存储通过运行收集日志的程序进行收集。收集步骤如下：
- 将laptop网口IP获取设置成DHCP模式
- 将laptop链接到Patch panel上module 5 的laptop port
- 会自动获取一个14.10.202.xx的地址，如果没用，手动设置laptop地址
- 检查是否能ping通地址14.10.202.250，不可以继续检查网络
- 在laptop上的命令行执行收集程序，运行命令：`xary collect 14.10.202.250`
- 收集完成后会自动Offload到laptop上运行xray_collect.exe的目录下

注意：
- 如果XIV的微码是10.1或以上，使用xray_collect_v2.0.1.exe文件
- 如果低于此版本，使用xray_collect.exe文件
- 两个文件在对于的微码的包里面，微码下载也是在IBM fix central

### XIV GUI收集XRAY
&#8195;&#8195;从XIV GUI2.4.1版本开始，XIV支持用GUI的方式收集XRAY。XIV的GUI在IBM fix central中有下载，里面包含XCLI，下载安装完成后发现还有演示模式，这是个很不错的功能，可以在模拟环境下对XIV进行一些操作，方便学习及测试操作。收集步骤如下：
- 启动XIV GUI程序，然后登录，用户：technician，密码：teChn1cian
- 选中需要收集XRAY的XIV系统
- 在菜单栏选中并点击“Help”
- 下拉菜单中选中并点击“Support Logs...”
- 弹出对话框“Get Support Logs”，点击“Collect”
- 收集完成后，对话框中“System Log File Name”里面会列出当前系统里面的日志信息
- 选择刚才时间点收集的日志，点击“Browse”选择存放位置
- 最后点击“Get”即完成收集

在一些新的GUI版本中，收集选项有所改变，但是大致上差不多：
- 登录到AIV GUI 选中对应XIV系统后
- 在菜单栏选中并点击“工具（t）”
- 在展开菜单中选择“收集支持日志”点击后弹出对话框“收集并发送支持日志”
- 点击“启动”开始收集XARY

### XIV GUI收集容量报告
步骤如下：
- 登录到AIV GUI 选中对应XIV系统后
- 在菜单栏选中并点击“工具（t）”
- 在展开菜单中选择“生成容量报告”点击
- 点击“启动”开始收集

## TA tool使用
IBM XIV存储工程师使用，不作详述。
