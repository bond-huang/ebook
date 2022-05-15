# RedHat-常见问题
## 应急模式
### 文件系统挂载异常
进入系统后，提示如下：
```
You are in emergency mode.After logging in,type "journalctl -xb" to view system logs,"systemctl reboot" to reboot,"systemctl default" or "exit" to boot into default mode.
```
&#8195;&#8195;一般是挂载磁盘或者文件系统出现问题，根据提示输入`journalctl -xb`查看logs，查找`fsck failed`,如果能找到可以找到对应磁盘进行`fsck`修复，然后重启。如果没有，可能是文件系统挂载出现问题，此次我遇到的是文件系统挂载问题，在`/etc/fstab`中异常的条目：
```
/dev/sr0 /var/ftp/pub/rhel7 iso9660 loop 0 0
```
注释掉然后运行命令`systemctl reboot`重启即可。

## YUM源问题
### YUM
配置阿里YUM源成功：
```
[root@redhat8 yum.repos.d]# wget https://mirrors.aliyun.com/repo/Centos-8.repo
--2022-05-14 11:25:49--  https://mirrors.aliyun.com/repo/Centos-8.repo
Resolving mirrors.aliyun.com (mirrors.aliyun.com)... 117.21.230.243, 117.21.230.244, 117.21.230.239, ...
Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|117.21.230.243|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2600 (2.5K) [application/octet-stream]
Saving to: ‘Centos-8.repo’
Centos-8.repo                  100%[=================================================>]   2.54K  --.-KB/s    in 0s      
2022-05-14 11:25:49 (29.3 MB/s) - ‘Centos-8.repo’ saved [2600/2600]
```
查看有报错，并且安装软件不行：
```
[root@redhat8 yum.repos.d]# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
CentOS-8 - AppStream - mirrors.aliyun.com                                                0.0  B/s |   0  B     00:24    
CentOS-8 - Base - mirrors.aliyun.com                                                     0.0  B/s |   0  B     00:24    
Failed to synchronize cache for repo 'AppStream', ignoring this repo.
Failed to synchronize cache for repo 'base', ignoring this repo.
Last metadata expiration check: 0:12:34 ago on Sat 14 May 2022 12:32:44 PM EDT.
repo id                                   repo name                                                                status
extras                                    CentOS-8 - Extras - mirrors.aliyuncom                                   39
```
修改掉阿里YUM源文件名称：
```
[root@redhat8 yum.repos.d]# mv Centos-8.repo Centos-8.repo.bk
```
获取华为的YUM源安装文件：
```
[root@redhat8 yum.repos.d]# wget  https://repo.huaweicloud.com/epel/epel-release-latest-7.noarch.rpm
--2022-05-14 12:53:26--  https://repo.huaweicloud.com/epel/epel-release-latest-7.noarch.rpm
Resolving repo.huaweicloud.com (repo.huaweicloud.com)... 182.106.149.164, 182.106.149.160, 182.106.149.161, ...
Connecting to repo.huaweicloud.com (repo.huaweicloud.com)|182.106.149.164|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15608 (15K) [application/x-redhat-package-manager]
Saving to: ‘epel-release-latest-7.noarch.rpm’
epel-release-latest-7.noarch.r 100%[=================================================>]  15.24K  --.-KB/s    in 0s      
2022-05-14 12:53:26 (122 MB/s) - ‘epel-release-latest-7.noarch.rpm’ saved [15608/15608]
[root@redhat8 yum.repos.d]# ls
Centos-8.repo.bk  epel-release-latest-7.noarch.rpm  redhat8.repo.bk  redhat.repo
```
安装华为YUM源：
```
[root@redhat8 yum.repos.d]# rpm  -ivh  --nodeps  epel-release-latest-7.noarch.rpm
warning: epel-release-latest-7.noarch.rpm: Header V4 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:epel-release-7-14                ################################# [100%]
[root@redhat8 yum.repos.d]# ls
Centos-8.repo.bk  epel-release-latest-7.noarch.rpm  epel.repo  epel-testing.repo  redhat8.repo.bk  redhat.repo
```
将`epel.repo`文件中的`#baseurl`替换成`baseurl`：
```
[root@redhat8 yum.repos.d]# sed -i "s/#baseurl/baseurl/g" /etc/yum.repos.d/epel.repo
```
刷新yum源缓存：
```
[root@redhat8 yum.repos.d]# yum clean all
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
5 files removed
[root@redhat8 yum.repos.d]# yum makecache
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Extra Packages for Enterprise Linux 7 - x86_64                                           944 kB/s |  17 MB     00:18    
Last metadata expiration check: 0:00:05 ago on Sat 14 May 2022 12:59:33 PM EDT.
Metadata cache created.
```
再次查看：
```
[root@redhat8 yum.repos.d]# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:37 ago on Sat 14 May 2022 12:59:33 PM EDT.
repo id                               repo name                                                                    status
*epel                                 Extra Packages for Enterprise Linux 7 - x86_64                               13,755
```
华为官网参考链接：[配置yum源](https://support.huaweicloud.com/prtg-kunpenghpcs/openmind_kunpengcdo_02_0006.html)
## 磁盘问题
### 文件系统扩容问题
#### Bad magic number
对`/`文件系统进行扩容：
```
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
tmpfs                  904M     0  904M   0% /dev/shm
tmpfs                  904M  9.4M  894M   2% /run
tmpfs                  904M     0  904M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  7.9G  9.2G  47% /
/dev/nvme0n1p1        1014M  169M  846M  17% /boot
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
tmpfs                  181M   16K  181M   1% /run/user/42
tmpfs                  181M  4.0K  181M   1% /run/user/1000
```
报错如下：
```
[root@redhat8 ~]# resize2fs /dev/mapper/rhel-root
resize2fs 1.44.3 (10-July-2018)
resize2fs: Bad magic number in super-block while trying to open /dev/mapper/rhel-root
Couldn't find valid filesystem superblock.
```
检查发现是xfs文件系统：
```
[root@redhat8 ~]# mount |grep root
/dev/mapper/rhel-root on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```
使用xfs文件系统相关命令，依旧报错：
```
[root@redhat8 ~]# xfs_growfs /dev/mapper/rhel-root
xfs_growfs: /dev/mapper/rhel-root is not a mounted XFS filesystem
```
命令`xfs_growfs`版本新旧使用方法不一样问题，可以查看命令描述。解决示例：
```
[root@redhat8 ~]# xfs_growfs /
meta-data=/dev/mapper/rhel-root  isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 9436160
```
查看确认：
```
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
tmpfs                  904M     0  904M   0% /dev/shm
tmpfs                  904M  9.4M  894M   2% /run
tmpfs                  904M     0  904M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   36G  8.0G   29G  23% /
/dev/nvme0n1p1        1014M  169M  846M  17% /boot
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
tmpfs                  181M   16K  181M   1% /run/user/42
tmpfs                  181M  4.0K  181M   1% /run/user/1000
```
问题解决参考链接：
- [resize2fs: Bad magic number in super-block while trying to open /dev/centos/root Couldn't find valid](https://blog.csdn.net/yaofengyaofeng/article/details/82353282)
- [在线扩容CentOS 8系统盘报“xfs_growfs:is not a mounted XFS filesystem”错误](https://help.aliyun.com/document_detail/155936.html#:~:text=xfs_growfs%3A%2Fdev%2Fvda1%20is%20not%20a%20mounted,XFS%20filesystem%20%E9%97%AE%E9%A2%98%E5%8E%9F%E5%9B%A0%20%E6%96%B0%E6%97%A7%E7%89%88%E6%9C%AC%E7%9A%84%20xfs_growfs%20%E5%91%BD%E4%BB%A4%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E3%80%82)
- [Bug 1885875 - xfs_growfs: /dev/mapper/rhel-root is not a mounted XFS filesystem ](https://bugzilla.redhat.com/show_bug.cgi?id=1885875)

## 待补充