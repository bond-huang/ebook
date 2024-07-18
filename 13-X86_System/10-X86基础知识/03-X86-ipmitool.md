# X86-ipmitool
## ipmitool安装
## ipmitool常用命令
查看帮助：
```sh
ipmitool -h
```
`ipmitool sensor`命令查看传感器信息，示例：
```s
ipmitool -H <host ip>  -U <user>  -P <passowrd> sensor list
ipmitool -H <host ip>  -U <user>  -P <passowrd> sensor get "CPU Temp"
ipmitool -H <host ip>  -U <user>  -P <passowrd> sensor get "FAN1"
ipmitool -H <host ip>  -U <user>  -P <passowrd> chassis status
ipmitool -H <host ip>  -U <user>  -P <passowrd> power on
```
电源操作：
```sh
### 关机
sensor power off
### 软关机
sensor power soft
### 硬重启
sensor power reset
### 查看当前电源状态
sensor power status：
```
查看BMC硬件信息：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> mc info
```
查看BMC当前已启动的配置：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> mc getenable
```
查看系统日志：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> sel list
```
查看详细扩展日志：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> sel elist
ipmitool -H <host ip>  -U <user>  -P <passowrd> sel elist first 10
ipmitool -H <host ip>  -U <user>  -P <passowrd> sel elist last 10
```
删除所有系统日志：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> sel clear
```
发送事件到系统消息日志：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> event 1
```
BMC用户管理：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> user list
```
修改BMC用户密码：
```sh
ipmitool -H <host ip>  -U <user>  -P <passowrd> user set password
```
不知道密码情况下重置，首先查看当前用户列表及权限，确保权限足够：
```sh
ipmitool user list 1
```
重置用户密码：
```sh
ipmitool user set password 1 <user ID> <new password>
```
设置风扇转速（raw）
读取出传感器仓库条目(sdr)

以上内容部分来自AI，大部分参考文档：[深入理解ipmitool：揭秘BMC与IPMI的智能服务器管理（带外管理）](https://cloud.tencent.com/developer/article/2375158)

## 待补充