# PowerLinux-常见问题
## 系统启动问题
### 文件系统引导相关
启动系统时候示例报错如下：
```
*** An error occurred during the file system check.
*** Dropping you to a shell; the system will reboot
*** when you leave the shell.
*** Warning -- SELinux is active
*** Disabling security enforcement for system recovery.
*** Run 'setenforce 1' to reenable.
Give root password for maintenance
(or type Control-D to continue): 
```
提示的文件系统进行修复后报错示例：
```
[/sbin/fsck.ext4 (1) -- /] fsck.ext4 -a /dev/mapper/VolGroup-lv_root 
/dev/mapper/VolGroup-lv_root: clean, 69777/6004736 files, 935418/23999488 blocks
[/sbin/fsck.ext4 (1) -- /boot] fsck.ext4 -a /dev/mapper/mpathap2 
/dev/mapper/mpathap2: clean, 23/128016 files, 81772/512000 blocks
[FAILED]
```
维护模式输入密码后重新挂载文件系统：
```
[root@localhost ~]# mount -o remount,rw /
```
&#8195;&#8195;应该是/etc/fstab里面配置错误，修改/etc/fstab，将配置错误的文件系统进行注释。然后重启系统，可以正常引导进入系统了。

## 待补充