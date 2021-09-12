
[root@VM-0-6-centos tmp]# rpm -i ibm-iaccess-1.1.0.15-1.0.x86_64.rpm
error: Failed dependencies:
	/usr/bin/odbcinst is needed by ibm-iaccess-1.1.0.15-1.0.x86_64
	libodbcinst.so.2()(64bit) is needed by ibm-iaccess-1.1.0.15-1.0.x86_64

[root@VM-0-6-centos tmp]# yum search unixODBC.x86_64


[root@VM-0-6-centos tmp]# yum install unixODBC.x86_64