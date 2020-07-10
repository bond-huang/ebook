# HMC收集数据
### 简介
HMC 是Power小型机管理控制台之一，可以对小型机进行硬件管理和逻辑配置等。
收集数据是解决问题的关键技能，本文介绍下收集HMC一些基础日志的方法。
### 收集iqyylog
iqyylog 主要用于受管小型机的硬件故障判断
#### Classic GUI
V8.7以前的HMC版本有Classis界面
收集方法如下：
- 在HMC物理设备插上U盘，必须是FAT32格式；推荐使用U盘，如果使用其它方式插入对应媒体设备
- 在导航区域中，打开“Service Management”
- 选择选项“Transmit Service Information”
- 选择选项“"Transmit Service Data to IBM”选项
- 通过选项“Service Data Destination”选择目标媒体，四种方式，后面三个选项通过远程WEB访问HMC是无法进行的：
  - IBM service support system：使用RSF将服务数据发送到IBM 服务支持系统
  - DVD-RAM:拷贝服务数据到DVD-RAM
  - USB memory stick:拷贝服务数据到U盘
  - Diskette：拷贝服务数据到软盘
- 在“product engineering files”选项中输入“/var/hsc/log/iqyy*”
- 最后点击 “send files”
- 完成后拔掉U盘，里面iqyy_.zip即是iqyylog

#### Enhanced GUI
V8.7以后的HMC版本就都是Enhanced GUI，并且不提供Classis界面
收集方法如下：
- 在HMC物理设备插上U盘，必须是FAT32格式；推荐使用U盘，如果使用其它方式插入对应媒体设备
- 左侧导航栏点击螺丝刀扳手图标，即“Serviceability”选项
- 选择选项“Service Management”
- 选择选项“Transmit Service Information”
- 选择选项“Send Problem Reports”
- 在“Service Data Destination”选项中选择“USB flash memory drive”
- 在“product engineering files”选项中输入“/var/hsc/log/iqyy*”
- 点击“Send Problem Reports”页面右上角的“Send Now”按钮，就可以把iqyylog数据发送到U盘了
- 完成后拔掉U盘，里面iqyy_.zip即是iqyylog

### ASMI LOG
ASMI全称Advanced System Management Interface,是IBM Power小型机管理界面
ASM log会记录小型机报告的软硬件事件，在进行故障判断时候此日志也很重要
通过HMC收集方法如下，建议通过web远程：
- 选中需要收集的受管Power小型机
- 展开选项“操作”
- 点击选项“启动Advanced System Management(ASM)”
- 用admin用户登录到ASMI
- 选择选项“Select System Service Aids”
- 选择选项“Error/Event Log”
- 显示出事件后，在事件左侧复选框中选中需要收集的事件，当然也可以全选
- 然后点击“Show Details”
- 然后crtl+a全选，crtl+c拷贝，最后crtl+v拷贝到登录自己电脑上

### 收集RIO Topology
Power小型机经常连接很多扩展柜，此日志对于排查I/O问题很重要
### 收集Resource DUMP
DUMP日志对于Power小型机和AIX分区突发性严重故障的排查很重要，例如机器宕机，AIX系统挂死等
收集方法如下：
- 登录到HMC
- 选中一台受管小型机
- 展开选项“Serviceability”
- 选择选项“Manage Dumps”
- 弹出窗口可以看到DUMP的list，选择对应DUMP
- 展开窗口上面导航栏的选项“Selected”
- 选择选项“Copy Dump to Media”可以传到U盘或者光盘
- 选择选项““Copy Dump to Remote System”可以通过FTP传出来

### 收集pedbg
对于HMC本身问题以及HMC和Power小型机连接之间的问题排查很重要
