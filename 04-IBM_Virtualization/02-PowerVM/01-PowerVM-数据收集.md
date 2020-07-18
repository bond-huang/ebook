# PowerVM-数据收集

### 硬件故障
PowerVM是Power小型机平台上产品，VIOS也是一个系统，出现硬件故障就是Power小型机硬件故障，在非操作系统层面需要收集的日志可以参照：[Power-小型机数据收集](https://bond-huang.github.io/huang/01-IBM_Power_System/02-Power_System/01-Power-%E5%B0%8F%E5%9E%8B%E6%9C%BA%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86.html)

### PowerVM问题
如果虚拟I/O的问题，一般是PowerVM配置或者设备参数问题，需要结合VIOS和VIOC来判断问题。
#### VIOS上收集
##### 收集snap
用padmin用户登录VIOS，输入命令`snap`收集snap，不需要加参数，在/home/padmin目录下生成的snap.pax.Z文件，拷贝出来即可。
##### 收集映射关系
收集vscsi、NPIV和网络映射关系，依次运行如下命令，通过终端日志功能记录下来：
```
$lsmap -all
$lsmap -all -npiv
$lsmap -all -net
```
将终端记录的日志拷贝下来。
#### VIOC上收集
用root用户登录到VIOC，依次运行：
```
snap -r
snap -ac
```
在/tmp/ibmsupr目录下生成的snap.pax.Z文件，拷贝出来即可。
#### HMC上收集
在HMC上同样也可以收集vscsi、NPIV和网络映射关系，用hscroot用户登录到HMC，依次运行如下命令，通过终端日志功能记录下来：
```
# lshwres -r virtualio -m <managed system> --rsubtype scsi --level lpar
# lshwres -r virtualio -m <managed system> --rsubtype fc --level lpar
# lshwres -r virtualio -m <managed system> --rsubtype eth --level lpar
```
