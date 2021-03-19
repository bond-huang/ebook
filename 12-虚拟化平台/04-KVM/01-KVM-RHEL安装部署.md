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
