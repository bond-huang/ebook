# KVM-虚拟机自动安装
之前有学习安装方式，这里继续学习记录自动安装的方法。
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
