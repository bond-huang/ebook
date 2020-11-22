# LinuxOne-云最佳实践笔记
学习IBM官方云最佳实践视频做的笔记，视频地址：[LinuxONE高密度云最佳实践成长之路 (KVM版）](https://csc.cn.ibm.com/roadmap/index/e96159c6-cf9b-47cb-bb13-17cb5cecdaf7?eventId=)

## LinuxOne云计算参考架构
多层级虚拟划技术：
- LinuxONE全面支持涵盖逻辑分区、虚机、容器得多层级虚拟胡技术
- 单机可承载高达：85个逻辑分区，8000个虚拟机，240万个容器

多层级混合架构及多云融合解决方案
- 容器云层级：支持Docker,K8S,Openshift等主流选择
- 虚拟机层级：支持基于Openstack得各种解决
- 逻辑分区级：支持将Lpar纳管，实现分区及OS自动化部署
- 多云化扩展：支持MCM等多云解决方案，CI/CD等

### LinuxOne云计算整体架构
LinuxOne云计算主要解决方案：
- 企业级Openstack方案：IBM Cloud Infrastructure Center
- 社区支持的Openstack on LinuxOne解决方案
- RedHat OpenShit on LinuxONE
- 基于Prometheus+Grafana的全面监控解决方案

## 利用DPM实现分区创建与管理
### DMP
DMP全称Dynamic Partition Management，LinuxONE LPAR&DMP优势：
- LPAR在满足分区隔离EAL5+标准的基础上，实现CPU、网卡、HBA卡的共享
- DLPAR可以在线动态实现分区管理，如创建分区，删除分区，动态调整分区资源等

### 创建Storage Group
创建Storage Group步骤：
- 进入HMC主页面
- 选择目标LinuxONE机器
- 在“Configuretion”选项里面选择“Configure Storage”
- 在“STORAGE CARDS”选项中设定HBA卡类型，有两种：FICON Cards和FCP cards，通常使用FCP类型
- 在“CREATE STORAGE GROUP”选项中可以选择“Templates”或“Without Templates”
- 选择“Without Templates”后，接着设置通道数量：
    - Tpye选项：之前设置了HBA卡类型时FCP就会默认时FCP
    - Shearability：允许被多个分区使用，选择“Shared”并设置分区数量
    - Connecttvity：如果独占一个分区，并设置4通道，会生成4个wwpn；如果共享10个分区，通道数量是4，那么会生成40个wwpn
    - Optimized for 2nd level virtualization：可以为Z/VM的虚拟机涉及虚拟wwpn数量
- “Add Storage Volumes”对话框：用来设定磁盘数量和大小，主要两种类型：
    - Boot类型用于操作系统安装
    - Data类型用于数据
- “Name and duplicate”对话框：设定Storage Group的名字和描述；“Duplicate the storage group”可以快速的将当前设定的Storage Group进行复制
- 最后是Storage Group的概览确认，核对清楚确认无误后提交
- “Manually Send Request”：会自动生成一个文本，里面描述了生成的wwpn，可以下载文本给存储工程师进行划zone等操作
- 最后点击“Finish”，Storage Group创建完成

&#8195;&#8195;SG里面添加的volumes没有在存储端实际添加磁盘，只是对SG里磁盘数量、大小和类型做一个标记，等SG创建完成后，需要在存储端添加对应的磁盘，SG会定期检查存储实际添加的磁盘数据、大小，如果不一致，会提示不匹配

### 创建LPAR
创建Lpar步骤：
- 进入HMC主页面
- 选择相应的主机
- 选择选项“configuration”
- 选择选项“New partition”
- 进入创建LPAR引导界面:
    - Name：设置Lpar名称和Partition Type(如果是Linux选择Linux)
    - Processors：分配给分区的逻辑CPU;支持独占和共享模式
    - Memory：“Memory”是操作系统启动后获取的内存；“Max Memory”指分区启动后可以动态调整的最大值
    - NetWork：添加网卡，列表中有所有网卡
    - Storage：添加Storage Group，列表中有所有创建的Storage Group，只有状态为Complete的SG才可以被使用
    - Boot：设定启动类型，支持多种类型：FTP、SAN等等

## Redhat与SUSE在LinuxONE
### 安装RedHat操作系统
#### 利用FTP server上boot开始系统安装
设定FTP启动，步骤如下：
- 选中分区选项“Partition Details”选项
- 点击“boot”选项
- 在“Boot from”选项中选择“FTP server”
- 在对应选项中填入FTP服务器的信息
- 在“INS file”中选定ins文件

&#8195;&#8195;启动分区后会读取FTP里面的文件，会出现“Operating System Messages”界面，RedHat安装方式采用的配置文件方式，会提示等待输入配置文件。

#### 设置安装参数并启动VNC
安装参数配置是一个文本的配置文件，即之前学习过的PRM文件：
- 第一行是FTP安装源的路径
- 第二行配置用到的网卡信息，例如layer类型，假如添加的网卡叫3000，那么需要输入3000、3001和3002，是LinuxONE上独有的，规则如下：
    - 3000代表读
    - 3001代表写
    - 3002代表data
- 第三行配置分区信息：IP、网关、掩码及hostname
- 接下来四行是配置目标存储的路径
- 最后“vnc”表示会启用vnc service去安装
- 在“Operating System Messages”界面输入刚才准备的参数，建议三行粘贴一次
- 所有配置文件输入完成后，输入英文符号点号，输入回车即可提交
- 配置参数无误，会出现启动vnc的提示：首先要通过ssh连接到机器，会自动启动vnc service

#### 登录VNC完成后续安装
登录vnc后步骤：
- 设置语言
- 然后是设置面板：主要设定都可以在此设定，例如软件包等
- 设定完成后点击“Begin installation”
- 等待几分钟，提示“Complete”代表安装完成
- 点击“Reboot”重启

### 安装SUSE操作系统
#### 利用FTP server上boot开始系统安装
步骤如下：
- 选中分区选择“Partition Details”选项
- 点击“boot”选项
- 在“Boot from”选项中选择“FTP server”
- 在对应选项中填入FTP服务器的信息
- 在“INS file”中选定ins文件
- 点击保存，然后启动分区：选中分区，选择Daily--Start
- 分区启动完成后提示“success”
- 启动完成后，选中分区，选择Daily--Operating System Messages

#### 设置安装参数并启动VNC
&#8195;&#8195;打开“Operating System Messages”后可以看到和RedHat差不多界面，不过RedHat采用的是配置文件模式，SUSE采用的是交互式模式，步骤如下：
- 选择“1”：“Start installation”
- 继续选择“1”：“Installation”开始安装
- 选择安装方式，这里选择的是“Network”，其它方式暂不支持
- 选择网络安装方式，这里选的是“FTP"
- 配置网卡，会列出当前分区已经配置的网卡，选择对应的网卡
- 输入端口值，默认情况下是“0”，如果是接在第二个口，选择“1”
- 设置网卡的三个通道地址，设定读通道地址（0.0.0001）、写通道地址（0.0.0002）及data通道地址（0.0.0003）
- 设置layer：根据网络需求设置对应类型
- 设置IP地址、掩码及网关，敲回车继续
- 设置name server：如果有就设定，没有就空着
- 设置domain：如果有就设定，没有就空着
- 设置FTP服务器的地址
- 设置安装介质的路径
- 设置FTP用户名和密码
- 提示是否使用HTTP proxy，如果环境里面没有网络代理就设置“No”
- 最后提示后续安装的方式，例如选择“vnc”，然后设置vnc的密码
- 设置完成后就可以通过vnc进行连接：
    - 打开vnc客户端
    - 在“VNC Server”选项里面输入目标端地址：IP:1，代表连接的目标5901端口
    - 输入密码，点击“OK”继续

#### 登录VNC完成后续安装
启动VNC后步骤：
- 设置语言：默认English，并同意协议后点击“next”
- 配置磁盘：点击“Configure ZFCP Disks”
    - 在“Configure ZFCP Device”界面点击“Add”进行添加
    - “Channel ID”对应的是Storage Group里面的device ID，会自动扫描，下拉菜单选择对应即可
    - 在“Configure ZFCP Device”界面里会看到添加的disks
    - 点击“next”继续
- 在“Registration”界面提示是否注册，跳过即可
- “Suggested Partitioning”界面会显示磁盘默认分区情况，如果想定制点击“Create Partition Setup”
- 选择时区：选择对应时区即可
- “Local Users”：创建用户，不是必须选项，可以创建也可以跳过
- 设置root密码，点击“next”继续
- “Installation Settings”：会显示当前SUSE系统的相关配置
- 确认无误后点击“Install”开始安装
- 等待几分钟后即可安装完成

### 设置从SAN启动操作系统
&#8195;&#8195;在LinuxONE中，任何操作系统安装完成后，都需要把boot选项从之前的FTP改成SAN启动，否则系统会无法启动，步骤如下：
- 选中分区选择“Partition Details”选项
- 点击“boot”选项
- 在“Boot from”选项中选择“Storage Group(SAN)”
- 然后自动列出Storage Group里的boot磁盘，选择对应即可
- 然后保存

最后将操作系统stop然后start后，整个操作系统即安装完成。

## 利用ECS实现LinuxONE LPAR的 批量安装部署
### LinuxONE ECS介绍
端到端的LinuxONE及SAN存储资源自动化管理解决方案：
- 自动化LinuxONE分区部署
- 自动创建存储卷和交换机zone
- 根据魔棒自动安装Linux操作系统
- 标准化LinuxONE资源配置流程
- 通过角色管理实现资源管理审批流程
- 完备的部署日志
- 通过Prometheus和Grafana全面监控LinuxONE系统
- 多合一LinuxONE设备一站管理
- 通过添加可选的存储日志分析和监控组件实现SAN fabric拓扑自动生成、SAN fabric日志、告警及问题诊断分析

#### 核心功能
资源部署：
- 一键部署：在获取LinuxONE和存储设备操作权限的前提下，ECS可以实现一键部署分区、配置存储、及安装操作系统
- 标准部署流程：LinuxONE、存储、管理员分别配置自己的管理网络范围内的i元，并最终完成系统安装
- 主流操作系统支持：支持主流企业级操作系统RedHat和SUSE的自动化安装

工作流体系：
- 自定义部署流程：管理员通过配置指定需要ECS自动完成的系统配置
- 资源操作全流程追踪：准确记录资源操作，确保任何资源所有的操作可以追溯

资源监控：
- 基于开源的监控：基于Prometheus及Grafana对逻辑分区以及操作系统进行监控
- 资源展示、统计、报警：根据用户需求宁或定制展示LinuxONE和存储资源内容，实现监控、数据统计及预警

权限控制：
- 多种管理员角色：ECS针对LinuxONE的资源部署类型进行了角色划分，共分为LinuxONE管理吗，存储管理员，网络管理员以及一个超级管理员，并针对各角色赋予不同管理权限

平台整合：
- RESTful API接入：ECS支持第三方系统通过RESTful API接入

#### 平台特性
