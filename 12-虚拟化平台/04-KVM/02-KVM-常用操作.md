# KVM-常用操作
## 磁盘管理
### 创建磁盘
在/var/lib/images/下创建一个名为mycentos.img的10GB大小，raw格式，空白虚拟机磁盘镜像文件：
```
qemu-img create -f raw /var/lib/images/mycentos.img 10G
```
使用示例：
```
[root@redhat8 ~]# qemu-img create -f raw /var/lib/images/mycentos.img 0.1G
Formatting '/var/lib/images/mycentos.img', fmt=raw size=107374182
[root@redhat8 ~]# ls -l /var/lib/images
total 0
-rw-r--r--. 1 root root 107374592 Apr  8 04:23 mycentos.img
```
## 克隆虚拟机
### 准备工作
&#8195;&#8195;为了使创建的克隆正常工作，通常须在克隆之前删除要克隆的虚拟机所独有的信息和配置：
- 虚拟机平台配置信息，例如网络接口卡（NIC）的数量及其MAC地址等
- 虚拟机系统配置，例如SSH密钥等
- 应用程序级别信息和配置，例如激活码和注册信息等

#### 删除网络配置
删除所有udev规则（如步删除则第一个NIC的名称可能是eth1而不是eth0）：
```
rm -f /etc/udev/rules.d/70-persistent-net.rules
```
通过编辑`/etc/sysconfig/network-scripts/ifcfg-eth[x]`从ifcfg脚本中删除唯一的网络详细信息。

删除HWADDR和静态线（如果HWADDR与新guest的MAC地址不匹配，则将忽略ifcfg）：
```
DEVICE = eth [x]
BOOTPROTO =无
ONBOOT =是
# NETWORK = 10.0.1.0 <-删除
# NETMASK = 255.255.255.0 <-删除
# IPADDR = 10.0.1.20 <-删除
# HWADDR = xx：xx：xx：xx：xx：xx <-删除
# USERCTL =否<-删除
# 删除任何其他*唯一*或非期望的设置，例如UUID。
```
确保保留不包含HWADDR或任何唯一信息的DHCP配置:
```
DEVICE = eth [x]
BOOTPROTO = DHCP
ONBOOT =是
```
确保文件包含以下几行：
```
DEVICE = eth [x]
ONBOOT =是
```
如果存在以下文件，请确保它们包含相同的内容：
```
/etc/sysconfig/networking/devices/ifcfg-eth[x]
/etc/sysconfig/networking/profiles/default/ifcfg-eth[x]
```
如果虚拟机使用了NetworkManager或任何特殊设置，确保从ifcfg脚本中删除了所有其他唯一信息。

#### 删除注册详细信息
对于红帽网络（RHN）注册的虚拟机，请使用以下命令：
```
# rm /etc/sysconfig/rhn/systemid
```
对于Red Hat Subscription Manager（RHSM）注册的虚拟机：
如果将不使用原始虚拟机，请使用以下命令：
```
# subscription-manager unsubscribe --all
# subscription-manager unregister
# subscription-manager clean
```
如果将使用原始虚拟机，则仅运行以下命令：
```
# subscription-manager clean
```
要在克隆后在虚拟机上重新激活RHSM注册。获取您的客户身份代码：
```
# subscription-manager identity
subscription-manager identity: 71rd64fx-6216-4409-bf3a-e4b7c7bd8ac9
```
使用获取的ID代码注册虚拟机：
```
# subscription-manager register --consumerid=71rd64fx-6216-4409-bf3a-e4b7c7bd8ac9
```
#### sshd公钥/私钥
使用以下命令删除所有sshd公钥/私钥对：
```
# rm -rf /etc/ssh/ssh_host_*
```
#### 配置虚拟机在下次启动时运行配置向导
对于RHEL6和更低版本，使用以下命令在名为`.unconfigured`的根文件系统上创建一个空文件：
```
# touch /.unconfigured
```
对于RHEL7，通过运行以下命令来启用首次引导和初始设置向导：
```
# sed -ie 's/RUN_FIRSTBOOT=NO/RUN_FIRSTBOOT=YES/' /etc/sysconfig/firstboot
# systemctl enable firstboot-graphical
# systemctl enable initial-setup-graphical
```
说明：在首次启动克隆虚拟机时，建议更改主机名。

关于克隆虚拟机红帽官方文档：[CHAPTER 4. CLONING VIRTUAL MACHINES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/cloning_virtual_machines#Preparing_a_VM_for_cloning)

## 管理快照
快速创建并查看快照：
```
[root@redhat8 ~]# virsh snapshot-create-as debian debian-snap
Domain snapshot debian-snap created
[root@redhat8 ~]# virsh snapshot-list debian
 Name                 Creation Time             State
------------------------------------------------------------
 debian-snap          2021-04-08 05:22:33 -0400 running
```
删除快照：
```
[root@redhat8 ~]# virsh snapshot-delete debian debian-snap
Domain snapshot debian-snap deleted
```
更多描述参考RHEL官方文档：[20.39 MANAGING SNAPSHOTS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-managing_guest_virtual_machines_with_virsh-managing_snapshots#sect-Managing_snapshots-Creating_Snapshots)

## 虚拟机备份
系统备份方式有克隆，快照，导出等等。
### 快速备份
完整快速备份步骤：
- 备份虚拟机对应的xml文件
- 备份虚拟机的磁盘镜像文件
- 备份虚拟机对应的网络定义xml文件

### 备份示例
&#8195;&#8195;示例快速备份方法，并且新旧两个虚拟机同时运行，修改了name，uuid删除 disk image文件路径进行了修改 5网卡的mac删除了：
```
[root@redhat8 qemu]# cp debian.xml debian1.xml
[root@redhat8 qemu]# ls
debian1.xml  debian.xml  networks  RHEL78.xml
[root@redhat8 /]# cd images
[root@redhat8 images]# ls
debian_wheezy_amd64_standard.qcow2
[root@redhat8 images]# cp debian_wheezy_amd64_standard.qcow2 debian1_wheezy_amd64_standard
.qcow2
[root@redhat8 qemu]# virsh define debian1.xml
Domain debian1 defined from debian1.xml
[root@redhat8 qemu]# virsh define debian1.xml
Domain debian1 defined from debian1.xml
```
运行虚拟机：
```
[root@redhat8 qemu]# virsh start debian1
Domain debian1 started
[root@redhat8 qemu]# virsh list
 Id    Name                           State
----------------------------------------------------
 3     RHEL78                         running
 4     debian                         running
 5     debian1                        running
root@debian-amd64:~# blkid
/dev/sda5: UUID="166ef3ed-f1f6-4be6-8368-d07589692992" TYPE="swap" 
/dev/sda1: UUID="8bf5feb2-6f3a-4842-a242-70aad1afeb98" TYPE="ext4" 
```
&#8195;&#8195;新的虚拟机MAC地址和ip变了，但是hostname没变，如果是生产环境，同时运行可能会产生很多冲突，不推荐，只是作为备份演示。

## 待补充
