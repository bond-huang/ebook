# HMC中shell脚本使用
## 简介
&#8195;&#8195;HMC底层是Redhat，经过修改和限制后，还是相对比较封闭；一般登录是采用hscroot这个用户，此用户权限一般，很多命令没有，此用户也无法ftp。     
&#8195;&#8195;官方有提供一些小工具去抓取一些数据，例如HMC Scanner，简单的可以直接在命令行进行输入，例如用for循环批量创建分区和分区资源的时候。
## HMC Scanner
&#8195;&#8195;HMC Scanner可以用于HMC配置信息收集，主要是Lpar的一些重要配置信息，会自动生成一个配置表格。此工具有shell也有java，跑的时候还是比较慢，耐心等待，但是数据确实很全面的。
官方链接：
[HMC Scanner](https://www.ibm.com/support/pages/node/1117515?mhsrc=ibmsearch_a&mhq=HMC%20scanner)
由于HMC的一些局限性，我觉得此工具提供了一个很好的接口，在上面进行修改或者添加，得到自己定制的内容，也是很不错的。
## 命令
使用过程中发现HMC很多命令或者工具没有：
- hscroot用户的hmcbash中没有awk和gawk程序，有sed编辑器
- 没有脚本执行的命令，使用`sh`及`./`都不行
- 没用ftp和sftp，有scp

## 运算问题
&#8195;&#8195;近期在HMC中用hscroot用户跑一个for循环去对分区进行批量资源创建的时候，发现运算还是有点不一样。在V9R1版本和V7R7.9版本中都进行了如下运算测试：
```sh
hscroot@hmc:~> for i in {2..5}
> do
> echo Lpar$(echo "$i*10+1"|bc)
> done
```
运行后都是会报错：bash: bc: command not found，说明HMC的hscroot用户没bc计算器，用下面运算方法替代即可：
```sh
hscroot@hmc:~> for i in {2..5}
> do
> echo Lpar$(($i*10+1))
> done
```
或者用方括号：
```sh
hscroot@TEST:~> for i in {2..5}
> do
> echo Lpar$[ $i * 10 + 1 ]
> done
```
运行后结果如下：
```shell
Lpar21
Lpar31
Lpar41
Lpar51
```
## 获取分区信息
&#8195;&#8195;抓取HMC受管机器的已激活分区信息：所在机器名称、分区名称、分区ip、分区操作系统版本。但是HMC的hscroot用户的有很多限制，很多命令用不了，awk和gawk都用不了，重定向也被限制了，研究了一下hscroot用户也配置不了ssh免密登录。当然也可以使用HMC Scanner，抓取信息很详细，抓取后后期也需要整理，想了个办法抓取这些信息，除了输入多次密码比较麻烦，整理是还算简单。
### 获取信息的命令
#### 通过机器获取对应信息
在HMC中，获取机器信息的命令如下：
```shell
# 获取受管机器信息
hscroot@TEST:~> lssyscfg -r sys -F
# 获取Lpar信息
hscroot@TEST:~> lssyscfg -r lpar -m Server-9117-570-SN65B4D6E -F
```
找一台可以ssh到HMC的机器，我找了个AIX系统，首先获取受管机器信息：
```
# ssh hscroot@9.210.114.112 "lssyscfg -r sys -F" > sysinfo.txt
Password: 
#
```
然后抓取受管机器名字：
```
# awk 'BEGIN{FS=","}{print $1}' sysinfo.txt > syslist.txt
# cat syslist.txt
Server-9117-570-SN65YDR6E
Server-9117-570-SN7578HB1
Server-9131-52A-SN066HC5G
Server-9117-MMA-SN10GF66F
```
然后获取Lpar信息（循环一次输入一次密码）：
```
# for i in `cat syslist.txt`
do 
ssh hscroot@9.210.114.112 "lssyscfg -r lpar -m $i -F"
done >> lparinfo.txt
Password: 
Password: 
Password: 
Password: 
# cat lparinfo.txt
teacher02 9.210.114.218,10,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6EA,client08,client08,none,0,0,none,norm,null,norm,0,0,active,9.210.114.218,1,1,1,1,0,08A64825-D324-4FBB-A6D3-D44143
ED8AF3
teacher01 9.220.154.227,9,aixlinux,Running,1,AIX 7.1 7100-04-03-1642,65YDR6E9,client07,client07,none,0,0,none,norm,null,norm,0,0,inactive,9.220.154.227,0,0,0,0,1,3E8A72FB-C815-4AAD-9754-89037
455DCFC
```
抓取需要的信息：
```sh
awk 'BEGIN{FS=","; OFS=" "}{if ($4 == "Running") print $1 $6 $7 $20}' lparinfo.txt > result.txt
```
在AIX中设置了OFS好像没什么效果,基本上已获取了需求的信息，AIX中使用下面命令也行：
```
awk -F, '{if ($4 == "Running") print $1 $6 $7 $20}' lparinfo.txt > result.txt
```
可以单独把ip取出来，遍历IP表去登录到系统（如果没有配置自动登录就手动输入密码）：
```sh
for i in `cat ip.list`;do ssh tmpusr@$1;done
```
#### 分区对应信息
&#8195;&#8195;如果只获取分区的IP和操作系统版本，RMC状态，LparID等，不需要对应物理机器，可以使用`lspartition`命令，获取的信息相对较少，但是清晰明了，字段以逗号隔开，awk命令中指定字段分隔符即可，使用示例如下：
```
hscroot@TEST:~> lspartition -i                                                           
12,9.200.104.174,0;5,9.200.104.233,0;1,9.200.104.108,3;10,9.200.104.238,0;1,9.200.104.107,
3;1,9.200.104.19,0;3,9.200.104.231,0;1,9.200.104.172,0;2,9.200.104.173,0;7,9.200.104.235,0
```
输出格式为:`LParID,IPaddress,active;`
```
hscroot@TEST:~> lspartition -ix    
12,9.200.104.174,0,,AIX,7.1;5,9.200.104.233,0,,AIX,6.1;1,9.200.104.108,3,,AIX,6.1;10,9.200
.104.238,0,,AIX,7.1;1,9.200.104.107,3,,AIX,6.1;1,9.200.104.19,0,,AIX,7.1;3,9.200.104.231,0,,AIX,5.3;1,9.200.104.172,0,,AIX,6.1;2,9.200.104.173,0,,AIX,6.1;7,9.200.104.235,0,,AIX,7.
```
输出格式为:`LParID,IPaddress,active,hostname,OStype,OSlevel;`
### 抓取分区配置信息
统计所有HMC上受管机器的CPU内存信息，当然也报告Lpar名称和IP等。

首先收集HMC上受管机器的信息：
```
# ssh hscroot@10.8.252.150 "lssyscfg -r sys -F" > sysinfo.txt
```
抓取受管机器名称：
```
# awk 'BEGIN{FS=","}{print $1}' sysinfo.txt > syslist.txt
```
抓取受管机器的CPU配置信息：
```
for i in `cat syslist.txt`
do
ssh hscroot@10.8.252.150 "lshwres -r proc -m $i --level sys" > sysproc.txt
done
```
然后筛选一下：
```
# awk 'BEGIN{FS=","}{print $1,$2,$4}' sysproc.txt > procinfo.txt
```
抓取受管机器的内存配置信息：
```
for i in `cat syslist.txt`
do
ssh hscroot@10.8.252.150 "lshwres -r mem -m $i --level sys" > sysmem.txt
done
```
然后筛选一下：
```
# awk 'BEGIN{FS=","}{print $1,$2,$4}' sysmem.txt > meminfo.txt
```

然后获取所有分区profile信息：
```
for i in `cat syslist.txt`
do
ssh hscroot@10.8.252.150 "lssyscfg -r prof -m $i"
done > lparprof.txt
```
获取分区的名称，内存和CPU配置信息：
```
# awk 'BEGIN{FS=","}{print $2,$7,$16} lparprof.txt > lparinfo.txt
```
发现用lshwres获取CPU内存信息也可以，并且更加方便，获取CPU信息：
```
# for i in `cat syslist.txt`
do
ssh hscroot@10.8.252.150 "lshwres -r proc -m $i --level lpar" 
done > lparproc.txt
```
提取出来：
```
# awk 'BEGIN{FS=","}{print $1,$7}' lparproc.txt > lparprocinfo.txt
```
内存一样：
```
# awk 'BEGIN{FS=","}{print $2,$7,$16} lparprof.txt > lparinfo.txt
```
发现用lshwres获取CPU内存信息也可以，并且更加方便，获取CPU信息：
```
# for i in `cat syslist.txt`
do
ssh hscroot@10.8.252.150 "lshwres -r mem -m $i --level lpar" 
done > lparmem.txt
# awk 'BEGIN{FS=","}{print $1,$7}' lparmem.txt > lparmeminfo.txt
```
说明：
- 物理Lpar和虚拟的vios的CPU配置信息格式有点不一样
- 物理Lpar和虚拟的vios内存配置信息格式一样

## 待补充
