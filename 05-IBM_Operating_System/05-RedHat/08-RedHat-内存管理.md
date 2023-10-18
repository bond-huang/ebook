# RedHat-内存管理
## 内存优化
参考链接：
- [https://linux-mm.org/Drop_Caches](https://linux-mm.org/Drop_Caches)

### 内存清理
#### 单次手动清理
清理三种操作说明如下：
- echo 1 > /proc/sys/vm/drop_caches:表示清除pagecache
- echo 2 > /proc/sys/vm/drop caches:表示清除回收slab分配器中的对象 (包括目录项缓存和inode缓存)。slab 分配器是内核中管理内存的一种机制，其中很多缓存数据实现都是用的pagecache
- echo 3 > /proc/sys/ym/drop_caches:表示清除pagecache和slab分配器中的缓存对象

在`/etc/sysctl.conf`文件中添加如下配置: 
```
ym.drop_caches = 1
```
然后执行`sysctl -p`命令，也是一次性操作。
#### 自动清理
##### 定时任务清理
每天自动清理，首先写个脚本`clear_cache.sh`：
```sh
#!/bin/bash
sync
sleep 10
echo 1 > /proc/sys/vm/drop_caches
```
添加权限：
```sh
chmod +x clear_cache.sh
```
编辑当前用户计划任务：
```sh
crontab -e
```
添加每天凌晨运行的计划任务：
```
0 0 * * * /path/to/clear_cache.sh
```
##### 系统自动回收清理
&#8195;&#8195;通过设置`min_free_kbytes`和`vfs_cache_pressure`的值让系统根据此配置对内存使用进行限制，以达到清理或回收内存目的，说明如下：
- `min_free_kbytes`：用于设置内核保留的最小可用内存量，默认66M：67584（以KB为单位）。它确保系统在内存紧张时保留一定数量的可用内存，以避免系统变得过度交换或出现性能问题。可以根据系统内存大小和用途来配置这个参数
- `vfs_cache_pressure`：用于调整页缓存回收策略，它影响内核在内存不足时如何管理页缓存。可以根据系统的需求调整它，以影响内核是倾向于保留页缓存还是更快地回收它们：
    - 缺省值100表示内核将根据pagecache和swapcache，把directory和inode cache保持在一个合理的百分比，适用于大多数情况
    - 降低该值低于100，将导致内核倾向于保留directory和inode cache
    - 增加该值超过100，将导致内核倾向于回收directory和inode cache。默认值为100，，可以根据需要进行微调。
 
`min_free_kbytes`值查看方法：
```
[root@centos82 ~]# cat /proc/sys/vm/min_free_kbytes
45056
[root@centos82 ~]# sysctl -a |grep "min_free_kbytes"
vm.min_free_kbytes = 45056
```
`vfs_cache_pressure`查看方法：
```
[root@centos82 ~]# sysctl -a |grep "vfs_cache_pressure"
vm.vfs_cache_pressure = 100
```
## 待补充