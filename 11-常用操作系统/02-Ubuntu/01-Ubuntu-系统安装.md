# Ubuntu-系统安装
记录常用安装方法。
## Windows下Ubuntu应用
&#8195;&#8195;在Microsoft Store中有Ubuntu应用程序，免费下载使用，下载后直接启动应用即可使用，使用起来比较方便，也不用虚拟机去安装那么麻烦。
### 安装步骤
首先打开Windows虚拟机平台：
- 控制面板-程序和功能
- 启动或关闭Windows功能
- 选中`适用于Linux的Windows子系统`和`虚拟机平台`后确定
- 等待功能开启并重启系统

如果不开启，启动Ubuntu会报错，如下所示：
```
Installing, this may take a few minutes...
WslRegisterDistribution failed with error: 0x8007019e
The Windows Subsystem for Linux optional component is not enabled. Please enable it and try again.
See https://aka.ms/wslinstall for details.
Press any key to continue...
```

下载安装步骤：
- Microsoft Store搜索Ubuntu
- 根据需求选择安装的版本，获取后下载安装应用
- 运行应用显示正在安装，等待若干分钟即可
- 提升新建一个用户，并且和Windows不一样的，新建即可

### Windows下Ubuntu目录
例如用户huang的根目录在Windows下路径：
```
C:\Users\admin\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs\home\huang
```
## 制作U盘引导安装Ubuntu