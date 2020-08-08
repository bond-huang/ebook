# PowerVM-VIOS常用命令
VIOS系统中常用的命令。
### AIX命令
&#8195;&#8195;VIOS底层是AIX系统，VIOS 2版本使用的是AIX6.1，VIOS 3版本开始使用的是AIX7.2，各版本命令基本一致。用padmin用户登录到VIOS后，可以使用命令：`oem_setup_env`切换到AIX root用户，o开头的命令使用的很少，个人一般使用`r o`进行切换，切换不了就输入完整命令，AIX层面的命令可以参考:[AIX-常用命令](https://bond-huang.github.io/huang/05-IBM_Operating_System/01-AIX/02-AIX-%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4.html)
### VIOS命令
&#8195;&#8195;默认使用padmin用户登录，VIOS相关命令只能在padmin用户下执行，有一些命令在AIX环境下可以执行的在VIOS里面也可以执行，有一些不行，不行就切换一下。下面介绍命令都是只在VIOS中的（有些一样但是输出不一样，例如`lspath`），记下来方便查阅。
##### 基础查看类
命令|用途
---|:---
ioslevel|查看vios版本
shutdown -restart|重启VIOS
license -accept|同意license
oem_setup_env|切换到AIX root
cfgdev|扫描识别新设备
errlog|查看报错信息
lspath|查看磁盘路径
lsrep|查看镜像对应的vtopt
lsvopt|查看vtopt对应的镜像
lsvg -pv rootvg|查看rootvg pv情况
lsvg -lv rootvg|查看rootvg lv情况
lsmap -all|查看vscsi映射关系
lsmap -all -npiv|查看npiv映射关系
lsmap -all -net|查看网络关系
lsmap ‑vadapter &#60;vhost&#62;|查看某个vhost映射
lsdev ‑dev &#60;device&#62;|查看设备(hdisk,vhost)
lsdev ‑dev &#60;device&#62; ‑attr &#60;attribute&#62;|查看设备指定属性
entstat -all &#60;ent&#62; &#124;grep Active|查看SEA状态

##### 属性修改
有些命令有点长，换个方式，命令加示例结合。

修改设备属性，例如修改hdisk的queue_depth：
```shell
chdev ‑dev hdisk2 ‑attr queue_depth=20
```
永久修改设备属性，例如hdisk2的reserve_policy属性：
```shell
chdev -dev hdisk2 -attr reserve_policy=no_reserve -perm
```
##### vtopt操作
创建VMLibrary，例如在rootvg下创建一个10G的：
```shell
mkrep -sp rootvg -size 10G
```
删除VMLibrary：
```shell
rmrep -f
```
创建vtopt，例如在vhost0上创建：
```shell
mkvdev -fbo -vadapter vhost0
```
将镜像加载到vtopt,例如AIX7231.iso加载到vtopt0：
```shell
loadopt -disk AIX7231.iso -vtd vtopt0
```
将镜像从vtopt上卸载,例如卸载vtopt0上的镜像：
```shell
unloadopt -vtd vtopt0
```
##### 基础配置
创建vscsi映射关系：
```shell
mkvdev -vdev <disk> -vadapter <vhost> -dev <VTD_name>
```
创建NPIV映射关系：
```shell
vfcmap -vadapter <vfchost> -fcp <fc_device> 
```
创建SEA:
```shell
mkvdev -sea <net_device> -vadapter <vnet> -default <vnet> -defaultid <vlan> -attr ha_mode=auto ctl_chan=<ctl_net>
```
取消NPIV映射关系：
```shell
vfcmap -vadapter <vfchost> -fcp
```
取消vscsi映射关系：
```shell
rmvdev -vtd <VTD_name>
```
##### SEA切换命令
有时候由于一些原因需要手动切换SEA。手动切换一种方式就是更改SEA网卡属性，先在每个vios上查看状态：
```shell
entstat -all <sea_adapter> |grep Active
```
在Active状态值为True的vios中输入如下命令进行切换：
```shell
chdev -dev <sea_adapter> -attr ha_mode=standby
```
当维护结束需要恢复时候的时候修改属性回auto：
```shell
chdev -dev <sea_adapter> -attr ha_mode=auto
```
##### 系统升级
```shell
 updateios -accept -install -dev <directory>
 ```
