# QEMU-虚拟AIX系统
AIX系统运行在IBM Power小型机上，由于底层架构上的区别，很少有平台可以去虚拟。QEMU很强大，可以模拟Power环境虚拟AIX系统，当然也有限制，不过用来学习AIX系统还是很方便的，有兴趣的可以联系我进行学习交流。
### 各平台简介
之前跟着大佬学习了Linux下用QEMU虚拟AIX，Linux下安装步骤：安装VMware、安装Linux、安装QEMU、最后虚拟AIX。安装特麻烦，个人电脑一般是Windows系统，对机器性能和使用上都很不方便。

后来尝试在Windows下用QEMU虚拟AIX，感觉学习起来很方便。但是AIX安装也是比较麻烦，很耗时间（顺利的话也要好几个小时），各种问题。所以这里分享一个直接用已安装配置好的虚拟磁盘拉起来的方法，好比VMware的`.ova`文件，直接恢复虚拟机。

### 准备文件
搭建系统和QEMU版本应该区别不大，我使用版本如下：
- 机器及系统版本：ThinkPad X250 Windows 10
- QUME版本：QEMU 4.1.0
- tap-windows版本：9.21.1
- HAXM版本：haxm-windows_v7_5_4
- AIX系统版本：AIX7.2.3.1

安装包：
- qemu-w64-setup-20200201.exe:官网下载
- tap-windows-9.21.1.exe：网上找
- haxm-windows_v7_5_4.zip：网上找
- AIX72.img：已安装配置好的QEMU虚拟机磁盘镜像

### 安装配置
##### 安装软件
步骤如下
- 安装HAXM，全称Hardware Accelerated Execution Manager，intel的硬件加速执行管理器。
- 安装QUEMU
- 安装tap-windows：用来搭建网桥让AIX访问外网
- 将AIX72.img放置到QEMU安装文件
