# AIX-常用命令和操作
## 硬件类
slibclean  除去内科和库中任何当前不用的模块
netstat -ni 查看网络配置
autoconf6 配置IPv6
netstat -rn 查看路由
netstat -a

netstat -v

entstat -d ent0 |grep -i vlan

tcpdump -lni en7 |grep -i echo 

entstat -d device 诊断IEEE 802.3ad etherchannel问题

entstat entX

svmon -G
svmon -P

netstat -in

chdev -l enX -a mtu_bypass=on

# refresh -s clcomdES
# stopsrc -s clcomdES;startsrc -s clcomdES

[AIX常用命令](https://www.ibm.com/developerworks/cn/aix/library/au-dutta_cmds.html?mhsrc=ibmsearch_a&mhq=AIX%E7%94%A8%E5%91%BD%E4%BB%A4%E6%B0%B8%E4%B9%85%E6%B7%BB%E5%8A%A0%E8%B7%AF%E7%94%B1)

du -sg

ssh -v <host>

###
&#8195;&#8195;有个需求，给网卡配置一个IP,但是不想启用，启用就会IP冲突，如果直接配置在`START Now`里选择`No`的话IP也会自动生效，配置方法：
- 首先配置一个无效的IP在此网卡上
- 然后修改网卡属性：
    - smit tcpip
    - Further Configuration
    - Network Interfaces
    - Network Interface Selection
    - Change / Show Characteristics of a Network Interface
    - 选择需要修改的网口
- 在Change / Show a Standard Ethernet Interface界面中：
    - INTERNET ADDRESS (dotted decimal)中填入需求的正确IP
    - Current STATE选项里面选择detach
    - 回车确认
- 需要启用ip时使用命令`ifconfig en1 up`即可

强制删除非空目录
rm -rf wkhtmltopdf


scp /tmp/aix7236.iso tmpusr@10.8.222.11:/tmp

目录：
scp -r /tmp/aix tmpusr@10.8.222.11:/tmp

#‌ oslevel -s

7200-03-05-2016

#‌ oslevel -rl 7200-04

Fileset Actual Level Recommended ML

-----------------------------------------------------------------------------

powerscStd.vtpm.rte 1.1.4.2 1.1.4.3 

sysmgt.cfgassist 7.2.3.16 7.2.4.0 

#‌ lppchk -vm3

#‌ instfix -i |grep ML

All filesets for 7.2.0.0_AIX_ML were found.

All filesets for 7200-00_AIX_ML were found.

All filesets for 7200-01_AIX_ML were found.

All filesets for 7200-02_AIX_ML were found.

All filesets for 7200-03_AIX_ML were found.

Not all filesets for 7200-04_AIX_ML were found.



After you download the TL, locally :
#cd /media directory
#installp -acFXYd . powerscStd.vtpm
#installp -acXYd . sysmgt.cfgassist


#cd <install-media- directory>
#installp -aXYgd . -e /tmp/install.log sysmgt.cfgassist
#installp -c all

and run and send me output of below commands:

#lppchk -v
#oslevel -s


installp -aXYgd . -e /tmp/install_sysmgt.log sysmgt.cfgassist





Additional info,
Please download the following files from below link:
https://developer.ibm.com/javasdk/support/aix-download-service/
>Java8_64.jre.tar.gz (140255885)
>Java8_64.sdk.tar.gz (14418713)

Thank you.

02:36 am
07/25

Good day Junaid,

Thank you for raising Case# TS002515360 with IBM Software Support.
I am Arlina from AIX software support and I will be working with you on the case.

You may download Java8 from below link.
Before you can download code, you need an IBM Registration ID.
https://developer.ibm.com/javasdk/support/aix-download-service/

For more reference:
IBM Java for AIX HowTo: Install or upgrade IBM Java to a specific release
https://www-01.ibm.com/support/docview.wss?uid=isg3T1022693




Resolution summary:He can download the latest version of the Java fileset from :
https://www-01.ibm.com/support/docview.wss?uid=isg3T1022644
After you download them for each of the filesets:
#gzip <fileset name>
#tar -xvf <fileset name>.tar
#cd <fileset directory>
#inutoc .
#installp -acXYd . <fileset name>
