# BladeCenter-常见故障

## 刀箱管理问题
### 刀箱管理刀片问题
#### Serial Over LAN异常
通过刀箱命令行进入刀片系统时候无法进入，示例：
```sh
system> console -o -T blade[4]
SOL is not ready
```
查看对应刀片的SOL状态命令：
```sh
sol -T blade[4]
```
或者在AMM管理界面：
- 主页左侧导航--Blade Tasks
- 点击`Serial Over LAN`选项，查看状态
- 选中需要操作的刀片，在Available actions里面尝试disable后再enable

## 刀片故障
### 刀片主板故障
#### 示例1
报错示例：
```
E SERVPROC 08/26/26 01:20:38 0x6f60c001 Media Tray 1 hardware failure
W SERVPROC 08/26/26 01:20:41 0x06a2e001 Chassis temperature device is unavailable. Cooling capacity set to maximum.
E Blade_02 08/26/26 06:20:39 0x10000002 SYS F/W: Error. Replace Sys Brd then (50D55415 11002647 003C0001 00002647 0000000000000000 00000000 00000000 00000000 00000000)
E Blade_02 08/26/26 06:20:39 0x806f0021 System board, connector (Sys Brd) fault
I Blade_02 08/26/26 06:21:30 0x806f0009 System board, (SysPwr Monitor) power off
```
主板报错，直接更换主板，换完第一块起不来，报错示例：
```
SYS F/W: Firmware. See procedure FSPSP04 (50769BC3 B181B50A 010100F0 531B3A10 C1009104 200000C5 00000000 00000006 C6C6C6C6 00000000)
```
继续换，报错要PCI卡:
```
SYS F/W: Firmware. See procedure FSPSP04 (50769BC3 B181B50A 010100F0 531B3A10 C1009104 200000C5 00000000 00000006 C6C6C6C6 00000000)
```
重新拔插下后正常启动了。
#### 示例2
系统挂掉无法进去，刀箱没有刀片相关报错，修复过程：
- 重启刀片，看起来正常，但是很卡
- 能进入到sms选着引导，但是找不到硬盘
- 再重启经常打不开sms，非常卡，刀箱没有相关报错
- 一开始怀疑硬盘问题，最后换了主板后可以正常引导，还是主板问题

## 待补充