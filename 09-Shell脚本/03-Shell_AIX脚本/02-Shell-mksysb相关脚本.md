# Shell-mksysb相关脚本
mksysb过程中遇到一些问题写的一些脚本。
### 修改image.data
#### 修改目的说明
&#8195;&#8195;在AIX系统中，对rootvg进行备份使用mksysb时候，默认会在根目录下创建image.data文件，mksysb会根据此文件生成对应的镜像。如果原分区rootvg是两块磁盘做镜像，现在要恢复到新分区上只有一块硬盘，并且只需要一块，多了浪费，如果直接做mksyb，恢复分区时候会报错，提示空间不够，有几种解决方法：
- 解除原分区的镜像，做mksyb后再重新做镜像（对一些高要求系统来说存在风险）
- 搞一块新的临时硬盘到新分区（也许配置会比较麻烦，后续可能会删掉概率也大）
- 修改image.data文件，在做mksysb的时候不创建新的image.data，使用修改的image.data

&#8195;&#8195;最简单并且风险比较低的还是修改image.data，仔细认真点改没问题，如果需要这样做的系统很多，就比较耗费时间了，还容易出错，通过脚本修改就比较方便。

#### 脚本使用说明
- 必须使用root用户或root权限的用户执行脚本
- 脚本中自动跳转到了根目录，在任何目录有权限运行即可
- rootvg中最好不要有自定义未mount的文件系统，不会备份
- 不能有跨盘的lv（即一个lv一份数据分布在不同盘上）,如果有要提前整合到一块盘中
- 脚本自动判断了拥有rootvg全部lv的磁盘，根据识别到盘符去修改image.data
- 不管rootvg有几份镜像，都修改成1了，也就是目标端一块盘即可（多块也没问题）
- 如果原rootvg有很多磁盘，lv分布很多磁盘，并且难以整合到一个中，那么此脚本没用（一些将数据放置在rootvg里面的系统可能会有这种情况，通常这种是不推荐的）

#### 脚本代码
代码如下所示：
```sh
#!/bin/ksh
# Change the image.data!
echo "Great a new image.data,please wait..."
mkszfile
sleep 15
cd /
# Change the LV_SOURCE_DISK_LIST!
echo "Change the LV_SOURCE_DISK_LIST..."
hdisk_list=$(lspv | awk '{if($3=="rootvg"){print $1}}')
for disk in $hdisk_list
do
    lv_count1=`lsvg -l rootvg | wc -l`
    lv_count2=`lspv -l $disk | wc -l`
    if [ $lv_count2 -eq $lv_count1 ]
    then
        hdisk=$disk
    fi
done
sed '
s/LV_SOURCE_DISK_LIST=.*/LV_SOURCE_DISK_LIST= '$hdisk'/
' image.data > image.data.img
mv image.data.img image.data
# Change the COPIES!
echo "Change the COPIES..."
sed 's/COPIES= [0-9]*/COPIES= 1/' image.data > image.data.img
mv image.data.img image.data
# Change the PPs!
echo "Change the PPs..."
sequence=`sed -n '/LPs=/p' image.data|sed -n '/LPs=/='`
LPs_vaule=`cat image.data|awk '/LPs=/{print $2}'`
PP_rows=$(sed -n '/PP=/=' image.data)
for i in $sequence
do
    PP_row=`echo $PP_rows | awk '{print $'$i'}'`
    PP=`echo $LPs_vaule | awk '{print $'$i'}'`
    sed ''$PP_row'{
    s/PP= [0-9]*/PP= '$PP'/
    }' image.data > image.data.img
    mv image.data.img image.data
done
echo "Changes are complete!"
echo "Please use command to do mksysb without '-i' parameter!"
echo "Or use 'smit mksysb' set 'Generate new /image.data file?' to 'No'!"
```
#### 运行示例
在AIX7100-04-03-1642中使用ksh和bash下都尝试运行：
```
# sh test2.sh
Great a new image.data,please wait...
Change the LV_SOURCE_DISK_LIST...
Change the COPIES...
Change the PPs...
Changes are complete!
Please use command to do mksysb without '-i' parameter!
Or use 'smit mksysb' set 'Generate new /image.data file?' to 'No'!
bash-5.0# sh test4.sh
Great a new image.data,please wait...
Change the LV_SOURCE_DISK_LIST...
Change the COPIES...
Change the PPs...
Changes are complete!
Please use command to do mksysb without '-i' parameter!
Or use 'smit mksysb' set 'Generate new /image.data file?' to 'No'!
```
原始image.data示例（以hd3为例）：
```
lv_data:
        VOLUME_GROUP= rootvg
        LV_SOURCE_DISK_LIST= hdisk0 hdisk2 
        LV_IDENTIFIER= 00cb4d6e00004c0000000168788002ed.7
        LOGICAL_VOLUME= hd3
        VG_STAT= active/complete
        TYPE= jfs2
        MAX_LPS= 512
        COPIES= 2
        LPs= 24
        STALE_PPs= 0
        INTER_POLICY= minimum
        INTRA_POLICY= center
        MOUNT_POINT= /tmp
        MIRROR_WRITE_CONSISTENCY= on/ACTIVE
        LV_SEPARATE_PV= yes
        PERMISSION= read/write
        LV_STATE= opened/syncd
        WRITE_VERIFY= off
        PP_SIZE= 128
        SCHED_POLICY= parallel
        PP= 48
        BB_POLICY= relocatable
[...]
```
修改后image.data如下(以hd3为例）：
```
lv_data:
        VOLUME_GROUP= rootvg
        LV_SOURCE_DISK_LIST= hdisk2
        LV_IDENTIFIER= 00cb4d6e00004c0000000168788002ed.7
        LOGICAL_VOLUME= hd3
        VG_STAT= active/complete
        TYPE= jfs2
        MAX_LPS= 512
        COPIES= 1
        LPs= 24
        STALE_PPs= 0
        INTER_POLICY= minimum
        INTRA_POLICY= center
        MOUNT_POINT= /tmp
        MIRROR_WRITE_CONSISTENCY= on/ACTIVE
        LV_SEPARATE_PV= yes
        PERMISSION= read/write
        LV_STATE= opened/syncd
        WRITE_VERIFY= off
        PP_SIZE= 128
        SCHED_POLICY= parallel
        PP= 24
        BB_POLICY= relocatable
[...]
```
### 待补充
