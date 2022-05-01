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

## 待补充

