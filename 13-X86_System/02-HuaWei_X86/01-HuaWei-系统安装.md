# HuaWei-系统安装
使用比较少，记录下避免忘记。
## 远程虚拟光驱安装
以RH2288H V2-8S服务器为例安装RHEL7.8，步骤如下：
- 登录到服务器iMana，用户root
- 左侧菜单点击“远程控制”
- 点击最下面“远程虚拟控制台（独占模式）”
- 虚拟控制台启动后（需要java），点击上部菜单栏中的光驱选项
- 选择镜像文件夹，浏览对应的iso安装包，此处采用定制auto安装包
- 启动or重启系统
- 启动时候按F11进入boot manager
- 在boot选项中选择HUAWEI DVD-ROM VM 1.1.0
- 两个选项，Select CD-ROM Boot Type：1
- 选择AutoInstall（根据需求选择，此处测试是为了快速安装，没特别要求）
- Identify network Device显示出此机器的网络
- 设置网络，设置hostname，ip掩码网关等，DNS服务器根据需求配置，配置完成后会显示配置，并提示是否correct
- 设置磁盘，根据需求选择对应的磁盘，配置完成后会显示配置，并提示是否correct
- 确认后开始安装，安装成功后自动重启，等待系统引导完成
- 登录采用定制auto里面预设的用户密码登录即可。

## 待补充
