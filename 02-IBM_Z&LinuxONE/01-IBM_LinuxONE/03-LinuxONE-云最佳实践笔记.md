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

## 待补充
