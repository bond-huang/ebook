# Power小型机收集数据
### 简介
系统日志是解决问题的关键，本文介绍下收集Power小型机故障日志一些方法，对硬件故障判断和处理很重要。
### 收集iqyylog
HMC 是Power小型机管理控制台之一，可以对小型机进行硬件管理和逻辑配置等
HMC 受管小型机故障事件都会报告给HMC进行记录，HMC 中的iqyylog 记录了HMC
收集方法参考HMC章节内容：
[HMC收集数据](https://bond-huang.github.io/huang/01-IBM_Power_System/01-HMC/01-HMC%E6%94%B6%E9%9B%86%E6%95%B0%E6%8D%AE.html)
### 收集ASMI log
ASMI全称Advanced System Management Interface,是IBM Power小型机管理界面
ASMI log会记录小型机报告的软硬件事件，在进行故障判断时候此日志也很重要
#### HMC中收集
收集方法参考HMC章节内容：
[HMC收集数据](https://bond-huang.github.io/huang/01-IBM_Power_System/01-HMC/01-HMC%E6%94%B6%E9%9B%86%E6%95%B0%E6%8D%AE.html)
#### Laptop直连收集
如果没有HMC，或者HMC不方便收集，可以通过笔记本电脑直连到设备进行收集。
Power小型机都配有两个HMC管理口，默认ip如下：
- Power5：192.168.2.147/192.168.3.147
- Power6及以上系列：169.254.2.147/169.254.3.147
- 高端系列有四个口：169.254.2.147/169.254.3.147/169.254.2.147/169.254.3.147

收集方法如下：
- 配置laptop的IP地址，和小型机的HMC口ip段一致
- 将laptop连上小型机的HMC口
- 在laptop浏览器上输入：https://169.254.2.147
- 用admin用户登录到ASMI 
- 选择选项“Select System Service Aids”
- 选择选项“Error/Event Log”
- 显示出事件后，在事件左侧复选框中选中需要收集的事件，当然也可以全选
- 然后点击“Show Details”
- 然后crtl+a全选，crtl+c拷贝，最后crtl+v拷贝到登录自己电脑上

### 收集snap
Power系列小型机大多运行AIX系统，部分是AS400或者Linux，此处介绍常见的AIX系统中有硬件故障时候的日志收集，其它系统在相应系统中介绍。
AIX系统排查小型机硬件故障的snap日志收集方法如下：
- root用户登录到AIX系统
- 清除以前收集的日志，运行命令：snap -r
- 收集新日志，运行命令：运行命令snap -gc
- 日志存放目录：/tmp/ibmsupt
- 将目录中snap.pax.Z文件通过FTP拷贝出来即可

### 收集RIO Topology
Power小型机经常连接很多扩展柜，此日志对于排查I/O问题很重要
收集方法同样
### 收集DUMP
如果是宕机自动生成的，直接去HMC上收集对应的dump就行；
如果需要检查机器问题，特别是FSP问题，需要手动生成，FSP dump生成方法如下：
- 用admin用户登录到ASMI 
- 选择选项“System Service Aids”
- 选择选项“Error/Event Log”
- 选择选项“Service Processor Dump”
- 确认“Setting”选项是“Enabled”
- 点击“Initiate dump”

收集在HMC中进行，方法同样参考HMC章节内容：
[HMC收集数据](https://bond-huang.github.io/huang/01-IBM_Power_System/01-HMC/01-HMC%E6%94%B6%E9%9B%86%E6%95%B0%E6%8D%AE.html)
