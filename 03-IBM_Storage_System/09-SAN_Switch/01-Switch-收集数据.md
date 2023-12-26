# Switch-收集数据
&#8195;&#8195;交换机硬件架构相对比较简单，故障也相对也少。大多数故障是SFP模块故障，有时候也有一些内部微码问题，当然也有一些外部问题需要通过交换机日志来进行排查。
### SAN B-type 数据收集
IBM SAN B-type系列交换机是OEM博科的，但是从交换机图形界面(EZSwitchSetup)收集的日志IBM support的工具解析不了。

一般收集IBM B-type系列交换机日志有两种：
- 一般硬件故障时候：supportshow
- 较复杂的问题：supportsave

##### supportshow收集
就是收集命令`supportshow`的输出
以Xsehll为例，登录到交换机后，在输入命令前依次点击Xsehll选项：文件-日志-启动
在交换机上运行命令：
```shell
supportshow
```
运行完成后依次点击Xsehll选项：文件-日志-停止
将Xsehll输出的文件重命名保存即可。
### supportsave收集
收集条件：
- 远程ftp服务器，文件提取到本地方便
- 服务器上指定目录有写权限
- 目录建议新建个空目录，因为内容较多

收集步骤如下：
- 运行命令：`supportsave`
- 提示"OK to proceed?"时输入：`yes`
- 提示"Host IP or Host Name"时输入ftp服务器ip
- 根据提示输入ftp访问用户名和密码
- 提示"Protocol"时输入：`ftp`
- 提示"Remote Directory"时输入ftp服务器上的指定目录
- 等待收集完成
- 完成后在ftp服务器上指定目录下的所有文件压缩打包

建议压缩打包成zip格式，IBM support远程工具不能解析rar格式。

### SAN C-type 数据收集
暂未接触到此产品
