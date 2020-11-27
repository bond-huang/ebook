# LinuxONE-通过RKE在LinuxONE上自动化部署Kubernetes集群
&#8195;&#8195;学习IBM官方LinuxONE高密度云最佳实践视频做的笔记，步骤基本摘自官方，结合上机实践步骤，整理成此文档方便查阅巩固知识。官方地址：[LinuxONE高密度云最佳实践成长之路 (KVM版）](https://csc.cn.ibm.com/roadmap/index/e96159c6-cf9b-47cb-bb13-17cb5cecdaf7?eventId=)

## Kubernetes简介
&#8195;&#8195;Kubernetes又称为k8s（首字母为k、首字母与尾字母之间有8个字符、尾字母为s，简称 k8s）或者简称为 "kube"，是一种可自动实施Linux容器操作的开源平台。最初由Google的工程师开发和设计。 

Kubernetes官网：[https://kubernetes.io/](https://kubernetes.io/)     
Kubernetes中文社区：[https://www.kubernetes.org.cn/](https://www.kubernetes.org.cn/)     
Kubernetes中文文档：[https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)     
Kubernetes红帽官方简介：[https://www.redhat.com/zh/topics/containers/what-is-kubernetes](https://www.redhat.com/zh/topics/containers/what-is-kubernetes)      

## 部署结构
&#8195;&#8195;RKE是本身作为一个部署工具，可以运行在Kubernetes集群中的节点上，也可以运行在单独的一个节点。本示例四以一个三节点的kubernets集群（由于需要通过Internet获取Kubernetes的镜像，所以要求3个Kubernetes节点可以直接访问docker.io镜像库）为例，信息如下：

IP address|Hostname|Role|Config
:---:|:---:|:---:|:---:
192.168.122.158|rke1|cluster node|2c/4g
192.168.122.147|rke2|cluster node|2c/4g
192.168.122.151|rke3|cluster node|2c/4g
192.168.122.1/172.16.27.86|ubu1|deploy node|

## 准备工作
在每个Kubernetes节点上完成。 
### 防火墙配置   
关闭firewalld：
```
# systemctl stop firewalld
# systemctl disable firewalld
```
关闭selinux：
```
# setenforce 0
```
编辑`/etc/selinux/config`并配置:
```
SELINUX=disabled
```
### 安装docker-ce
创建一个新的工作组：
```
# groupadd docker
```
下载Docker-ce的binary:
```
# wget -O https://download.docker.com/linux/static/stable/s390x/docker-18.06.3-ce.tgz
# tar xfz docker-18.06.3-ce.tgz
# cp docker/* /usr/bin/​
```
#### 创建docker-ce的systemd服务配置
创建文件/usr/lib/systemd/system/docker.service:
```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service
Wants=network-online.target
Requires=docker.socket
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd://
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=1048576
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target​
```
创建文件/usr/lib/systemd/system/docker.socket:
```
[Unit]
Description=Docker Socket for the API
PartOf=docker.service
[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker
[Install]
WantedBy=sockets.target
```
启动docker:
```
# systemctl start docker
# systemctl enable docker​
```
### 创建新用户
由于rke不能使用root用户部署，所以需要在每个节点上创建一个新的用户：
```
# useradd -g docker -d /home/user1 -m user1
```
创建密码,用于deploy节点ssh访问:
```
# passwd user1
```
## 获取RKE s390x版本
在deploy节点上获取RKE-s390x工具:
```
# wget https://releases.s3.cn-northwest-1.amazonaws.com.cn/rke-s390x
# chmod +x rke-s390x
# cp rke-s390x /usr/bin/rke
```
## 设置ssh免密码登陆
设置如下：
```
$ ssh-copy-id user1@rke1
$ ssh-copy-id user1@rke2
$ ssh-copy-id user1@rke3
```
## 配置rke 配置文件
&#8195;&#8195;在deploy节点上，创建cluster.yml文件，内容如下（可以根据环境，调整nodes信息，包括ip地址，访问用户，role）：

```
nodes:
- address: 192.168.122.158
  port: "22"
  internal_address: ""
  role:
  - controlplane
  - worker
  - etcd
  hostname_override: ""
  user: user1
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
- address: 192.168.122.147
  port: "22"
  internal_address: ""
  role:
  - controlplane
  - worker
  - etcd
  hostname_override: ""
  user: user1
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
- address: 192.168.122.151
  port: "22"
  internal_address: ""
  role:
  - controlplane
  - worker
  - etcd
  hostname_override: ""
  user: user1
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
services:
  etcd:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    external_urls: []
    ca_cert: ""
    cert: ""
    key: ""
    path: ""
    uid: 0
    gid: 0
    snapshot: null
    retention: ""
    creation: ""
    backup_config: null
  kube-api:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    service_cluster_ip_range: 10.43.0.0/16
    service_node_port_range: ""
    pod_security_policy: false
    always_pull_images: false
    secrets_encryption_config: null
    audit_log: null
    admission_configuration: null
    event_rate_limit: null
  kube-controller:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    cluster_cidr: 10.42.0.0/16
    service_cluster_ip_range: 10.43.0.0/16
  scheduler:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
  kubelet:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    cluster_domain: cluster.local
    infra_container_image: ""
    cluster_dns_server: 10.43.0.10
    fail_swap_on: false
    generate_serving_certificate: false
  kubeproxy:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
network:
  plugin: flannel
  options: {}
  mtu: 0
  node_selector: {}
  update_strategy: null
authentication:
  strategy: x509
  sans: []
  webhook: null
addons: ""
addons_include:
system_images:
  etcd: ibmcom/etcd-s390x:3.2.24
  alpine: openjupyter/rke-tools-s390x:v0.1.52
  nginx_proxy: openjupyter/rke-tools-s390x:v0.1.52
  cert_downloader: openjupyter/rke-tools-s390x:v0.1.52
  kubernetes_services_sidecar: openjupyter/rke-tools-s390x:v0.1.52
  kubedns: rancher/k8s-dns-kube-dns:1.15.0
  dnsmasq: rancher/k8s-dns-dnsmasq-nanny:1.15.0
  kubedns_sidecar: rancher/k8s-dns-sidecar:1.15.0
  kubedns_autoscaler: rancher/cluster-proportional-autoscaler:1.7.1
  coredns: rancher/coredns-coredns:1.6.7-s390x
  coredns_autoscaler: openjupyter/cluster-proportional-autoscaler-s390x:1.7.1
  kubernetes: gcr.io/google_containers/hyperkube-s390x:v1.17.4
  flannel: quay.io/coreos/flannel:v0.11.0-s390x
  flannel_cni: openjupyter/flannel-cni-s390x:v0.3.0
  calico_node: ibmcom/calico-node:v3.5.2.1
  calico_cni: ibmcom/calico-cni:v3.5.2.1
  calico_controllers: ibmcom/calico-kube-controllers:v3.5.2.1
  calico_ctl: ibmcom/calico-ctl:v2.0.2
  calico_flexvol: rancher/calico-pod2daemon-flexvol:v3.13.0
  canal_node: rancher/calico-node:v3.13.0
  canal_cni: rancher/calico-cni:v3.13.0
  canal_flannel: rancher/coreos-flannel:v0.11.0
  canal_flexvol: rancher/calico-pod2daemon-flexvol:v3.13.0
  weave_node: weaveworks/weave-kube:2.5.2
  weave_cni: weaveworks/weave-npc:2.5.2
  pod_infra_container: ibmcom/pause:3.1
  ingress: ibmcom/nginx-ingress-controller:0.23.1
  ingress_backend: openjupyter/defaultbackend-s390x:1.4
  metrics_server: rancher/metrics-server:v0.3.6
  windows_pod_infra_container: rancher/kubelet-pause:v0.1.3
ssh_key_path: ~/.ssh/id_rsa
ssh_cert_path: ""
ssh_agent_auth: false
authorization:
  mode: rbac
  options: {}
ignore_docker_version: false
kubernetes_version: ""
private_registries: []
ingress:
  provider: ""
  options: {}
  node_selector: {}
  extra_args: {}
  dns_policy: ""
  extra_envs: []
  extra_volumes: []
  extra_volume_mounts: []
  update_strategy: null
cluster_name: ""
cloud_provider:
  name: ""
prefix_path: ""
addon_job_timeout: 0
bastion_host:
  address: ""
  port: ""
  user: ""
  ssh_key: ""
  ssh_key_path: ""
  ssh_cert: ""
  ssh_cert_path: ""
monitoring:
  provider: ""
  options: {}
  node_selector: {}
  update_strategy: null
  replicas: null
restore:
  restore: false
  snapshot_name: ""
dns: null
```
如果使用自动创建集群初始化配置文件：
```
$ rke config --name cluster.yml
```
## 部署Kubernetes集群
在cluster.yml文件所在的目录运行如下命令：
```
$ rke up
```
如果指向指定配置文件：
```
$ rke up --config /tmp/cluster.yml
```
## 配置kubectl
rke部署结束后，会生成kube_config_cluster.yml文件，需要放在默认的kubectl配置路径下：
```
mkdir .kube
cp kube_config_cluster.yml .kube/config
```
下载kubectl:
```
wget https://dl.k8s.io/v1.18.0/kubernetes-client-linux-s390x.tar.gz
tar xf kubernetes-client-linux-s390x.tar.gz
cp kubernetes/client/bin/kubectl /usr/bin
```
## 验证kubectl是否工作
验证信息如下表：

IP address|Status|Role|Age|Version
:---:|:---:|:---:|:---:|:---:
192.168.122.147|Ready|controlplane,etcd,worker|9d|v1.17.4
192.168.122.151|Ready|controlplane,etcd,worker|9d|v1.17.4
192.168.122.158|Ready|controlplane,etcd,worker|9d|v1.17.4

## 系统中心访问方式
ssh到ubu1(172.16.27.86)上，可以访问该三节点的Kubernetes集群:

IP address|User|Password
:---:|:---:|:---:
172.16.27.86|root|zlinux
192.168.122.158|root|zlinux
192.168.122.147|root|zlinux
192.168.122.151|root|zlinux

