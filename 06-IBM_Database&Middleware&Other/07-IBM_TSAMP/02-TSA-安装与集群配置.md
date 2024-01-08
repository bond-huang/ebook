# TSA-安装与集群配置
## TSA安装
### TSA软件安装
两个节点上解压安装包：
```
tar -xvf 4.1.0-TIV-SAMP-Linux64-FP0004.tar.gz
cd SAM4104MPLinux64
```
将license文件放到安装包的目录，运行安装程序：
```
[root@node1 SAM4104MPLinux64]# ./installSAM
......
installSAM: To accept all terms of the preceding License Agreement and License Information type 'y', 
anything else to decline. 
......
installSAM: The following license is installed: 
Product: IBM Tivoli System Automation for Multiplatforms 4.1.0.0
Creation date: Fri 16 Aug 2013 12:00:01 AM CST
Expiration date: Thu 31 Dec 2037 12:00:01 AM CST
......
Subsystem         Group            PID     Status 
 ctrmc            rsct             64383   active
installSAM: Warning: Must set CT_MANAGEMENT_SCOPE=2 
......
installSAM: All packages were installed successfully
```
两个节点都执行安装。安装完成后检查license：
```
[root@node1 SAM4104MPLinux64]# samlicm -s
Product: IBM Tivoli System Automation for Multiplatforms 4.1.0.0
Creation date: Fri 16 Aug 2013 12:00:01 AM CST
Expiration date: Thu 31 Dec 2037 12:00:01 AM CST
```
### 安装报错
#### 缺少依赖
安装时候报错示例：
```
prereqSAM: Error: Prerequisite checking for the ITSAMP installation failed:  RHEL 7.2 x86_64
prereqSAM: One or more required packages are not installed: ksh, perl-Sys-Syslog
prereqSAM: For details, refer to the 'Error:' entries in the log file:  /tmp/installSAM.1.log
```
挂载安装光盘进行安装：
```
[root@node1 SAM4104MPLinux64]# mount /dev/cdrom /mnt
mount: /dev/sr0 is write-protected, mounting read-only
[root@node1 SAM4104MPLinux64]# yum install ksh
[root@node1 mnt]# yum install perl-Sys-Syslog
[root@node1 SAM4104MPLinux64]# ./installSAM
```
## 集群配置
### 基础配置
两个节点上都设置环境变量：
```sh
export CT_MANAGEMENT_SCOPE=2
```
创建应用启停脚本及其所在文件夹：
```sh
mkdir -p /root/cluster/scripts
touch /root/cluster/scripts/mqctrl
chmod 777 /root/cluster/scripts/mqctrl
```
在存储端分配共享磁盘到两个节点上。
### 配置共享卷组
#### 2节点node2上操作
创建PV：
```
[root@node2 SAM4104MPLinux64]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created
[root@node2 SAM4104MPLinux64]# pvdisplay
  --- Physical volume ---
......
  "/dev/sdb" is a new physical volume of "300.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               300.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               8D3o2D-Cdmv-23cP-FmL7-eMQF-f0eL-SsBO08
```
查看pv：
```
[root@node2 SAM4104MPLinux64]# pvs
  PV         VG   Fmt  Attr PSize   PFree  
  /dev/sda2  rhel lvm2 a--  249.51g   4.00m
  /dev/sdb        lvm2 ---  300.00g 300.00g
  
  vgcreate /dev/datavg /dev/sdb
```
创建vg：
```
[root@node2 SAM4104MPLinux64]# vgcreate /dev/datavg /dev/sdb
  Volume group "datavg" successfully created
[root@node2 SAM4104MPLinux64]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree  
  datavg   1   0   0 wz--n- 300.00g 300.00g
  rhel     1   3   0 wz--n- 249.51g   4.00m
```
创建LV：
```
[root@node2 SAM4104MPLinux64]# lvcreate -l +100%FREE -n /dev/datavg/datalv /dev/datavg
  Logical volume "datalv" created.
[root@node2 SAM4104MPLinux64]# lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  datalv datavg -wi-a----- 300.00g                                                    
  home   rhel   -wi-ao---- 100.00g                                                    
  root   rhel   -wi-ao---- 117.50g                                                    
  swap   rhel   -wi-ao----  32.00g 
```
创建文件系统和挂载点：
```sh
mkfs.ext4 /dev/datavg/datalv
mkdir /data
```
挂载文件系统并查看：
```
[root@node2 SAM4104MPLinux64]# mount /dev/datavg/datalv /data
[root@node2 SAM4104MPLinux64]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root      118G  7.0G  111G   6% /
devtmpfs                   7.9G     0  7.9G   0% /dev
tmpfs                      7.9G  144K  7.9G   1% /dev/shm
tmpfs                      7.9G  8.9M  7.9G   1% /run
tmpfs                      7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-home      100G  293M  100G   1% /home
/dev/sda1                  497M  140M  357M  29% /boot
tmpfs                      1.6G   16K  1.6G   1% /run/user/0
/dev/sr0                   3.8G  3.8G     0 100% /mnt
/dev/mapper/datavg-datalv  296G   65M  281G   1% /data
```
卸载文件系统并停用卷组：
```
[root@node2 SAM4104MPLinux64]#umount /data
  lvchange -an /dev/datavg/datalv
  vgchange -an /dev/datavg
[root@node2 SAM4104MPLinux64]#vgchange -an /dev/datavg
  0 logical volume(s) in volume group "datavg" now active
```
#### 1节点node1上操作
创建挂载点：
```sh
mkdir /data
```
查看LV:
```
[root@node1 ~]# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rhel -wi-ao---- 100.00g                                                    
  root rhel -wi-ao---- 117.50g 
```
如果没看到，需要重启系统扫描下磁盘：
```
[root@node1 ~]# lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  datalv datavg -wi-a----- 300.00g                                                    
  home   rhel   -wi-ao---- 100.00g                                                    
  root   rhel   -wi-ao---- 117.50g                                                    
  swap   rhel   -wi-ao----  32.00g  
```
激活卷组：
```
[root@node1 ~]# vgchange -ay /dev/datavg
  1 logical volume(s) in volume group "datavg" now active
```
挂载文件系统：
```
[root@node1 ~]# mount /dev/datavg/datalv /data
[root@node1 ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root      118G  7.0G  111G   6% /
devtmpfs                   7.9G     0  7.9G   0% /dev
tmpfs                      7.9G   84K  7.9G   1% /dev/shm
tmpfs                      7.9G  8.8M  7.9G   1% /run
tmpfs                      7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-home      100G  293M  100G   1% /home
/dev/sda1                  497M  140M  357M  29% /boot
tmpfs                      1.6G   12K  1.6G   1% /run/user/42
tmpfs                      1.6G     0  1.6G   0% /run/user/0
/dev/mapper/datavg-datalv  296G   65M  281G   1% /data
```
卸载文件系统并停用卷组：
```
[root@node1 SAM4104MPLinux64]#umount /data
  lvchange -an /dev/datavg/datalv
  vgchange -an /dev/datavg
[root@node1 SAM4104MPLinux64]#vgchange -an /dev/datavg
  0 logical volume(s) in volume group "datavg" now active
```
### domain配置
两个节点配置host表：
```sh
#######TSA#########
10.10.145.116 node1
10.10.145.117 node2
10.10.145.118 svcIP
```
两个节点运行：
```sh
preprpnode node1 node2
```
node1集群配置domain：
```
[root@node1 ~]# mkrpdomain TSA_Domain node1 node2
[root@node1 ~]# lsrpdomain
Name       OpState RSCTActiveVersion MixedVersions TSPort GSPort 
TSA_Domain Offline 3.2.3.2           No            12347  12348  
```
node1启动并查看domain：
```
[root@node1 ~]# startrpdomain TSA_Domain
[root@node1 ~]# lsrpdomain
Name       OpState        RSCTActiveVersion MixedVersions TSPort GSPort 
TSA_Domain Pending online 3.2.3.2           No            12347  12348  
[root@node1 ~]# lsrpdomain
Name       OpState RSCTActiveVersion MixedVersions TSPort GSPort 
TSA_Domain Online  3.2.3.2           No            12347  12348 
```
node2查看状态：
```
node2查看状态：
[root@node2 ~]# lsrpdomain 
Name       OpState RSCTActiveVersion MixedVersions TSPort GSPort 
TSA_Domain Online  3.2.3.2           No            12347  12348
```
### 网络配置
node1上配置网络：
```sh
mkequ NetInt IBM.NetworkInterface:eth0:node1,eth0:node2
```
两个节点分别查看：
```
[root@node1 ~]# lsrsrc -l IBM.NetworkInterface
Resource Persistent Attributes for IBM.NetworkInterface
resource 1:
        Name             = "eth0"
        DeviceName       = ""
        IPAddress        = "10.10.145.116"
        SubnetMask       = "255.255.255.0"
        Subnet           = "10.10.145.0"
        CommGroup        = "CG1"
        HeartbeatActive  = 1
        Aliases          = {}
        DeviceSubType    = 1
        LogicalID        = 0
        NetworkID        = 0
        NetworkID64      = 0
        PortID           = 0
        HardwareAddress  = "00:50:56:9f:0f:a1"
        DevicePathName   = ""
        IPVersion        = 4
        Role             = 0
        ActivePeerDomain = "TSA_Domain"
        NodeNameList     = {"node1"}
resource 2:
        Name             = "eth0"
        DeviceName       = ""
        IPAddress        = "10.10.145.117"
        SubnetMask       = "255.255.255.0"
        Subnet           = "10.10.145.0"
        CommGroup        = "CG1"
        HeartbeatActive  = 1
        Aliases          = {}
        DeviceSubType    = 1
        LogicalID        = 0
        NetworkID        = 0
        NetworkID64      = 0
        PortID           = 0
        HardwareAddress  = "00:50:56:9f:46:c9"
        DevicePathName   = ""
        IPVersion        = 4
        Role             = 0
        ActivePeerDomain = "TSA_Domain"
        NodeNameList     = {"node2"}
```
网络心跳配置：
```sh
mkrsrc IBM.TieBreaker Type="EXEC" Name="networktb" DeviceInfo='PATHNAME=/usr/sbin/rsct/bin/samtb_net Address=10.10.145.254  Log=1' PostReserveWaitTime=30
```
查看配置：
```
[root@node1 ~]# lsrsrc -l IBM.TieBreaker
Resource Persistent Attributes for IBM.TieBreaker
resource 1:
        Name                = "networktb"
        Type                = "EXEC"
        DeviceInfo          = "PATHNAME=/usr/sbin/rsct/bin/samtb_net Address=10.10.145.254  Log=1"
        ReprobeData         = ""
        ReleaseRetryPeriod  = 0
        HeartbeatPeriod     = 0
        PreReserveWaitTime  = 0
        PostReserveWaitTime = 30
        NodeInfo            = {}
        ActivePeerDomain    = "TSA_Domain"
resource 2:
        Name                = "Success"
        Type                = "Success"
        DeviceInfo          = ""
        ReprobeData         = ""
        ReleaseRetryPeriod  = 0
        HeartbeatPeriod     = 0
        PreReserveWaitTime  = 0
        PostReserveWaitTime = 0
        NodeInfo            = {}
        ActivePeerDomain    = "TSA_Domain"
resource 3:
        Name                = "Fail"
        Type                = "Fail"
        DeviceInfo          = ""
        ReprobeData         = ""
        ReleaseRetryPeriod  = 0
        HeartbeatPeriod     = 0
        PreReserveWaitTime  = 0
        PostReserveWaitTime = 0
        NodeInfo            = {}
        ActivePeerDomain    = "TSA_Domain"
resource 4:
        Name                = "Operator"
        Type                = "Operator"
        DeviceInfo          = ""
        ReprobeData         = ""
        ReleaseRetryPeriod  = 0
        HeartbeatPeriod     = 0
        PreReserveWaitTime  = 0
        PostReserveWaitTime = 0
        NodeInfo            = {}
        ActivePeerDomain    = "TSA_Domain"
```
查网络服务状态：
```
[root@node1 ~]# lssam
Online IBM.Equivalency:NetInt
        |- Online IBM.NetworkInterface:eth0:node1
        '- Online IBM.NetworkInterface:eth0:node2
```
### 配置Application
在/root/cluster/scripts创建mq.def配置文件,内容示例：
```ini
PersistentResourceAttributes::
Name="mq"
StartCommand="/root/cluster/scripts/mqctrl start"
StopCommand="/root/cluster/scripts/mqctrl stop"
MonitorCommand="/root/cluster/scripts/mqctrl status"
MonitorCommandPeriod=5
MonitorCommandTimeout=60
NodeNameList={"node1","node2"}
StartCommandTimeout=60
StopCommandTimeout=60
UserName="root"
ResourceType=1
```
更改权限：
```
[root@node1 scripts]# ls -l
total 8
-rwxrwxrwx 1 root root 582 Dec  6 21:51 mqctrl
-rw-r----- 1 root root 357 Dec  6 21:59 mq.def
[root@node1 scripts]# chmod 755 mq.def 
```
从配置文件创建应用：
```sh
mkrsrc -f mq.def IBM.Application
```
查看创建配置：
```
[root@node1 scripts]# lsrsrc -l IBM.Application
Resource Persistent Attributes for IBM.Application
resource 1:
        Name                  = "mq"
        ResourceType          = 0
        AggregateResource     = "0x2028 0xffff 0x869b2142 0xc615b8e4 0x979e42f9 0x04edf620"
        StartCommand          = "/root/cluster/scripts/mqctrl start"
        StopCommand           = "/root/cluster/scripts/mqctrl stop"
        MonitorCommand        = "/root/cluster/scripts/mqctrl status"
        MonitorCommandPeriod  = 5
        MonitorCommandTimeout = 60
        StartCommandTimeout   = 60
        StopCommandTimeout    = 60
        UserName              = "root"
        RunCommandsSync       = 1
        ProtectionMode        = 0
        HealthCommand         = ""
        HealthCommandPeriod   = 10
        HealthCommandTimeout  = 5
        InstanceName          = ""
        InstanceLocation      = ""
        SetHealthState        = 0
        MovePrepareCommand    = ""
        MoveCompleteCommand   = ""
        MoveCancelCommand     = ""
        CleanupList           = {}
        CleanupCommand        = ""
        CleanupCommandTimeout = 10
        ProcessCommandString  = ""
        ResetState            = 0
        ReRegistrationPeriod  = 0
        CleanupNodeList       = {}
        MonitorUserName       = ""
        ActivePeerDomain      = "TSA_Domain"
        NodeNameList          = {"node2"}
resource 2:
        Name                  = "mq"
        ResourceType          = 0
        AggregateResource     = "0x2028 0xffff 0x869b2142 0xc615b8e4 0x979e42f9 0x04edf620"
        StartCommand          = "/root/cluster/scripts/mqctrl start"
        StopCommand           = "/root/cluster/scripts/mqctrl stop"
        MonitorCommand        = "/root/cluster/scripts/mqctrl status"
        MonitorCommandPeriod  = 5
        MonitorCommandTimeout = 60
        StartCommandTimeout   = 60
        StopCommandTimeout    = 60
        UserName              = "root"
        RunCommandsSync       = 1
        ProtectionMode        = 0
        HealthCommand         = ""
        HealthCommandPeriod   = 10
        HealthCommandTimeout  = 5
        InstanceName          = ""
        InstanceLocation      = ""
        SetHealthState        = 0
        MovePrepareCommand    = ""
        MoveCompleteCommand   = ""
        MoveCancelCommand     = ""
        CleanupList           = {}
        CleanupCommand        = ""
        CleanupCommandTimeout = 10
        ProcessCommandString  = ""
        ResetState            = 0
        ReRegistrationPeriod  = 0
        CleanupNodeList       = {}
        MonitorUserName       = ""
        ActivePeerDomain      = "TSA_Domain"
        NodeNameList          = {"node1"}
resource 3:
        Name                  = "mq"
        ResourceType          = 1
        AggregateResource     = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        StartCommand          = "/root/cluster/scripts/mqctrl start"
        StopCommand           = "/root/cluster/scripts/mqctrl stop"
        MonitorCommand        = "/root/cluster/scripts/mqctrl status"
        MonitorCommandPeriod  = 5
        MonitorCommandTimeout = 60
        StartCommandTimeout   = 60
        StopCommandTimeout    = 60
        UserName              = "root"
        RunCommandsSync       = 1
        ProtectionMode        = 0
        HealthCommand         = ""
        HealthCommandPeriod   = 10
        HealthCommandTimeout  = 5
        InstanceName          = ""
        InstanceLocation      = ""
        SetHealthState        = 0
        MovePrepareCommand    = ""
        MoveCompleteCommand   = ""
        MoveCancelCommand     = ""
        CleanupList           = {}
        CleanupCommand        = ""
        CleanupCommandTimeout = 10
        ProcessCommandString  = ""
        ResetState            = 0
        ReRegistrationPeriod  = 0
        CleanupNodeList       = {}
        MonitorUserName       = ""
        ActivePeerDomain      = "TSA_Domain"
        NodeNameList          = {"node1","node2"}
```
### 配置服务IP
node1上配置，命令：
```sh
mkrsrc IBM.ServiceIP NodeNameList="{'node1','node2'}" Name="svcip" NetMask=255.255.255.0 IPAddress=10.10.145.118 ResourceType=1
```
查看配置：
```
[root@node1 scripts]# lsrsrc -l IBM.ServiceIP
Resource Persistent Attributes for IBM.ServiceIP
resource 1:
        Name              = "svcip"
        ResourceType      = 0
        AggregateResource = "0x2029 0xffff 0x869b2142 0xc615b8e4 0x979e4346 0x7e288e68"
        IPAddress         = "10.10.145.118"
        NetMask           = "255.255.255.0"
        ProtectionMode    = 1
        NetPrefix         = 0
        ActivePeerDomain  = "TSA_Domain"
        NodeNameList      = {"node2"}
resource 2:
        Name              = "svcip"
        ResourceType      = 0
        AggregateResource = "0x2029 0xffff 0x869b2142 0xc615b8e4 0x979e4346 0x7e288e68"
        IPAddress         = "10.10.145.118"
        NetMask           = "255.255.255.0"
        ProtectionMode    = 1
        NetPrefix         = 0
        ActivePeerDomain  = "TSA_Domain"
        NodeNameList      = {"node1"}
resource 3:
        Name              = "svcip"
        ResourceType      = 1
        AggregateResource = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        IPAddress         = "10.10.145.118"
        NetMask           = "255.255.255.0"
        ProtectionMode    = 1
        NetPrefix         = 0
        ActivePeerDomain  = "TSA_Domain"
        NodeNameList      = {"node1","node2"}
```
### 配置共享文件系统
node1上操作，配置命令如下：
```sh
mkrsrc IBM.AgFileSystem Name=mqdata Vfs=ext4 Force=1 ResourceType=1 NodeNameList={"node1","node2"} DeviceName=/dev/datavg/datalv MountPoint=/data
```
查看共享文件系统配置：
```
[root@node1 scripts]# lsrsrc -l IBM.AgFileSystem
Resource Persistent Attributes for IBM.AgFileSystem
resource 1:
        ResourceHandle      = "0x6038 0xffff 0x5bd29117 0xc615b8e4 0x179e4387 0xac4d0d78"
        Name                = "mqdata"
        ResourceType        = 0
        MountPoint          = "/data"
        DeviceName          = "/dev/datavg/datalv"
        Vfs                 = "ext4"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e4387 0xac4b1978"
        ContainerResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        GhostDevice         = 0
        ResourceId          = "RID_869b2142_a7bf385f_979e4387_ac4b1978"
        ProtectionMode      = 1
        UserControl         = 1
        SysMountPoint       = ""
        Label               = ""
        FSID                = ""
        PreOnlineMethod     = 0
        ContainerResourceId = ""
        AutoMonitor         = 1
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2"}
resource 2:
        ResourceHandle      = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e4387 0xac4b1978"
        Name                = "mqdata"
        ResourceType        = 1
        MountPoint          = "/data"
        DeviceName          = "/dev/datavg/datalv"
        Vfs                 = "ext4"
        AggregateResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        ContainerResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        GhostDevice         = 0
        ResourceId          = "RID_869b2142_a7bf385f_979e4387_ac4b1978"
        ProtectionMode      = 1
        UserControl         = 1
        SysMountPoint       = ""
        Label               = ""
        FSID                = ""
        PreOnlineMethod     = 0
        ContainerResourceId = ""
        AutoMonitor         = 1
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1","node2"}
resource 3:
        ResourceHandle      = "0x6038 0xffff 0x82321854 0xa7bf385f 0x179e4387 0xac4c3a88"
        Name                = "mqdata"
        ResourceType        = 0
        MountPoint          = "/data"
        DeviceName          = "/dev/datavg/datalv"
        Vfs                 = "ext4"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e4387 0xac4b1978"
        ContainerResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        GhostDevice         = 0
        ResourceId          = "RID_869b2142_a7bf385f_979e4387_ac4b1978"
        ProtectionMode      = 1
        UserControl         = 1
        SysMountPoint       = ""
        Label               = ""
        FSID                = ""
        PreOnlineMethod     = 0
        ContainerResourceId = ""
        AutoMonitor         = 1
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1"}
resource 4:
        ResourceHandle      = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4e3278"
        Name                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        ResourceType        = 1
        MountPoint          = ""
        DeviceName          = ""
        Vfs                 = "xfs"
        AggregateResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        ContainerResource   = "0x2037 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4126a0"
        GhostDevice         = 0
        ResourceId          = "d3549b88_1b1f_4ffc_be2c_a7cfdf1895e0"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/boot"
        Label               = ""
        FSID                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        PreOnlineMethod     = 0
        ContainerResourceId = "VMware__0_0_1"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2","node1"}
resource 5:
        ResourceHandle      = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4b5800"
        Name                = "36fce73c-0756-46fb-aafb-63760184acd2"
        ResourceType        = 1
        MountPoint          = ""
        DeviceName          = ""
        Vfs                 = "xfs"
        AggregateResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        ContainerResource   = "0x204c 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb46f300"
        GhostDevice         = 0
        ResourceId          = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/"
        Label               = ""
        FSID                = "36fce73c-0756-46fb-aafb-63760184acd2"
        PreOnlineMethod     = 0
        ContainerResourceId = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2","node1"}
resource 6:
        ResourceHandle      = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4cbf60"
        Name                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        ResourceType        = 1
        MountPoint          = ""
        DeviceName          = ""
        Vfs                 = "ext4"
        AggregateResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        ContainerResource   = "0x204c 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb483b20"
        GhostDevice         = 0
        ResourceId          = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = ""
        Label               = ""
        FSID                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        PreOnlineMethod     = 0
        ContainerResourceId = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        AutoMonitor         = 0
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2","node1"}
resource 7:
        ResourceHandle      = "0x6038 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xea8e1ab0"
        Name                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/sda1"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4e3278"
        ContainerResource   = "0x6037 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xea64d830"
        GhostDevice         = 0
        ResourceId          = "d3549b88_1b1f_4ffc_be2c_a7cfdf1895e0"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/boot"
        Label               = ""
        FSID                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        PreOnlineMethod     = 0
        ContainerResourceId = "VMware__0_0_1"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2"}
resource 8:
        ResourceHandle      = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb49b608"
        Name                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        ResourceType        = 1
        MountPoint          = ""
        DeviceName          = ""
        Vfs                 = "xfs"
        AggregateResource   = "0x3fff 0xffff 0x00000000 0x00000000 0x00000000 0x00000000"
        ContainerResource   = "0x204c 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb459b40"
        GhostDevice         = 0
        ResourceId          = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/home"
        Label               = ""
        FSID                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        PreOnlineMethod     = 0
        ContainerResourceId = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2","node1"}
resource 9:
        ResourceHandle      = "0x6038 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc144f390"
        Name                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/sda1"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4e3278"
        ContainerResource   = "0x6037 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc128e3f8"
        GhostDevice         = 0
        ResourceId          = "d3549b88_1b1f_4ffc_be2c_a7cfdf1895e0"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/boot"
        Label               = ""
        FSID                = "d3549b88-1b1f-4ffc-be2c-a7cfdf1895e0"
        PreOnlineMethod     = 0
        ContainerResourceId = "VMware__0_0_1"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1"}
resource 10:
        ResourceHandle      = "0x6038 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xead40520"
        Name                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/datavg/datalv"
        Vfs                 = "ext4"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4cbf60"
        ContainerResource   = "0x604c 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xeaab7a38"
        GhostDevice         = 0
        ResourceId          = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = ""
        Label               = ""
        FSID                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        PreOnlineMethod     = 0
        ContainerResourceId = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        AutoMonitor         = 0
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2"}
resource 11:
        ResourceHandle      = "0x6038 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xead8c3f8"
        Name                = "36fce73c-0756-46fb-aafb-63760184acd2"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/rhel/root"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4b5800"
        ContainerResource   = "0x604c 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xeaaf0090"
        GhostDevice         = 0
        ResourceId          = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/"
        Label               = ""
        FSID                = "36fce73c-0756-46fb-aafb-63760184acd2"
        PreOnlineMethod     = 0
        ContainerResourceId = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2"}
resource 12:
        ResourceHandle      = "0x6038 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xeabea090"
        Name                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/rhel/home"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb49b608"
        ContainerResource   = "0x604c 0xffff 0x5bd29117 0xc615b8e4 0x179e406b 0xea66bc90"
        GhostDevice         = 0
        ResourceId          = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/home"
        Label               = ""
        FSID                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        PreOnlineMethod     = 0
        ContainerResourceId = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node2"}
resource 13:
        ResourceHandle      = "0x6038 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc131fc18"
        Name                = "36fce73c-0756-46fb-aafb-63760184acd2"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/rhel/root"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4b5800"
        ContainerResource   = "0x604c 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc12a1c78"
        GhostDevice         = 0
        ResourceId          = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/"
        Label               = ""
        FSID                = "36fce73c-0756-46fb-aafb-63760184acd2"
        PreOnlineMethod     = 0
        ContainerResourceId = "NMGnqU_YZOg_VPPc_Zb4e_finv_9yRe_cizzZo"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1"}
resource 14:
        ResourceHandle      = "0x6038 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc13186e8"
        Name                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/rhel/home"
        Vfs                 = "xfs"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb49b608"
        ContainerResource   = "0x604c 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc129e1e0"
        GhostDevice         = 0
        ResourceId          = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = "/home"
        Label               = ""
        FSID                = "3d5018e4-465b-4a4a-a128-cf5ed652f033"
        PreOnlineMethod     = 0
        ContainerResourceId = "UZshSI_VMtT_kGAV_7GcJ_kAhY_t3Iv_ONl5Vk"
        AutoMonitor         = 0
        Options             = "defaults"
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1"}
resource 15:
        ResourceHandle      = "0x6038 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc13261a8"
        Name                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        ResourceType        = 0
        MountPoint          = ""
        DeviceName          = "/dev/datavg/datalv"
        Vfs                 = "ext4"
        AggregateResource   = "0x2038 0xffff 0x869b2142 0xa7bf385f 0x979e406b 0xeb4cbf60"
        ContainerResource   = "0x604c 0xffff 0x82321854 0xa7bf385f 0x179e406b 0xc12a5710"
        GhostDevice         = 0
        ResourceId          = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        ProtectionMode      = 1
        UserControl         = 0
        SysMountPoint       = ""
        Label               = ""
        FSID                = "61a076f5-dbe5-433b-ae1e-d15216e787ba"
        PreOnlineMethod     = 0
        ContainerResourceId = "E8hond_exSI_mBoK_j0Wi_E7qh_Zr3I_rw09qR"
        AutoMonitor         = 0
        Options             = ""
        PreOfflineMethod    = 0
        ActivePeerDomain    = "TSA_Domain"
        NodeNameList        = {"node1"}
```
### 激活网络心跳
命令示例：
```
[root@node1 scripts]# lsrsrc -c IBM.PeerNode OpQuorumTieBreaker
Resource Class Persistent Attributes for IBM.PeerNode
resource 1:
        OpQuorumTieBreaker = "Operator"
[root@node1 scripts]# chrsrc -c IBM.PeerNode OpQuorumTieBreaker="networktb"
[root@node1 scripts]# lsrsrc -c IBM.PeerNode OpQuorumTieBreaker
Resource Class Persistent Attributes for IBM.PeerNode
resource 1:
        OpQuorumTieBreaker = "networktb"
```
### 配置资源组
创建资源组命令：
```sh
mkrg mqrg
```
将资源添加到资源组：
```sh
addrgmbr -g mqrg IBM.ServiceIP:svcip
addrgmbr -g mqrg IBM.Application:mq
addrgmbr -g mqrg IBM.AgFileSystem:mqdata
```
查看集群状态：
```
[root@node1 scripts]# lssam
Offline IBM.ResourceGroup:mqrg Nominal=Offline
        |- Offline IBM.AgFileSystem:mqdata
                |- Offline IBM.AgFileSystem:mqdata:node1
                '- Offline IBM.AgFileSystem:mqdata:node2
        |- Offline IBM.Application:mq
                |- Offline IBM.Application:mq:node1
                '- Offline IBM.Application:mq:node2
        '- Offline IBM.ServiceIP:svcip
                |- Offline IBM.ServiceIP:svcip:node1
                '- Offline IBM.ServiceIP:svcip:node2
Online IBM.Equivalency:NetInt
        |- Online IBM.NetworkInterface:eth0:node1
        '- Online IBM.NetworkInterface:eth0:node2
```
### 配置依赖关系
配置命令示例：
```sh
mkrel -p dependson -S IBM.ServiceIP:svcip -G IBM.Equivalency:NetInt  mqip_net
mkrel -p dependson -S IBM.AgFileSystem:mqdata -G IBM.ServiceIP:svcip mqdata_mqip
mkrel -p dependson -S IBM.Application:mq -G IBM.AgFileSystem:mqdata  mq_mqdata
```
查看配置：
```
[root@node1 scripts]# lsrel
Displaying Managed Relations :

Name        Class:Resource:Node[Source] ResourceGroup[Source] 
mq_mqdata   IBM.Application:mq          mqrg                  
mqip_net    IBM.ServiceIP:svcip         mqrg                  
mqdata_mqip IBM.AgFileSystem:mqdata     mqrg    
```
### 脚本配置
一个IBM MQ程序启停脚本示例：
```sh
#!/bin/bash
ACTION=${1}
OPSTATE_ONLINE=1
OPSTATE_OFFLINE=2
case ${ACTION} in
start)
     su - mqm -c "strmqm CS_QMGR" >/dev/null 2>&1
     logger -i -t "SAM-mq" "mq started"
     RC=0
     ;;
stop)
     su - mqm -c "endmqm -i CS_QMGR" >/dev/null 2>&1
     logger -i -t "SAM-mq" "mq stopped"
     RC=0
     ;;
status)
     PortNum=`netstat -lnt|grep 1414|grep -v grep|wc -l`
     if [ $PortNum == 1 ]
     then
         RC=${OPSTATE_ONLINE}
     else
         RC=${OPSTATE_OFFLINE}
     fi
     echo "mq status $RC"
     logger -i -t "SAM-mq" "mq status : $RC"
     ;;
esac
exit $RC
```
### 启动集群及切换
启动操作：
```
[root@node1 scripts]# chrg -o online mqrg
[root@node1 scripts]# lssam
Pending online IBM.ResourceGroup:mqrg Nominal=Online
        |- Online IBM.AgFileSystem:mqdata
                |- Online IBM.AgFileSystem:mqdata:node1
                '- Offline IBM.AgFileSystem:mqdata:node2
        |- Pending online IBM.Application:mq
                |- Pending online IBM.Application:mq:node1
                '- Offline IBM.Application:mq:node2
        '- Online IBM.ServiceIP:svcip
                |- Online IBM.ServiceIP:svcip:node1
                '- Offline IBM.ServiceIP:svcip:node2
Online IBM.Equivalency:NetInt
        |- Online IBM.NetworkInterface:eth0:node1
        '- Online IBM.NetworkInterface:eth0:node2
```
### 配置报错
#### 通信问题
报错示例：
```
[root@node1 ~]# mkrpdomain TSA_Domain node1 node2
2632-044 The domain cannot be created due to the following errors that were detected while harvesting information from the target nodes:
node1: 2645-061 The requesting node cannot be authenticated by the target node.
```
检查网络，网络没问题检查`/etc/hosts`，两个节点上注释或删掉下面行:
```
## 127.0.1.1       node1.localdomain node1
```
## 待补充