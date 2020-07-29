# 检查系统
一些检查系统状态的脚本，初学者，很多有待优化
### 检查AIX系统在升级后是否重启
#### 说明
AIX系统在升级后需要重启，但是有些情况下耽搁或者忘记了，后期如果不知道，如果是PowerHA环境，会导致切换失败。

脚本中对于获取的系统升级时间是用户自定义的系统时间，可能是CST，也可能是CDT,脚本中默认是CST，可以根据不同系统时间修改脚本；获取的重启时间固定是UTC。

在Linux中用命令： `date +%s -d "Mon Jul 06 02:03:52 UTC 2019`可以直接转换成UNIX时间戳，这样判断就很简单，但是AIX中`date`命令没有`-d`这个参数，需要自己写函数进行转换，

代码如下：
```python
#!/usr/bin/python3
#读取bos.rte安装时间和读取系统重启时间，判断在升级后系统是否重启过
import os
import time
class CheckReboot():
    upgrade_time_cmd = 'lslpp -h bos.rte|awk \'{print \"\"$4,$5\"\"}\'|tail -1'
    reboot_time_cmd = 'alog -t boot -o|grep date|awk \'{print \"\"$7,$8,$9,$10,$11,$12\"\"}\'|tail -1'
    #命令输出时间格式为：03/21/19 11:32:57，定义方法转化时间
    def __get_upgrade_time(self):
        upgrade_time = os.popen(CheckReboot.upgrade_time_cmd)
        upgrade_time = upgrade_time.read(17)
        upgrade_time = time.strptime(upgrade_time,'%m/%d/%y %H:%M:%S')
        upgrade_time = time.mktime(upgrade_time)
        print('The latest upgrade time is:'+ time.ctime(upgrade_time))
        return upgrade_time
    #命令输出时间格式为：Fri Jul 19 09:08:38 UTC 2019，定义方法转化时间
    def __get_reboot_time(self):
        reboot_time = os.popen(CheckReboot.reboot_time_cmd)
        reboot_time = reboot_time.read(28)
        reboot_time = time.strptime(reboot_time,'%a %b %d %H:%M:%S %Z %Y')
        reboot_time = time.mktime(reboot_time)
        reboot_time = reboot_time + 28800
        print('The latest reboot time is:' + time.ctime(reboot_time))
        return reboot_time
    #定义方法进行判断
    def __determine(self,upgrade_time,reboot_time):
        if upgrade_time < reboot_time:
            print('The system has been restart after upgrade!')
        else:
            print('The system did not restart after upgrade!')
    def go_check(self):
        upgrade_time = self.__get_upgrade_time()
        reboot_time = self.__get_reboot_time()
        self.__determine(upgrade_time,reboot_time)
checkreboot = CheckReboot()
checkreboot.go_check()
```
#### 示例
在AIX7100-04-03-1642中运行示例如下：
```
bash-5.0# python3 test1.py
The latest upgrade time is:Tue Jan 22 20:20:24 2019
The latest reboot time is:Mon Jul 13 21:51:49 2020
The system has been restart after upgrade!
```
### 检查AIX系统
