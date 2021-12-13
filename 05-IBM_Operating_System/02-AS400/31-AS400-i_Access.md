# AS400-i_Access
## 安装iAccess
### Linux系统
```
[root@VM-0-6-centos tmp]# rpm -i ibm-iaccess-1.1.0.15-1.0.x86_64.rpm
error: Failed dependencies:
	/usr/bin/odbcinst is needed by ibm-iaccess-1.1.0.15-1.0.x86_64
	libodbcinst.so.2()(64bit) is needed by ibm-iaccess-1.1.0.15-1.0.x86_64

[root@VM-0-6-centos tmp]# yum search unixODBC.x86_64

[root@VM-0-6-centos tmp]# yum install unixODBC.x86_64
```
以上是草稿，具体安装回头再试。
## System i导航器软件

## WEB Navigator for i
### WEB界面导航器卡
&#8195;&#8195;有时候使用WEB登录Navigator for i时候很卡，在登录界面那里卡住进不去，可以重启下HTTP服务相关JOB或子系统，对应子系统是`QHTTPSVR`，首先停止相关作业或子系统(直接停子系统可能会影响其它相关HTTP服务)，然后启动：
```
STRTCPSVR SERVER(*HTTP) HTTPSVR(*ADMIN)
```
然后再次登录：
```
http://<system_name or IP>:2001/
```
官方参考链接：[Configuring an Integrated Web Application Server](https://www.ibm.com/docs/en/i/7.2?topic=browser-configuring-integrated-web-application-server)
## IBM个人通信
&#8195;&#8195;IBM Personal communication软件可以在官方下载，IBM i Access中也有集成Personal communication，完全一样，二选一即可。
### 常用功能键
功能键|描述
:---|:---
F1|对命令参数或参数值进行帮助
F4|列出参数的所有有效值
F5|刷新屏幕，清除所有的参数值
F10|列出命令的所有参数
F11|激活/取消 参数的关键字显示
F12|退出

## 待补充