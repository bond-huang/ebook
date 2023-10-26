# RedHat-系统性能
官方参考链接：
- [RHEL 7 Performance Tuning Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/index)

## 性能查看命令
### sar命令
#### sar命令参数
主要参数说明如下
- `-A`：相当于`-bBdFHqSuvwWy -I SUM -I ALL -m ALL -n ALL -r ALL -u ALL -P ALL`
- `-B`：Paging统计信息
- `-b`：I/O和传输速率统计信息
- `-d`：块设备统计信息
- `-F [MOUNT]`：文件系统统计信息
- `-f [ filename ]`：从文件名中提取记录（由-o文件名标志创建）
- `-H`：Hugepages利用率统计信息
- `-I { <int_list> | SUM | ALL }`：中断统计信息
- `-i interval`：选择以秒为单位的数据记录，尽可能接近间隔参数指定的数字。
- `-m { <keyword> [,...] | ALL }`：电源管理统计信息，关键字有：
    - CPU：CPU瞬时时钟频率
    - FAN：风扇转速
    - FREQ：CPU平均时钟频率
    - IN：电压输入
    - TEMP：设备温度
    - USB：插入系统的USB设备
- `-n { <keyword> [,...] | ALL }`：网络统计数据，关键字有：
    - DEV：网络接口
    - EDEV：网络接口(errors)
    - NFS：NFS客户端
    - NFSD：NFS服务器
    - SOCK：套接字(v4)
    - IP：IP流量(v4)
    - EIP：IP流量(v4) (errors)
    - ICMP：ICMP流量(v4)
    - EICMP：ICMP流量(v4) (errors)
    - TCP：TCP流量(v4)
    - ETCP：TCP流量(v4) (errors)
    - UDP：UDP流量(v4)
    - SOCK6：套接字(v6)
    - IP6：IP流量(v6)
    - EIP6：IP流量(v6) (errors)
    - ICMP6：ICMP流量(v6)
    - EICMP6：ICMP流量(v6) (errors)
    - UDP6：UDP流量(v6)
    - FC：光纤通道HBAs
    - SOFT：基于软件的网络处理
- `-o [ filename ]`：将读取数据以二进制形式保存在文件中
- `-q`：队列长度和负载均衡值统计信息
- `-r [ ALL ]`：内存利用率统计信息
- `-S`：交换空间使用率统计信息
- `-s [ hh:mm[:ss] ]`：设置数据的开始时间。默认开始时间为08:00:00（使用24小时制）。只有从文件中读取数据时才能使用此选项（选项-f）
- `-u [ ALL ]`：CPU使用率统计信息
- `-v`：内核表统计信息
- `-W`：Swapping统计信息
- `-w`：任务创建和系统切换统计
- `-y`：TTY设备统计信息
- `--help`：显示简单帮助信息

#### 示例及详细说明
查看版本：
```
[root@centos82 ~]# sar -V
sysstat version 11.7.3
(C) Sebastien Godard (sysstat <at> orange.fr)
```
间隔2秒显示三次CPU统计信息：
```
[root@centos82 ~]# sar -u 2 3
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

07:26:15 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
07:26:17 PM     all      1.51      0.00      2.77      0.00      0.00     95.72
07:26:19 PM     all      1.52      0.00      2.27      0.25      0.00     95.96
07:26:21 PM     all      2.78      0.00      2.53      0.00      0.00     94.70
Average:        all      1.93      0.00      2.52      0.08      0.00     95.46
```
每2秒报告IRQ 14的统计信息，显示3行。数据存储在int14.file文件中：
```
[root@centos82 ~]# sar -I 14 -o int14.file 2 3
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)  10/12/2023   _x86_64_   (2 CPU)
07:30:59 PM      INTR    intr/s
07:31:01 PM        14      0.00
07:31:03 PM        14      0.00
07:31:05 PM        14      0.00
Average:           14      0.00
```
显示当前每日数据文件中保存的所有统计信息：
```
[root@centos82 ~]# sar -A
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

12:00:06 AM     CPU      %usr     %nice      %sys   %iowait    %steal      %irq     %soft    %guest    %gnice     %idle
12:10:06 AM     all      2.35      0.00      1.35      0.03      0.00      0.62      0.17      0.00      0.00     95.47
12:10:06 AM       0      2.39      0.00      1.32      0.01      0.00      0.63      0.19      0.00      0.00     95.46
......
```
显示保存在每日数据文件“sa16”中的内存和网络统计信息：
```
[root@centos82 ~]# sar -r -n DEV -f /var/log/sa/sa16
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   09/16/2023      _x86_64_        (2 CPU)
12:00:05 AM kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
12:10:06 AM    147224    673956   1722932     92.13         0    638512   5925636    316.85   1178424    232796       140
12:20:06 AM    152992    675784   1717164     91.82         0    634568   5925636    316.85   1176264    230884       104
......
```
参考链接：[How to Use the sar Command on Linux](https://www.howtogeek.com/793513/how-to-use-the-sar-command-on-linux/#the-sar-command)
### vmstat命令
#### vmstat参数
主要参数说明如下：
- `-a`或`--active`：active/inactive内存
- `-f`或`--forks`：启动后的forks数量
- `-m`或`--slabs`：slabinfo
- `-n`或`--one-header`：不要重新显示标题
- `-s`或`--stats`：事件计数器统计信息
- `-d`或`--disk`：磁盘统计信息
- `-D`或`--disk-sum`：汇总磁盘统计信息
- `-p`或`--partition <dev>`：分区特定统计信息
- `-S`或`--unit <char>`：定义显示单元
- `-w`或`--wide`：间隔放宽输出
- `-t`或`--timestamp`：显示时间戳
- `-h`或`--help`：显示帮助并退出
- `-V`或`--version`：输出版本并退出

#### vmstat命令使用示例
查看事件记录数据统计信息：
```
[root@centos82 ~]# vmstat -s
      1870156 K total memory
      1088232 K used memory
      1187592 K active memory
       266024 K inactive memory
        83212 K free memory
            0 K buffer memory
       698712 K swap cache
    ......
   2543843634 interrupts
   3307181057 CPU context switches
   1692111285 boot time
     39000160 forks
```
每隔2秒显示3次统计信息：
```
[root@centos82 ~]# vmstat -a 2 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 2  0      0  73768 314752 1142512    0    0     2     8    1    1  2  2 97  0  0
 0  0      0  73336 314752 1142768    0    0     0    49 1773 3204  2  3 96  0  0
 0  0      0  73292 314752 1142776    0    0     0     8 2098 3811  2  3 96  0  0
```
每隔2秒显示3次磁盘统计信息：
```
[root@centos82 ~]# vmstat -d 2 3
disk- ------------reads------------ ------------writes----------- -----IO------
       total merged sectors      ms  total merged sectors      ms    cur    sec
vda   759761    571 31169208 1342987 25965226 190224 154388107 8430084      0  18074
vda   759761    571 31169208 1342987 25965234 190224 154388139 8430086      0  18074
vda   759761    571 31169208 1342987 25965248 190224 154388203 8430089      0  18074
```
### iostat命令
#### iostat参数说明
iostat参数说明如下：
- `-c`：显示CPU利用率报告
- `-d`：显示设备利用率报告
- `-g group_name { device [...] | ALL }`：显示设备组利用率报告
- `-H`：此选项必须与选项`-g`一起使用，表示只显示组的全局统计信息，而不显示组中单个设备的统计信息     
- `-h`：使“设备利用率报告”更易于人类阅读，`--human`通过此选项隐式启用
- `--human`：以可读格式打印大小（例如1.0k、1.2M等）使用此选项显示的单位将取代与度量相关的任何其他默认单位（例如千字节、扇区…）
- `-j { ID | LABEL | PATH | UUID | ... } [ device [...] | ALL ]`：显示持久设备名称
- `-k`：以千字节每秒为单位显示统计信息
- `-m`：以兆字节每秒为单位显示统计信息
- `-N`：显示任何设备映射器设备的已注册设备映射器名称。用于查看LVM2统计信息
- `-o JSON`：使用JSON(Javascript Object Notation)格式显示统计信息
- `-p [ { device [,...] | ALL } ]`：显示系统使用的块设备及其所有分区的统计信息
- `-s`：显示报告的短（窄）版本，该版本应适合80个字符宽的屏幕。
- `-t`：打印显示的每个报告的时间。时间戳格式可能取决于S_TIME_format环境变量的值
- `-V`：打印版本并退出
- `-x`：显示扩展统计信息
- `-y`：如果以给定的间隔显示多个记录，则省略自系统启动以来的第一个带有统计信息的报告
- `-z`：忽略在采样期间没有活动的任何设备的输出

#### iostat命令使用示例
显示自启动以来的所有CPU和设备的单个历史记录报告：
```
[root@centos82 ~]# iostat
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.65    0.01    1.76    0.03    0.00   96.55
Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               5.35         3.12        15.44   15590600   77198994
```
每隔两秒显示一次设备报告，共两次：
```
[root@centos82 ~]# iostat -d 2 2
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               5.35         3.12        15.44   15590600   77199731

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               2.00         0.00         4.00          0          8
```
每隔两秒显示一次vda设备报告，共两次：
```
[root@centos82 ~]# iostat -x vda 2 2
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.65    0.01    1.76    0.03    0.00   96.55

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
vda              0.15    5.19      3.12     15.44     0.00     0.04   0.08   0.73    1.77    0.32   0.00    20.51     2.97   0.68   0.36

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.77    0.00    2.52    0.00    0.00   94.71

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
vda              0.00    6.50      0.00     18.00     0.00     0.00   0.00   0.00    0.00    0.31   0.00     0.00     2.77   0.46   0.30
```
每隔两秒显示一次vda设备的所有分区报告，共两次：
```
[root@centos82 ~]# iostat -p vda 2 2
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos82)   10/12/2023      _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.65    0.01    1.76    0.03    0.00   96.55

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               5.35         3.12        15.44   15590600   77201302
vda1              2.96         3.12        15.44   15588884   77201302

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.00    0.00    2.00    0.00    0.00   94.00

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               6.00         0.00        16.00          0         32
vda1              4.00         0.00        16.00          0         32
```
## 磁盘性能
### 磁盘性能测试
#### dd命令
命令参数说明：
- `bs=BYTES`：一次读取和写入最多Bytes字节（默认值：512），包括ibs和obs
- `cbs=BYTES`：一次转换Bytes字节
- `conv=CONVS`：根据逗号分隔的符号列表转换文件，`CONVS`可以是：
    - ascii：从EBCDIC到ASCII
    - ebcdic：从ASCII到EBCDIC
    - ibm：从ASCII到alternate EBCDIC
    - block：用cbs大小的空格填充换行终止的记录
    - unblock：用换行符替换cbs大小记录中的尾随空格
    - lcase：将大写更改为小写
    - ucase：将小写更改为大写
    - sparse：尝试查找而不是写入NUL输入块的输出
    - swab：交换每对输入字节
    - sync：用NUL将每个输入块填充到ibs大小；与block或unlock一起使用时，使用空格而不是NUL填充
    - excl：如果输出文件已存在，则失败
    - nocreat：不创建输出文件
    - notrunc：不要截断输出文件
    - noerror：读取错误后继续
    - fdatasync：在完成之前物理写入输出文件数据
    - fsync：同上，还要写元数据
- `count=N`：仅复制N个输入块
- `ibs=BYTES`：一次最多读取Bytes字节（默认值：512） 
- `if=FILE`：从`FILE`而不是stdin读取
- `iflag=FLAGS`：按照逗号分隔的符号列表读取，`FLAGS`可以是：
    - append：附加模式（仅对输出有意义；建议使用conv=notrunc）
    - direct：对数据使用直接I/O，不经过缓存
    - directory：不是目录的情况下失败
    - dsync：对数据使用同步I/O
    - sync：同上，也适用于元数据 
    - fullblock：累积完整的输入块（仅限iflag）
    - nonblock：使用非阻塞I/O
    - noatime：不更新访问时间
    - nocache：请求删除缓存。参考oflag=sync
    - noctty：不要从文件中分配控制终端
    - nofollow：不要接收符号链接
    - count_bytes：将`count=N`视为字节计数（仅限iflag）
    - skip_bytes：将`skip=N`视为字节计数（仅限iflag）
    - seek_bytes：将`seek=N`视为字节计数（仅限oflog）
- `obs=BYTES`：一次最多写入Bytes字节（默认值：512）
- `of=FILE`：写入`FILE`而不是stdout
- `oflag=FLAGS`：按照逗号分隔的符号列表写入，`FLAGS`选项同上
- `seek=N`：在输出开始时跳过N个obs大小的块
- `skip=N`：在写入开始时跳过N个ibs大小的块
- `status=LEVEL`：要打印到stderr的信息的`LEVEL`：
    - “none”禁止除错误消息之外的所有内容
    - “noxfer”抑制最终传输统计信息
    - “progress”显示定期传输统计信息

写测试：
```sh
time dd if=/dev/zero of=/data/test bs=64k count=1000000
```
示例：
```
[root@mongodb06 sa]# time dd if=/dev/zero of=/data/test bs=64k count=1000000
1000000+0 records in
1000000+0 records out
65536000000 bytes (66 GB) copied, 107.921 s, 607 MB/s

real    1m47.929s
user    0m0.134s
sys     0m53.222s
```
读测试：
```sh
time dd if=/data/test of=/dev/null bs=64k count=1000000
```
不使用缓存直接写入：
```sh
dd if=/dev/zero of=/data/test bs=64k count=1000000 oflag=direct
```
不使用缓存直接读取：
```sh
dd if=/data/test of=/dev/null bs=64k count=1000000 iflag=direct
```
不使用缓存读写测试：
```sh
dd if=/data/test of=/data/test1 bs=64k count=1000000 iflag=direct oflag=direct
```
在执行到最后执行一次`sync`操作：
```sh
dd if=/dev/zero of=/data/test bs=64k count=1000000 conv=fdatasync
```
执行每次都会同步写入到磁盘，也就是写入64k到磁盘后再继续下一个
```sh
dd if=/dev/zero of=/data/test bs=64k count=1000000 oflag=dsync
```
## 待补充