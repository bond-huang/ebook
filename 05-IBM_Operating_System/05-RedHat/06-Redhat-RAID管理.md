# Redhat-RAID管理
## mdadm管理raid
参考链接：
- [使用 mdadm 管理 RAID 阵列](https://zhuanlan.zhihu.com/p/63990027)
- [A guide to mdadm](https://raid.wiki.kernel.org/index.php/A_guide_to_mdadm)
- [RHEL Chapter 21. Managing RAID](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_storage_devices/managing-raid_managing-storage-devices)
- [How To Configure RAID Arrays on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-manage-raid-arrays-with-mdadm-on-ubuntu-22-04)
- [MDADM - Adding a new hard disk to an existing RAID1](https://unix.stackexchange.com/questions/665389/mdadm-adding-a-new-hard-disk-to-an-existing-raid1)
- [How to force mdadm to stop RAID5 array?](https://superuser.com/questions/471327/how-to-force-mdadm-to-stop-raid5-array)
- [构建软RAID5磁盘阵列&RAID设备恢复](https://blog.51cto.com/zencode/1791671)
- [Linear mode to RAID 1 migration](https://serverfault.com/questions/321080/linear-mode-to-raid-1-migration)
- [5.4.16.2. Converting a Linear Device to a RAID Device](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/linear-to-raid)

### linear raid更改为raid1
提问：Linux系统中如何使用mdadm将linear raid更改为raid1，在不迁移数据的情况下。       
ChatGPT回答（未经过验证）：在Linux系统中，可以使用mdadm工具将线性RAID更改为RAID1，而不会导致数据丢失。下面是更改步骤:
- 确保RAID阵列处于活动状态并没有故障，运行以下命令检查RAID阵列的状态
    ```
    $ sudo mdadm --detail /dev/md0
    ```
    其中`/dev/md0`是要更改的RAID阵列的名称
- 停止RAID阵列，运行以下命令:
    ```
    $ sudo mdadm --stop /dev/md0
    ```
- 通过将两个硬盘分区添加到RAID阵列中，创建一个RAID1阵列，例如:
    ```
    $ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdcl
    ```
    其中`/dev/sdb1`和`/dev/sdc1`是要添加到RAID1阵列中的硬盘分区
- 检查新RAID1阵列的状态，运行以下命令:
    ```
    $ sudo mdadm --detail /dev/md0
    ```
    确保RAID1阵列状态正常
- 更新`mdadm.conf`文件，以便在启动时自动加载RAID1阵列。运行以下命令：
    ```
    $ sudo mdadm --examine --scan >> /etc/mdadm/mdadm.conf
    ```
- 将新RAID1阵列标记为启动设备，运行以下命令：
    ```
    $ sudo grub-install /dev/md0
    ```
    这将更新GRUB启动器，以便引导到新的RAID1阵列
- 重新启动系统，确保新的RAID1阵列已经成功启动

&#8195;&#8195;注意:在进行这些更改之前，请务必备份所有重要数据。虽然此过程不会导致数据丢失，但由于意外情况发生的风险仍然存在。

## 待补充