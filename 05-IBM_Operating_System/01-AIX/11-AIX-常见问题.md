# AIX-常见问题
记录一些常见的AIX系统问题。
## 系统升级问题
### 升级时空间报错
升级版本过程中报错：
```
AIX 6.1 Not enough disk space for sni_lv.
```
问题说明：
```
sni_lv is an lv that is temporarily created during
the install but needs 256M of space to be created. Encouraged the 
customer to reduce the size of a fs to have at least 256 MB free in the 
rootvg, then retry the installation. 
```
解决方法：       
保证rootvg里面至少有256M的空余空间，再升级过程中会创建一个临时文件需要需要空间。

### 升级bosboot报错
升级AIX系统过程中报错：
```
0503-409 installp: bosboot verification starting... 
0503-497 installp: An error occurred during bosboot verification 
processing. 
```
查看boot信息重建bosboot命令都可以执行：
```
bootlist -m normal -o
bootlist -v
bosboot -ad /dev/hdisk0
bosboot -ad /dev/hdisk1
bootlist -m normal hdisk0 hdisk1
```
再次尝试升级还是同样报错，解决方法：

如果hdisk0是引导设备，那么下面命令输出应该是这样的：
```
#ls -l /dev/rhdisk0 /dev/ipldevice 
crw------- 2 root system 20, 0 Apr 07 2004 /dev/ipldevice 
crw------- 2 root system 20, 0 Apr 07 2004 /dev/rhdisk0 
```
如果`/dev/ipdevice`丢失或者不正确, 重建链接即可，示例：
```
#ln -f /dev/rhdisk0 /dev/ipldevice
```
再次尝试升级成功。

## 命令反应慢
&#8195;&#8195;系统正常运行，但是登录特别慢，登录成功后输入命令反应也特别慢，系统资源也不是很紧张，文件系统也没爆。刚开始怀疑DNS问题，检查没发现问题，再检查日志有大量以下报错：
```
3A30359F   1030230010 T S init           SOFTWARE PROGRAM ERROR 
```
报错详细内容：
```
Detail Data 
SOFTWARE ERROR CODE 
Command is respawning too rapidly. Check for possible errors. 
COMMAND 
id: audit "/usr/sbin/audit start # System Auditing" 
```
&#8195;&#8195;检查`/etc/inittab`文件，关于此命令的条目`Aciton`项设置是`respawn`,此设置表示如果进程不存在，启动进程；进程终止后重新启动。结合报错信息:Command is respawning too rapidly,说明执行过于频繁导致其它命令反应比较慢，检查命令是否有问题。

### 待补充
