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
