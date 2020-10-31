# RedHat-文件和文字处理
使用RedHat8系统学习文件处理时候记录的学习笔记。
## 文件链接命令ln
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-ln.html](https://www.runoob.com/linux/linux-comm-ln.html)
### 软链接
使用ln命令创建软链接,并对源进行修改及删除,观察影响：
```
[root@redhat8 linuxone]# touch test1.log
[root@redhat8 linuxone]# vi test1.log
[root@redhat8 linuxone]# cat test1.log
link files test file
[root@redhat8 linuxone]# ll
total 4
-rw-r--r--. 1 root root 21 Oct 31 01:07 test1.log
[root@redhat8 linuxone]# ln -s test1.log link1
[root@redhat8 linuxone]# ll
total 4
lrwxrwxrwx. 1 root root  9 Oct 31 01:09 link1 -> test1.log
-rw-r--r--. 1 root root 21 Oct 31 01:07 test1.log
[root@redhat8 linuxone]# cat link1
link files test file
[root@redhat8 linuxone]# vi test1.log
[root@redhat8 linuxone]# cat test1.log
link files test file
try to change the source file
[root@redhat8 linuxone]# cat link1
link files test file
try to change the source file
[root@redhat8 linuxone]# rm test1.log
rm: remove regular file 'test1.log'? y
[root@redhat8 linuxone]# cat link1
cat: link1: No such file or directory
[root@redhat8 linuxone]# ll
total 0
lrwxrwxrwx. 1 root root 9 Oct 31 01:09 link1 -> test1.log
```
### 硬链接
使用ln命令创建硬链接,并对源进行修改及删除,观察影响：：
```
[root@redhat8 linuxone]# ll
total 4
lrwxrwxrwx. 1 root root  9 Oct 31 01:09 link1 -> test1.log
-rw-r--r--. 1 root root 21 Oct 31 01:17 test1.log
[root@redhat8 linuxone]# cat test1.log
link files test file
[root@redhat8 linuxone]# ln test1.log ln1
[root@redhat8 linuxone]# ll
total 8
lrwxrwxrwx. 1 root root  9 Oct 31 01:09 link1 -> test1.log
-rw-r--r--. 2 root root 21 Oct 31 01:17 ln1
-rw-r--r--. 2 root root 21 Oct 31 01:17 test1.log
[root@redhat8 linuxone]# cat ln1
link files test file
[root@redhat8 linuxone]# vi test1.log
[root@redhat8 linuxone]# cat test1.log
link files test file
change the source file
[root@redhat8 linuxone]# cat ln1
link files test file
change the source file
[root@redhat8 linuxone]# rm test1.log
rm: remove regular file 'test1.log'? y
[root@redhat8 linuxone]# ll
total 4
lrwxrwxrwx. 1 root root  9 Oct 31 01:09 link1 -> test1.log
-rw-r--r--. 1 root root 44 Oct 31 01:20 ln1
[root@redhat8 linuxone]# cat ln1
link files test file
change the source file
```
## 文件查找命令find
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-find.html](https://www.runoob.com/linux/linux-comm-find.html)

使用find命令在指定路径按文件名查找:
```
[root@redhat8 linuxone]# find /var/log -name boot.log
/var/log/boot.log
```
列出当前目录及其子目录下所有扩展名为`log`的文件:
```
[root@redhat8 log]# find . -name "*.log"
./audit/audit.log
./rhsm/rhsmcertd.log
...
```
列出当前目录及其子目录下所有一般文件:
```
[root@redhat8 linuxone]# find . -type f
./ln1
./test1.log
```
列出当前目录及其子目录下所有最近2分钟更新过的文件：
```
[root@redhat8 linuxone]# vi test1.log
[root@redhat8 linuxone]# find . -cmin -2
.
./test1.log
```
查找`/var/log`目录中更改时间在30天以前的普通文件，并在删除前进行询问：
```
[root@redhat8 linuxone]# find /var/log -type f -mtime +30 -ok rm {} \;
< rm ... /var/log/README > ? n
< rm ... /var/log/cups/access_log-20200726 > ? n
< rm ... /var/log/cups/access_log > ? n
...
```
查找当前目录文件属性具有读、写权限，并且文件所属组的哦那个和和其它用户具有读权限的文件：
```
[root@redhat8 linuxone]# find . -type f -perm 644 -exec ls -l {} \;
-rw-r--r--. 1 root root 44 Oct 31 01:20 ./ln1
-rw-r--r--. 1 root root 46 Oct 31 01:40 ./test1.log
```
查找系统中所有文件长度为0的不同文件，并列出它们的完整路径：
```
[root@redhat8 linuxone]# find / -type f -size 0 -exec ls -l {} \;
-r--r--r--. 1 root root 0 Oct 31 01:46 /proc/fb
-r--r--r--. 1 root root 0 Oct 31 01:46 /proc/fs/xfs/xqm
-r--r--r--. 1 root root 0 Oct 31 01:46 /proc/fs/xfs/xqmstat
···
```
## 文件查看命令
### less命令
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-less.html](https://www.runoob.com/linux/linux-comm-less.html)

使用find命令查找后缀为`.sh`的文件，使用less命令查看文件内容
```
less `find . -name "*.sh"`
```
### tail命令
&#8195;&#8195;tail命令用于查看文件的内容，参数`-f`常用于查阅正在改变的日志文件。详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-tail.html](https://www.runoob.com/linux/linux-comm-tail.html)

使用tail查看日志文件`/var/log/messages`，并使用`-n`参数（显示文件的尾部n行内容 ）控制输出的行数:
```
[root@redhat8 linuxone]# tail -n 4  /var/log/messages
Oct 31 02:59:11 redhat8 dbus-daemon[899]: [system] Successfully activated service 'org.fre
edesktop.nm_dispatcher'Oct 31 02:59:11 redhat8 systemd[1]: Started Network Manager Script Dispatcher Service.
Oct 31 02:59:11 redhat8 nm-dispatcher[4287]: req:1 'dhcp4-change' [ens160]: new request (3
 scripts)Oct 31 02:59:11 redhat8 nm-dispatcher[4287]: req:1 'dhcp4-change' [ens160]: start running 
ordered scripts...
```
使用tail查看日志文件`/var/log/messages`，显示从2685行至文件末尾:
```
[root@redhat8 linuxone]# tail -n +2685 /var/log/messages
Oct 31 02:59:11 redhat8 systemd[1]: Starting Network Manager Script Dispatcher Service...
Oct 31 02:59:11 redhat8 dbus-daemon[899]: [system] Successfully activated service 'org.fre
edesktop.nm_dispatcher'Oct 31 02:59:11 redhat8 systemd[1]: Started Network Manager Script Dispatcher Service.
Oct 31 02:59:11 redhat8 nm-dispatcher[4287]: req:1 'dhcp4-change' [ens160]: new request (3
 scripts)Oct 31 02:59:11 redhat8 nm-dispatcher[4287]: req:1 'dhcp4-change' [ens160]: start running 
ordered scripts...
```
### head命令
&#8195;&#8195;head命令用于查看文件的开头部分的内容，常用的参数`-n`用户显示查看行数。详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-head.html](https://www.runoob.com/linux/linux-comm-head.html)

使用haead查看日志文件`/var/log/messages`，控制输出的行数:
```
[root@redhat8 linuxone]# head -n 5 /var/log/messages
Oct 28 04:07:02 redhat8 rsyslogd[1438]: [origin software="rsyslogd" swVersion="8.37.0-9.el
8" x-pid="1438" x-info="http://www.rsyslog.com"] rsyslogd was HUPedOct 28 04:20:00 redhat8 NetworkManager[1045]: <info>  [1603873200.6151] dhcp4 (ens160):   
address 192.168.18.131Oct 28 04:20:00 redhat8 NetworkManager[1045]: <info>  [1603873200.6156] dhcp4 (ens160):   
plen 24Oct 28 04:20:00 redhat8 NetworkManager[1045]: <info>  [1603873200.6157] dhcp4 (ens160):   
expires in 1800 secondsOct 28 04:20:00 redhat8 NetworkManager[1045]: <info>  [1603873200.6157] dhcp4 (ens160):   
nameserver '192.168.18.2'
```
## 文件编辑命令sed
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-sed.html](https://www.runoob.com/linux/linux-comm-sed.html)

之前学习sed编辑器做的笔记：[Shell笔记-sed和gawk基础](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/01-Shell%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/07-Shell%E7%AC%94%E8%AE%B0-sed%E5%92%8Cgawk%E5%9F%BA%E7%A1%80.html)

sed编辑器进阶学习笔记：[Shell笔记-sed编辑器](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/01-Shell%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/09-Shell%E7%AC%94%E8%AE%B0-sed%E7%BC%96%E8%BE%91%E5%99%A8.html)

使用sed命令屏蔽`/etc/fstab`文件中错误的定义行,例如如下两行定义：
```
[root@redhat8 linuxone]# tail -n 5 /etc/fstab
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=0a510af6-8abf-4e64-b559-902804c93568 /boot                   xfs     defaults        
0 0/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
/tmp/not-existing	/mnt/not-existing	iso9660 default		0 0
/tmp/test		/mnt/test		iso9660 default		0 0
[root@redhat8 linuxone]# mkdir /mnt/test
[root@redhat8 linuxone]# mount -a
mount: /mnt/not-existing: mount point does not exist.
mount: /mnt/test: special device /tmp/test does not exist.
```
如果知道`/mnt/not-existing`是无效的，在输出中屏蔽示例（如需修改原文件加上`-i`参数）：
```
[root@redhat8 linuxone]# sed '/mnt\/not-existing/ s/^/# /' /etc/fstab
...
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=0a510af6-8abf-4e64-b559-902804c93568 /boot                   xfs     defaults        
0 0/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
#/tmp/not-existing	/mnt/not-existing	iso9660 default		0 0
/tmp/test		/mnt/test		iso9660 default		0 0

```
如果不知道哪些是否有效，可以`mount -a`查看无效项目，然后使用sed进行屏蔽：
```sh
#!/bin/bash
tmp=$(date +"%y%m%d")
mount -a 2> mount$tmp.log
invalid_list=`cat mount$tmp.log | sed -n '/does not exist/p'|gawk 'BEGIN{FS=": "}{print $2}'`
echo $invalid_list
for i in $invalid_list
do
    i=`echo $i |sed 's/\//\\\&/g'`
    sed -i '/'$i'/ s/^/# /' /etc/fstab
done
```
运行后查看`/etc/fstab`文件如下：
```
[root@redhat8 linuxone]# tail -n 5 /etc/fstab
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=0a510af6-8abf-4e64-b559-902804c93568 /boot                   xfs     defaults        
0 0/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
# /tmp/not-existing	/mnt/not-existing	iso9660 default		0 0
# /tmp/test		/mnt/test		iso9660 default		0 0
```
## grep命令
&#8195;&#8195;grep比较常用，用于查找文件里符合条件的字符串,常用`-i`参数（忽略字符大小写的差别）。详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-grep.html](https://www.runoob.com/linux/linux-comm-grep.html)

使用grep命令列出`/var/log/messages`文件中包含eth的行:
```
[root@redhat8 linuxone]# grep eth /var/log/messages
Oct 31 00:58:58 redhat8 kernel: vmxnet3 0000:03:00.0 eth0: NIC Link is Up 10000 Mbps
Oct 31 00:58:58 redhat8 kernel: vmxnet3 0000:03:00.0 ens160: renamed from eth0
...
[root@redhat8 linuxone]# cat /var/log/messages |grep eth
Oct 31 00:58:58 redhat8 kernel: vmxnet3 0000:03:00.0 eth0: NIC Link is Up 10000 Mbps
Oct 31 00:58:58 redhat8 kernel: vmxnet3 0000:03:00.0 ens160: renamed from eth0
...
```
