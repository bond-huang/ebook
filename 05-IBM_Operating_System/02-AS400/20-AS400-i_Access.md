# AS400-i_Access
## 安装iAccess
### Linux系统
[root@VM-0-6-centos tmp]# rpm -i ibm-iaccess-1.1.0.15-1.0.x86_64.rpm
error: Failed dependencies:
	/usr/bin/odbcinst is needed by ibm-iaccess-1.1.0.15-1.0.x86_64
	libodbcinst.so.2()(64bit) is needed by ibm-iaccess-1.1.0.15-1.0.x86_64

[root@VM-0-6-centos tmp]# yum search unixODBC.x86_64


[root@VM-0-6-centos tmp]# yum install unixODBC.x86_64
以上是草稿，具体安装回头再试。

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