# KVM-RHEL8安装部署
参考文档：[Virtualization Deployment and Administration Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/index)
## 系统要求
我使用系统是Red Hat Enterprise Linux 8.0，笔记本通过VMware虚拟的。
### 主机系统要求
要求如下：
- 6 GB可用磁盘空间
- 2 GB RAM
### KVM虚拟机监控程序要求
CPU需要有虚拟功能并开启了，VMware开启步骤：
- 停机状态选中虚拟机，进入设置选项
- 选中处理器，在虚拟化选项中勾选：虚拟化Intel VT-x/EPT或AMD-V/RVI(V)

验证方法：
```
[root@redhat8 ~]# grep -E 'svm|vmx' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 
clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 invpcid rtm rdseed adx smap xsaveopt arat flush_l1d arch_capabilitiesflags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 
clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 invpcid rtm rdseed adx smap xsaveopt arat flush_l1d arch_capabilities
```
附加检查，验证模块是否已加载到内核中：
```
[root@redhat8 ~]# lsmod | grep kvm
kvm_intel             245760  0
kvm                   745472  1 kvm_intel
irqbypass              16384  1 kvm
```
### KVM GUEST虚拟机兼容性
以下URL解释了Red Hat Enterprise Linux的处理器和内存量限制：
- 对于主机系统：[https://access.redhat.com/articles/rhel-limits](https://access.redhat.com/articles/rhel-limits)
- 对于KVM虚拟机管理程序：[https://access.redhat.com/articles/rhel-kvm-limits](https://access.redhat.com/articles/rhel-kvm-limits)

### 支持GUEST虚拟机CPU型号
在`cpu_map.xml`文件中 包含受支持的CPU型号和功能的完整列表:
```
# cat /usr/share/libvirt/cpu_map.xml
```
## 安装虚拟化软件包
### 在RHEL安装过程中安装虚拟化软件包
跳过，系统已经安装了，如果没安装RHEL，参考官方文档进行。
### 在现有RHEL上安装虚拟化软件包
#### 安装虚拟化软件包
在RHEL上使用虚拟化，至少需要安装以下软件包：
- qemu-kvm：此软件包提供了用户级KVM仿真器，并促进了主机与guest虚拟机之间的通信
- qemu-img：该软件包为来宾虚拟机提供磁盘管理。qemu-img包安装为的依赖QEMU的KVM包
- libvirt：此软件包提供用于与虚拟机管理程序和主机系统进行交互的服务器和主机端库，以及libvirtd用于处理库调用，管理虚拟机和控制虚拟机管理程序的守护程序

之前已经配置好了本地YUM源，挂载虚拟光驱即可：
```
[root@redhat8 /]# mount /dev/cdrom /mnt/cdrom
mount: /mnt/cdrom: WARNING: device write-protected, mounted read-only.
```
安装：
```
[root@redhat8 /]# yum install qemu-kvm libvirt
...
Package qemu-kvm-15:2.12.0-63.module+el8+2833+c7d6d092.x86_64 is already installed.
...
Installed:
  libvirt-4.5.0-23.module+el8+2800+2d311f65.x86_64                                        
  autogen-libopts-5.18.12-7.el8.x86_64                                                    
  gnutls-dane-3.6.5-2.el8.x86_64                                                          
  gnutls-utils-3.6.5-2.el8.x86_64                                                         
  libvirt-bash-completion-4.5.0-23.module+el8+2800+2d311f65.x86_64                        
  libvirt-client-4.5.0-23.module+el8+2800+2d311f65.x86_64                                 
  libvirt-daemon-config-nwfilter-4.5.0-23.module+el8+2800+2d311f65.x86_64                
Complete!
```
附加的虚拟化管理程序包，在使用虚拟化时推荐使用这些软件包：
- virt-install：该软件包提供了virt-install用于从命令行创建虚拟机的命令
- libvirt-python：该软件包包含一个模块，该模块允许以Python编程语言编写的应用程序使用libvirt API提供的接口
- virt-manager：该软件包提供了virt-manager工具，也称为Virtual Machine Manager。这是用于管理虚拟机的图形工具。它使用libvirt-client库作为管理API
- libvirt-client：此软件包提供用于访问libvirt服务器的客户端API和库。所述libvirt的客户端包包括virsh命令行工具来管理和从命令行或特殊的虚拟化壳控制虚拟机和管理程序

安装：  
```
[root@redhat8 /]# yum install virt-install
...
Installed:
  virt-install-2.0.0-5.el8.noarch                                                         
  python3-libvirt-4.5.0-1.module+el8+2529+a9686a4d.x86_64                                 
  virt-manager-common-2.0.0-5.el8.noarch                                                  
Complete!
[root@redhat8 /]# yum install libvirt-python
No match for argument: libvirt-python
Error: Unable to find a match
[root@redhat8 /]# yum install virt-manager
...
Installed:
  virt-manager-2.0.0-5.el8.noarch                                                         
Complete!
[root@redhat8 /]# yum install libvirt-client
Package libvirt-client-4.5.0-23.module+el8+2800+2d311f65.x86_64 is already installed.
```
libvirt-python没有找到，查找一下应该是这个：
```
[root@redhat8 /]# yum search libvirt
python3-libvirt.x86_64 : The libvirt virtualization API python3 binding
python3-libvirt.x86_64 : The libvirt virtualization API python3 binding
[root@redhat8 /]# yum install python3-libvirt
...
Package python3-libvirt-4.5.0-1.module+el8+2529+a9686a4d.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
#### 安装虚拟化软件包组
&#8195;&#8195;虚拟化软件包也可以从软件包组中安装。各软件包组包含内容参考官方文档。命令：`yum groupinstall package_group`示例如下：
```
# yum groupinstall "Virtualization Tools" --optional
```
选项`--optional`选将可选软件包安装在软件包组中。

## 创建虚拟机
&#8195;&#8195;在安装虚拟化软件包之后，可以使用`virt-manager`界面创建虚拟机并安装guest操作系统。或者通过参数列表或脚本使用`virt-install`命令行实用程序。
### Guest虚拟机部署注意事项
省略，详细参考官方说明。
### 通过VIRT-INSTALL创建虚拟机
&#8195;&#8195;使用`virt-install`命令从命令行创建虚拟机并在这些虚拟机上安装操作系统。`virt-install`可以交互使用或作为脚本的一部分使用，以自动创建虚拟机。如果使用交互式图形安装，则必须先安装`virt-viewer`，然后再运行`virt-install`。此外，可以`virt-install`与`kickstart`文件一起使用来启动虚拟机操作系统的无人值守安装。
#### virt-install命令行选项
虚拟客户机安装的主要必需选项：
- `--name`：虚拟机的名称
- `--memory`：以MiB为单位分配给Guest的内存（RAM）

#### Guest存储
使用以下选项之一：
- `--disk`：虚拟机的存储配置详细信息。如果使用`--disk none`，则虚拟机将没有磁盘空间
- `--filesystem`：guest虚拟机的文件系统路径

#### 安装方式
使用以下安装方法之一：
- `--location`：安装介质的位置
- `--cdrom`：用作虚拟CD-ROM设备的文件或设备。可以是ISO映像的路径，也可以是从中获取或访问最小引导ISO映像的URL。但不能是物理主机CD-ROM或DVD-ROM设备
- `--pxe`：使用PXE引导协议加载初始ramdisk和内核，以启动虚拟机安装过程
- `--import`：跳过操作系统安装过程，并围绕现有磁盘映像构建虚拟机。用于引导的设备是由disk或filesystem选项指定的第一个设备
- `--boot`：安装后VM的启动配置。此选项允许指定启动设备顺序，使用可选的内核参数永久启动内核和initrd并启用BIOS引导菜单

#### 查看帮助
查看选项的完整列表：
```
[root@redhat8 /]# virt-install --help
```
要查看选项属性的完整列表：
```
[root@redhat8 /]# virt-install --option=?
usage: virt-install --name NAME --memory MB STORAGE INSTALL [options]
virt-install: error: unrecognized arguments: --option=?
```
在运行之前virt-install，可能还需要使用qemu-img来配置存储选项。参考链接：[Using qemu-img](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/chap-Using_qemu_img)
#### 从ISO映像安装虚拟机
从ISO映像安装虚拟机示例：
```
# virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --cdrom /path/to/rhel7.iso \ 
  --os-variant rhel7 
```
#### 导入虚拟机image
从虚拟磁盘映像导入虚拟机示例：
```
# virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk /path/to/imported/disk.qcow \ 
  --import \ 
  --os-variant rhel7 
```
选项`--import`从该选项指定的虚拟磁盘image导入虚拟机`--disk /path/to/imported/disk.qcow`。
#### 从网络安装虚拟机
从网络位置安装虚拟机示例：
```
# virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --location http://example.com/path/to/os \ 
  --os-variant rhel7 
```
#### 使用PXE安装虚拟机
使用PXE引导协议安装虚拟机时，必须指定用于指定桥接网络的`--network`选项和`--pxe`选项：
```
# virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --network=bridge:br0 \ 
  --pxe \ 
  --os-variant rhel7 
```
#### 使用Kickstart安装虚拟机
使用kickstart文件安装虚拟机示例：
```
# virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --location http://example.com/path/to/os \ 
  --os-variant rhel7 \
  --initrd-inject /path/to/ks.cfg \ 
  --extra-args="ks=file:/ks.cfg console=tty0 console=ttyS0,115200n8" 
```
`initrd-inject`和`extra-args`选项指定使用Kickstarter文件安装虚拟机。 
#### 在Guest创建期间配置虚拟机网络
创建Guest虚拟机时，可以为虚拟机指定和配置网络。

带有NAT的默认网络
- 默认网络使用libvirtd的网络地址转换（NAT）虚拟网络交换机，
- 在使用带有NAT的默认网络创建来宾虚拟机之前，请确保已安装libvirt-daemon-config-network软件包
- 要为来宾虚拟机配置NAT网络，在`virt-install`中使用选项：`--network default`
- 如果未network指定任何选项，则为来宾虚拟机配置具有NAT的默认网络

官方更多参考：[Network Address Translation (NAT) with libvirt](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/chap-Network_configuration#sect-Network_configuration-Network_Address_Translation_NAT_with_libvirt)

带DHCP的桥接网络
- 当配置为桥接网络时，guest虚拟机将使用外部DHCP服务器
- 如果主机具有静态网络配置，并且guest需要与局域网（LAN）的完全入站和出站连接，则应使用此选项
- 如果将使用guest虚拟机执行实时迁移，则应使用它
- 要为guest宾虚拟机使用DHCP配置桥接网络，使用选项：`--network br0`
- 在运行之前，必须单独创建网桥`virt-install`。

有关创建网桥的详细信息:[Configuring Bridged Networking on a Red Hat Enterprise Linux 7 Host](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-Network_configuration-Bridged_networking#sect-Network_configuration-Bridged_networking_in_RHEL)

具有静态IP地址的桥接网络
- 桥接网络也可以用于将访客配置为使用静态IP地址
- 要为guest虚拟机配置具有静态IP地址的桥接网络，请使用以下选项：
```
--network br0 \
--extra-args "ip=192.168.1.2::192.168.1.1:255.255.255.0:test.example.com:eth0:none"
```

有关网络引导选项的更多信息：[Red Hat Enterprise Linux 7 Installation Guide. ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-anaconda-boot-options#sect-boot-options-installer)

要配置没有网络接口的guest虚拟机，请使用以下选项：`--network=none`

#### 安装示例
机器性能有限，磁盘空间有限，安装个两百多兆的debian系统，导入虚拟机image方式：
#### 
从虚拟磁盘映像导入虚拟机示例：
```
# virt-install \ 
  --name debian \ 
  --memory 1024 \ 
  --vcpus 1 \ 
  --disk /images/debian_wheezy_amd64_standard.qcow2 \ 
  --import \ 
```
运行：
```
[root@redhat8 images]# virt-install --name debian --memory 1024 --vcpus 1 --disk /images/d
ebian_wheezy_amd64_standard.qcow2 --importWARNING  KVM acceleration not available, using 'qemu'
WARNING  No operating system detected, VM performance may suffer. Specify an OS with --os-
variant for optimal results.WARNING  Unable to connect to graphical console: virt-viewer not installed. Please install
 the 'virt-viewer' package.WARNING  No console to launch for the guest, defaulting to --wait -1

Starting install...
Domain installation still in progress. Waiting for installation to complete.

```
shell卡在这里了，但是通过`virt-manager`打开图形界面可用看到是运行状态。

或者通过命令行查看是运行状态：
```
[root@redhat8 ~]# virsh
Welcome to virsh, the virtualization interactive terminal.
Type:  'help' for help with commands
       'quit' to quit
virsh # list
 Id    Name                           State
----------------------------------------------------
 1     debian                         running
```
卡住的shell Ctrl+C掉，提示：
```
Domain install interrupted.
Installation aborted at user request
```
尝试进入虚拟机是可以进去的：
```
[root@redhat8 ~]# virsh console 1
Connected to domain debian
Escape character is ^]

Debian GNU/Linux 7 debian-amd64 ttyS0

debian-amd64 login: root
Password: 
Linux debian-amd64 3.2.0-4-amd64 #1 SMP Debian 3.2.51-1 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@debian-amd64:~# hostname
debian-amd64
```
卡住原因可能是我之前没有开启CPU的虚拟化功能，KVM模块没有加载进去。
### 使用VIRT-MANAGER创建虚拟机
`New Virtual Machine`向导创建虚拟机过程分为五个步骤：
- 选择系统管理程序和安装类型
- 查找和配置安装介质
- 配置内存和CPU选项
- 配置虚拟机的存储
- 配置虚拟机名称，网络，体系结构和其他硬件设置

#### 安装RHEL7
&#8195;&#8195;RHEL7至少需要1GB的存储空间。但是，对于RHEL7安装和本指南中的过程，Red Hat建议至少有5GB的存储空间。我使用版本是RHEL-server-7.8，物理机磁盘空间严重不足，不想把ISO拷贝到虚拟机中，打算直接从虚拟光驱安装试试，选中光驱后，提示找不到operating system，官网有说使用主机CD-ROM或DVD-ROM设备是不行的，放弃了。

安装几种途径和命令行一样，通过镜像步骤如下：
- 首先把RHEL7镜像拷贝到RHEL8中，默认的文件夹：/var/lib/libvirt/images
- 打开`Virtual Machine Manager`：
    - 通过超级用户`virt-manager`执行命令
    - 图形打开应用程序→系统工具→虚拟机管理器来打开`virt-manager`
    - 如果用命令的话，并且使用的shell终端没有图形界面，应在本地执行命令开启，我使用Xmanager，有图形终端
- 点击新建虚拟机图标，或者依次选择：File--New Virtual Machine
- 弹出`New VM`对话框，选择`Local install media(ISO image or CDROM)`，然后下一步
- `Choose ISO`输入框边点击`Browse`，选择`RHEL-7.8_Server.x86_64.iso`，回到`New VM`对话框
- 手动或者自动选择操作系统（我的自动显示None detected，手动也没用RHEL7.8，选了个Generic default)，然后下一步
- 设置内存和CPU，然后下一步
- 设置磁盘大小（我设置6G），然后下一步
- 设置虚拟机名字和网络，网络默认NAT，然后下一步
- 弹出新的图形界面，选择安装或者测试
- 开始安装后等待到选择语言对话框，选择语言后下一步
- 进入自定义设置界面，例如磁盘和软件安装设置，我选择了minmal install
- 点击`begin install`开始安装，中途可以设置设root密码或者用户
- Reboot选项出来后重启系统，安装完成

查看：
```
[root@redhat8 ~]# virsh list
 Id    Name                           State
----------------------------------------------------
 2     RHEL78                         running
```
### virt-install和virt-manager安装选项的比较
参考官方说明：[virt-install和virt-manager安装选项的比较](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-virtual_machine_installation-virt-install-virt-manager-matrix)

## PXE方式自动安装
参考文档：
- [3.3. PREPARING INSTALLATION SOURCES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-making-media-additional-sources#sect-making-media-sources-network)
- [Chapter 4.2 Automatic Installation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-simple-install-kickstart)
- [PART I.AMD64,INTEL 64,AND ARM 64-INSTALLATION AND BOOTING](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/part-installation-intel-amd)
- [CHAPTER 24. PREPARING FOR A NETWORK INSTALLATION](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-installation-server-setup#sect-network-boot-setup-ppc-grub2)

### 环境准备
#### 安装vsftpd
宿主系统上安装vsftpd:
```
[root@redhat8 tmp]# rpm -ivh vsftpd-3.0.3-32.el8.x86_64.rpm
warning: vsftpd-3.0.3-32.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: 
NOKEYVerifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:vsftpd-3.0.3-32.el8              ################################# [100%]
```
如果yum源里面有：
```
yum install -y vsftpd
```
启用vsftpd服务：
```
[root@redhat8 tmp]# systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/sys
temd/system/vsftpd.service.
[root@redhat8 tmp]# systemctl start vsftpd
```
将RHEL7安装文件拷贝到系统中，或者挂载光驱，空间有限我使用光驱：
```
[root@redhat8 /]# mount /dev/cdrom /mnt/cdrom
mount: /mnt/cdrom: WARNING: device write-protected, mounted read-only.
```
将文件复制到FTP目录：
```
[root@redhat8 rhel7]# cp -r /mnt/cdrom/ /var/ftp/
```
或者直接将光盘挂载到ftp服务作为FTP安装源，更加节省空间：
```
[root@redhat8 /]# mkdir -p /var/ftp/pub/rhel7 /var/ftp/pub/pxeboot /var/ftp/pub/ks
[root@redhat8 /]# echo "/dev/sr0 /var/ftp/pub/rhel7 iso9660 loop 0 0" >> /etc/fstab
[root@redhat8 /]# mount -a
[root@redhat8 /]# cd /var/ftp/pub/rhel7
[root@redhat8 rhel7]# ls
addons  extra_files.json  isolinux    Packages                 RPM-GPG-KEY-redhat-release
EFI     GPL               LiveOS      repodata                 TRANS.TBL
EULA    images            media.repo  RPM-GPG-KEY-redhat-beta
```
#### 准备PXE启动目录和文件
准备PXE启动目录和文件:
```
[root@redhat8 tmp]# rpm2cpio /var/ftp/pub/rhel7/Packages/syslinux-4.05-15.el7.x86_64.rpm | cpio -dimv
[root@redhat8 syslinux]# cd /tmp
[root@redhat8 tmp]# cp ./usr/share/syslinux/vesamenu.c32 /var/ftp/pub/pxeboot/
[root@redhat8 tmp]# cp ./usr/share/syslinux/pxelinux.0 /var/ftp/pub/pxeboot/
[root@redhat8 tmp]# cp /var/ftp/pub/rhel7/images/pxeboot/* /var/ftp/pub/pxeboot/
[root@redhat8 tmp]# mkdir /var/ftp/pub/pxeboot/pxelinux.cfg
```
配置文件`/var/lib/tftpboot/pxelinux/pxelinux.cfg/default`，内容：
```
default vesamenu.c32
timeout 600

display boot.msg

label linux
  menu label ^Install RHEL 7.8
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.ks=ftp://192.168.122.1/pub/ks/rhel7_minimal.cfg ip=192.168.122.110:::255.255.255.0:node1:eth0:off console=tty0 console=ttyS0,19200n8
```
配置libvirtd自带的TFTP服务作为PXE启动服务器 ，命令：
```
[root@redhat8 tmp]# virsh net-edit default 
Network default XML configuration edited
```
内容如下：
```xml
<network>
  <name>default</name>
  <uuid>00e869b2-4147-40e3-ad93-38aa7ff5a914</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:be:8e:dc'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>  
    <tftp root='/var/ftp/pub/pxeboot/'/>    
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
      <bootp file='pxelinux.0' server='192.168.122.1'/>
    </dhcp>
  </ip>
</network>
```
启动配置但是报错了：
```
[root@redhat8 tmp]# virsh net-destroy default
Network default destroyed

[root@redhat8 tmp]# virsh net-start default
error: Failed to start network default
error: internal error: Child process (VIR_BRIDGE_NAME=virbr0 /usr/sbin/dnsmasq --conf-file
=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper) unexpected exit status 3: dnsmasq: TFTP directory /var/ftp/pub/pxeboot/ inaccessible: Permission denied
```
提示权限不够，查了下解决方法：
```
[root@redhat8 pxelinux.cfg]# setenforce 0
[root@redhat8 pxelinux.cfg]# virsh net-destroy default
Network default destroyed
[root@redhat8 pxelinux.cfg]# virsh net-start default
Network default started
```
参考文档：[dnsmasq TFTP directory /tftpd inaccessible: Permission denied](http://www.voidcn.com/article/p-rcqogdgu-mw.html)

#### kickstart自动安装文件
修改生成kickstart自动安装文件，从host中拷贝然后修改
```
[root@redhat8 ~]# cp /root/anaconda-ks.cfg /var/ftp/pub/ks/rhel7_minimal.cfg
[root@redhat8 ks]# vi /var/ftp/pub/ks/rhel7_minimal.cfg
```
修改后文件如下：
```
#version=RHEL8
ignoredisk --only-use=nvme0n1
autopart --type=lvm
# Partition clearing information
clearpart --none --initlabel
# Use graphical install
# graphical
repo --name="AppStream" --baseurl=file:///run/install/repo/AppStream
# Use CDROM installation media
# cdrom
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8

# cmdline
ignoredisk --only-use=vda

# Network information
network  --bootproto=static --device=eth0 --ip=192.168.122.110 --netmask=255.255.255.0 --gateway=192.168.122.1.1 --ipv6=auto --activate
network  --hostname=node1
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=vda
clearpart --all --initlabel --drives=vda
part /boot --fstype="xfs" --ondisk=vda --size=1024
part pv.614 --fstype="lvmpv" --ondisk=vda --grow
volgroup node1 --pesize=4096 pv.614
logvol /  --fstype="xfs" --size=4096 --grow --name=root --vgname=node1
# Root password
rootpw --iscrypted $6$Q9tSQSFpjwq0eOZ7$SxPBLdnUyPHGbhLXfGEomqwfH5cRiImcPB6Y07Mu8PfAKjjMLMh
mW2XhbHAEqMtQ./y0n5dGEPa3E.YqY/nVQ.# X Window System configuration information
# xconfig  --startxonboot
# Run the Setup Agent on first boot
firstboot --enable
# System services
services --enabled="chronyd"
# System timezone
timezone America/New_York --isUtc
user --name=huang --password=$6$TwYrj0gr3EMjRAlU$N3aIbs/Dq7cT4hGMavXQjgCY9w/QmhFXlnZDs0vXL
M3YlHdyPBKjqpqyR6G86wu9APHoVvvf31ffPULETrXO7/ --iscrypted --gecos="huang"
%packages
@^minimal
kexec-tools
curl
wget
net-tools
ftp 
telnet
%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

url --url="ftp://192.168.122.1/pub/rhel7 
firewall --disabled
selinux --disabled 
eula --agreed
reboot
```
### 创建虚拟机
创建新的xml文件:
```
[root@redhat8 qemu]# pwd
/etc/libvirt/qemu
[root@redhat8 qemu]# vi rhel7.xml
```
内容如下：
```xml
<domain type='qemu'>
  <uuid></uuid>
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='static'>1</vcpu>
  <name>node1</name>
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.0.0'>hvm</type>
    <boot dev='hd'/>
    <boot dev='network'/>
    <boot dev='cdrom'/>
    <bootmenu enable='yes'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <vmport state='off'/>
  </features>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/images/node1.qcow2'/>
      <target dev='hda' bus='ide'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x7'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <master startport='0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0' multifunction='on'/>  
    </controller>
    <controller type='usb' index='0' model='ich9-uhci2'>
      <master startport='2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x1'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci3'>
      <master startport='4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'/>
    <controller type='ide' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <interface type='network'>
      <source network='default'/>
      <model type='e1000'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <listen type='address'/>
      <image compression='off'/>
    </graphics>
    <sound model='ich6'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='3'/>
    </redirdev>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </memballoon>
  </devices>
</domain>
```
创建磁盘：
```
[root@redhat8 qemu]# qemu-img create -f qcow2 /var/lib/images/node1.qcow2 10G
Formatting '/var/lib/images/node1.qcow2', fmt=qcow2 size=10737418240 cluster_size=65536 la
zy_refcounts=off refcount_bits=16
```
执行定义：
```
[root@redhat8 qemu]# virsh define node1.xml
Domain node1 defined from node1.xml
```
### 启动安装
启动虚拟机，进入控制台观察安装进度：
```
[root@redhat8 qemu]# virsh list
 Id    Name                           State
----------------------------------------------------
 4     node1                          running
```
