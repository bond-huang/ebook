# Python-AIX日常巡检
学习过程中写的AIX自动巡检并生成HTML格式报告的脚本，初学者，还有很多有待优化。        
脚本存放位置：[https://github.com/bond-huang/AIX-Check-Script](https://github.com/bond-huang/AIX-Check-Script)
## 结构
整体结构如下：
- System Information
- System Error Check
  - System Hardware Error Event
  - System Software Error Event
  - System Errlogger Event
  - System Unknown Error Event
- System Performance Check
  - CPU Performance
  - Memory And PageSpace Check
  - System Disk Performance Data
  - System Adapter Performance Data
- System Rootvg Check
- File System Check
  - Check the filesystem usage rate
  - Check unmount filesystems of rootvg
- Check System Fix And Lpp Filesets
  - Check the AIX system fix filesets
  - Check the AIX system lpp filesets
- Check The System Device
  - The system hdisk information
  - The system adapter information
- Check The MPIO Device Path
  - Check the disable path
  - Check the defined path
  - Check the missing path
  - Check the failed path
- Check The System Process And Setting
  - Check the system reboot and upgrade time
  - Check the system process
  - Check the system ulimit setting
- Check The PowerHA
  - PowerHA 

## 详细结构说明
### System Information
获取的信息有：
- Machine Type
- Serial Number
- Platform Firmware Level
- Check Date
- Host Name
- AIX Level
- CPU Entitled Capacity
- Memory Size
- Page Space Size
- IP Address

### System Error Check
&#8195;&#8195;检查errpt命令输出的系统日志，分为四个类别分别检查和显示报错信息，有时候系统会持续报错，脚本根据报错ID进行了去重处理，避免大篇幅输出：
- System Hardware Error Event
- System Software Error Event
- System Errlogger Event
- System Unknown Error Event

### System Performance Check
分别检查了CPU,Memory&PageSpace,Disk及Adapter。
#### CPU Performance
&#8195;&#8195;示例中采样时间比较短，可以根据需求修改脚本，对CPU使用率进行了简单分析，对于CPU是Share类型的分区，对CPU性能项目entc也作了简单分析。
#### Memory And PageSpace Check
只是取出了系统相应的值，没有进行分析，只是简单说明下如何注意内存和换页空间的使用量。
#### System Disk Performance Data
对采样值进行了取最大和取平均，每个磁盘都进行了采样，采样频率可以根据需求修改脚本。
#### System Adapter Performance Data
对采样值进行了取最大和取平均，VSCSI和fcs适配器都进行了采样，采样频率可以根据需求修改脚本。

### System Rootvg Check
&#8195;&#8195;获取了rootvg一些值和镜像同步状态，rootvg LV状态以及lg_dumplv配置，同时对一些项目进行简单说明，有如下项目：
- Rootvg State
- Rootvg Disk Member
- Total PPs
- Free PPs
- PP Size
- Rootvg disks all on bootlist?
- Rootvg have a mirror?
- All LVs are syncd?
- All LVs are open?
- Primary dump LV
- Forced copy flag
- Always allow dump
- Dump compression
- Rootvg last backup time

### File System Check
#### Check the filesystem usage rate
对文件系统使用率进行了分析，大于80%的会列出来。
#### Check unmount filesystems of rootvg
有些情况下rootvg里面的文件系统不是全部挂载了，这里进行了检查，没挂载的会列出来。

### Check System Fix And Lpp Filesets
#### Check the AIX system fix filesets
检查了系统 fix filesets 安装情况，使用命令是`instfix -i`。
#### Check the AIX system lpp filesets
检查了系统 lpp filesets 安装情况，使用命令是`lppchk -v`。

### Check The System Device
主要是列出硬盘和挂载硬盘的适配器的几个重要属性。
#### The system hdisk information
列出项目有：Status,VG,ALGO,HC_interval,HC_mode,Reserve及PCM。
#### The system adapter information
主要是两类适配器：
- 对于vscsi适配器列出项目有：Status和vscsi_err_recov      
- 对于fscsi适配器列出项目有：Status,vscsi_err_recov,attach,dyntrk及fc_err_recov

### Check The MPIO Device Path
分四个类型检查path，正常的布列出，可能很多，如果检查出有问题的就会列出：
- Check the disable path
- Check the defined path
- Check the missing path
- Check the failed path

### Check The System Process And Setting
系统重要设置及进程检查。
#### Check the system reboot and upgrade time
一般AIX系统在升级后都建议重启，此处列出了重启时间和升级时间，并判断是否在升级后进行了重启。
#### Check the system process
只检查了两个进程：errdemon和srcmstr。
#### Check the system ulimit setting
列出了系统ulimit设置，并对各参数进行了简单说明。

### Check The PowerHA
首先检查了系统是否配置了PowerHA。

## 使用说明
使用说明如下：
- 推荐使用Python3，测试使用Python版本3.7.6
- 脚本中尽量避免使用Python第三方库，但是jinja2是必须的
- 推荐AIX系统版本7.1或更高，测试使用AIX版本7100-04-03-1642
- 需要AIX系统root用户权限才能运行，有些命令必须要root权限
- 如果系统使用非IBM多路径软件，则路径检查脚本可能用处不大
- 如果使用了多个pagespace，可能获取信息不准确，脚本中暂时没考虑到此点，有空修改

## 其它说明
&#8195;&#8195;之前打算使用第三方库reportlab，可以画图、画表格、编辑文字,最后可以输出PDF格式，尝试了几次AIX系统安装不上。打算通过jinja2生成HTML报告，然后再使用wkhtmltopdf转成pdf，尝试了几次AIX系统安装不上。生成的HTML报告可以通过其它平台进行转换。对于报告更完善的界面和舒适阅读方式还在研究中。
