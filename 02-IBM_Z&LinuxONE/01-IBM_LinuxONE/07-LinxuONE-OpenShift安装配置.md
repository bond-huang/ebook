# LinuxONE-OpenShift在LinuxONE上的安装配置_KVM
&#8195;&#8195;学习IBM官方LinuxONE高密度云最佳实践视频做的笔记，步骤基本摘自官方，结合上机实践步骤，整理成此文档方便查阅巩固知识。官方地址：[LinuxONE高密度云最佳实践成长之路 (KVM版）](https://csc.cn.ibm.com/roadmap/index/e96159c6-cf9b-47cb-bb13-17cb5cecdaf7?eventId=)

## OpenShift简介
&#8195;&#8195;OpenShift是由红帽（RedHat）开发的容器化软件解决方案，这是基于企业级Kubernetes管理的平台即服务（PaaS）。OKD是嵌入在Red Hat OpenShift中的上游Kubernetes发行版。       
&#8195;&#8195;Kubernetes又称为k8s（首字母为k、首字母与尾字母之间有8个字符、尾字母为s，简称 k8s）或者简称为 "kube"，是一种可自动实施Linux容器操作的开源平台。最初由Google的工程师开发和设计。       
&#8195;&#8195;OpenShift 虚拟化是红帽OpenShift的一项功能，其将每个虚拟机（VM）封装在特殊容器内，便于您对传统应用以及新的云原生工作负载和无服务器工作负载进行现代化改造，且全部可通过单一的 Kubernetes 原生架构进行管理。

官方网站：[https://www.openshift.com/](https://www.openshift.com/)        
官方中文网站：[https://www.redhat.com/zh/technologies/cloud-computing/openshift](https://www.redhat.com/zh/technologies/cloud-computing/openshift)    
OKD GitHub：[https://github.com/openshift/origin](https://github.com/openshift/origin)     
Kubernetes简介：[https://www.redhat.com/zh/topics/containers/what-is-kubernetes](https://www.redhat.com/zh/topics/containers/what-is-kubernetes)

## 系统环境说明
介绍配置安装配置的环境。
### 宿主机系统信息
为了使用KVM支持，测试使用的是Red Hat Enterprise Linux 7.6 ALT版本，系统信息如下：
```
Static hostname: lpar07
       Icon name: computer
      Machine ID: d93aa775a60e4447bb1746db80a9eecb
         Boot ID: c5c2b1a68cda4c4283d65f70c9cf2555
Operating System: Red Hat Enterprise Linux Server 7.6 (Maipo)
     CPE OS Name: cpe:/o:redhat:enterprise_linux:7.6:GA:server
          Kernel: Linux 4.14.0-115.el7a.s390x
    Architecture: s390x
```
### Openshift 集群环境参数
集群环境参数如下：
- 1台Bootstrap主机: 192.168.122.30
- 3台Master主机: 192.168.122.31-33
- 2台Worker主机: 192.168.122.34-35
- 实验环境需使用的代理服务器: http://172.16.15.192:3128/
- 实验环境KVM子网: 192.168.122.0/24
- 实验环境KVM网关: 192.168.122.1
- 实验环境主域名: example.com
- Openshift集群名: ocp

### 软件包
需求的软件包如下：
```
# yum install libvirt virt-install libvirt-daemon-kvm httpd
# wget http://172.16.27.78/pub/Packages/haproxy-1.5.18-9.el7.s390x.rpm
# yum install haproxy-1.5.18-9.el7.s390x.rpm
```
## 宿主机准备工作
### 配置DNS服务（使用dnsmasq服务）
DNS服务配置是依据Openshift安装的官方文档要求，官方文档：[User-provisioned DNS requirements](https://docs.openshift.com/container-platform/4.2/installing/installing_ibm_z/installing-ibm-z.html#installation-dns-user-infra_installing-ibm-z)
#### 配置主机的正向及反向域名解析
请注意确认在原始hosts文件中没有以上IP地址的任何记录，如果有请先删除后再操作：
```
# cat <> /etc/hosts
192.168.122.1 lpar07.ocp.example.com
192.168.122.1 api.ocp.example.com
192.168.122.1 api-int.ocp.example.com
192.168.122.30 bootstrap.ocp.example.com
192.168.122.31 master-0.ocp.example.com
192.168.122.32 master-1.ocp.example.com
192.168.122.33 master-2.ocp.example.com
192.168.122.31 etcd-0.ocp.example.com
192.168.122.32 etcd-1.ocp.example.com
192.168.122.33 etcd-2.ocp.example.com
192.168.122.34 worker-0.ocp.example.com
192.168.122.35 worker-1.ocp.example.com
EOF
```
#### 配置集群的服务记录及泛域名解析记录
配置如下：
```
# cat < /etc/dnsmasq.d/cluster.conf
srv-host=_etcd-server-ssl._tcp.ocp.example.com,etcd-0.ocp.example.com,2380,0,10
srv-host=_etcd-server-ssl._tcp.ocp.example.com,etcd-1.ocp.example.com,2380,0,10
srv-host=_etcd-server-ssl._tcp.ocp.example.com,etcd-2.ocp.example.com,2380,0,10
address=/apps.ocp.example.com/192.168.122.1
EOF
```
#### 启动系统的dnsmasq服务
启动命令如下：
```
# systemctl start dnsmasq
```
### 配置libvirt的缺省网络
配置方法如下：
```
# systemctl start libvirtd
# cat <<EOF > net-default.xml
<network>
  <name>default</name>
  <uuid>$(virsh net-uuid default | head -1)</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:6a:00:01'/>
  <dns>
    <forwarder addr='127.0.0.1'/>
  </dns>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.101' end='192.168.122.254'/>
      <host mac='52:54:00:6a:00:30' ip='192.168.122.30'/>
      <host mac='52:54:00:6a:00:31' ip='192.168.122.31'/>
      <host mac='52:54:00:6a:00:32' ip='192.168.122.32'/>
      <host mac='52:54:00:6a:00:33' ip='192.168.122.33'/>
      <host mac='52:54:00:6a:00:34' ip='192.168.122.34'/>
      <host mac='52:54:00:6a:00:35' ip='192.168.122.35'/>
    </dhcp>
  </ip>
</network>
EOF
# virsh net-define net-default.xml
# virsh net-destroy default
# virsh net-start default
```
注意：
- 因为低版本的libvirt暂不支持泛域名解析，所以配置内部DNS服务，使其将虚拟机的DNS请求发送到系统的DNS服务
- 配置MAC和IP绑定，实现固定的IP地址分配，使Openshift的各个虚拟机每次启动都可以获得固定的IP地址

### 配置Apache服务器
用来提供ignition文件供Openshift集群的各虚拟机初始化。
#### 编辑修改/etc/httpd/conf/httpd.conf文件
修改信息如下：
```
...
#Listen 12.34.56.78:80
Listen 81                                                
...
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
    ScriptAlias /openstack/latest/ "/var/www/cgi-bin/"  
...
```
说明：
- 修改apache的监听端口为81 ，避免和openshift的http服务冲突
- 增加script alias的路径供openshift节点启动时获取ignition文件

#### 创建CGI脚本
根据客户机的IP地址提供对应的ignition文件：
```
# cat << 'EOF' > /var/www/cgi-bin/user_data
#!/usr/bin/perl

$client = $ENV{"REMOTE_ADDR"};
$fname = "/var/www/html/ocp/$client.ign";

print "Content-Type:text/plain\r\n\r\n";

if (open(FH, "<$fname")) {
  while(read FH,$buffer,16384) {
    print $buffer;
  }
  close(FH);
} else {
  print "\n";
}

1;
EOF
# chmod 755 /var/www/cgi-bin/user_data
# systemctl start httpd
```
#### 创建iptables规则
将ignition文件的http请求重新定向到apache服务的真实地址:
```
# iptables -A PREROUTING -p tcp -d 169.254.169.254 \
--dport 80 -j DNAT --to 192.168.122.1:81 -t nat
```
### 配置NFS服务
目前主机上的Openshift仅支持NFS作为提供persistent volume的存储介质：
```
# mkdir /openshift
# cat <<EOF > /etc/exports
/openshift *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0)
EOF
# systemctl start nfs-server
# showmount -e
Export list for lpar07:
/openshift *
```
### 配置负载均衡服务
&#8195;&#8195;这里使用haproxy提供负载均衡服务，将API/HTTP/HTTPS等请求分发到对应节点。配置是依据 Openshift安装的官方文档需求，配置示例如下：
```
# cat <<EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats
    ssl-default-bind-ciphers PROFILE=SYSTEM
    ssl-default-server-ciphers PROFILE=SYSTEM

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    option                  forwardfor except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen  stats
    bind    *:8080
    mode        http
    log         global
    maxconn 10
    clitimeout      100s
    srvtimeout      100s
    contimeout      100s
    timeout queue   100s
    stats enable
    stats hide-version
    stats refresh 30s
    stats show-node
    stats uri  /stats

frontend ocp_api
    bind            *:6443
    mode tcp
    option tcplog
    default_backend ocp_api_servers

backend ocp_api_servers
    mode tcp
    balance source
    server          server0 192.168.122.30:6443 check
    server          server1 192.168.122.31:6443 check
    server          server2 192.168.122.32:6443 check
    server          server3 192.168.122.33:6443 check

frontend ocp_config
    bind            *:22623
    mode tcp
    option tcplog
    default_backend ocp_config_servers

backend ocp_config_servers
    mode tcp
    balance source
    server          server0 192.168.122.30:22623 check
    server          server1 192.168.122.31:22623 check
    server          server2 192.168.122.32:22623 check
    server          server3 192.168.122.33:22623 check

frontend ocp_http
    bind            *:80
    mode tcp
    option tcplog
    default_backend ocp_http_servers

backend ocp_http_servers
    mode tcp
    server          server4 192.168.122.34:80 check
    server          server5 192.168.122.35:80 check

frontend ocp_https
    bind            *:443
    mode tcp
    option tcplog
    default_backend ocp_https_servers

backend ocp_https_servers
    mode tcp
    server          server4 192.168.122.34:443 check
    server          server5 192.168.122.35:443 check
EOF
# systemctl start haproxy
```
具体可参考官方说明：[Networking requirements for user-provisioned infrastructure](https://docs.openshift.com/container-platform/4.2/installing/installing_ibm_z/installing-ibm-z.html#installation-network-user-infra_installing-ibm-z)

## 下载安装文件
### 下载openshift-install-linux
下载地址：[openshift-install-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/s390x/clients/ocp/latest/openshift-install-linux.tar.gz)         
下载并解压安装程序到/usr/local/bin：
```
# cd /usr/local/bin
# wget -O- https://mirror.openshift.com/pub/openshift-v4/s390x/clients/ocp/latest/openshift-install-linux.tar.gz | tar zxf -
```
### 下载openshift-client-linux
下载地址：[openshift-client-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/s390x/clients/ocp/latest/openshift-client-linux.tar.gz)       
下载并解压Openshift客户端命令行工具到/usr/local/bin:
```
# cd /usr/local/bin
# wget -O- https://mirror.openshift.com/pub/openshift-v4/s390x/clients/ocp/latest/openshift-client-linux.tar.gz | tar zxf -
```
### 下载OS磁盘镜像文件
下载地址：[rhcos-4.2.18-s390x-qemu.qcow2](https://mirror.openshift.com/pub/openshift-v4/s390x/dependencies/rhcos/4.2/latest/rhcos-4.2.18-s390x-qemu.qcow2)
```
# cd /root
# wget https://mirror.openshift.com/pub/openshift-v4/s390x/dependencies/rhcos/4.2/latest/rhcos-4.2.18-s390x-qemu.qcow2
# mv rhcos-4.2.18-s390x-qemu.qcow2 rhcos-4.2.18-s390x-qemu.qcow2.gz
```
说明：
- 下载可能需要使用代理服务器
- 下载的qcow2文件后缀名缺失，实际是压缩过的文件，需解压后使用

## 生成ignition文件
### 下载pull-secret.txt文件
&#8195;&#8195;Pull-Secret需要登录RedHat网站下载，该文件含有用来下载容器镜像的密钥，同时用来将OCP集群关联到Red Hat账户，下载地址：[Pull Secret](https://cloud.redhat.com/openshift/install/pull-secret?_ga=2.32547645.288546206.1606306636-1004033657.1601380096)

### 创建或使用已有的SSH密钥
&#8195;&#8195;密钥是用来SSH登录到OCP集群里面各节点获得Shell访问权限，可以使用现有的密钥，也可以生成一对新的密钥。以下命令可以用来生成一对无密码保护的新RSA密钥：
```
# ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
```
### 创建install-config.yaml文件
&#8195;&#8195;install-config.yaml文件是Openshift安装工具的配置文件，它会通过这个配置文件来生成集群的 ignition 文件。创建目录`/root/ocp`并将install-config.yaml文件创建在该目录下:
```
# mkdir /root/ocp
# cd /root/ocp
# vi install-config.yaml
apiVersion: v1
baseDomain: example.com
proxy:
  httpProxy: http://172.16.15.192:3128
  httpsProxy: http://172.16.15.192:3128
  noProxy: localhost,localhost.localdomain,.example.com,192.168.122.0/24
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
pullSecret: 'put your pull-secret here (a)'
sshKey: 'put your ssh public key here (b)'
```
说明：
- 在(a)处填入之前下载的pull-secret.txt的内容（仅一行）
- 在(b)处填入已有的SSH公钥或上一步生成的`/root/.ssh/id_rsa.pub`文件的内容（仅一行）

查看：
```
# ls /root/ocp
install-config.yaml
```
### 用安装工具生成ignition文件
&#8195;&#8195;ignition文件是用来初始化Openshift集群中各节点的配置文件。Openshift使用RedHat CoreOS作为运行容器的操作系统，系统安装的过程中，会自动寻找ignition文件并按里面的定义来完成系统的初始化及安装和配置Openshift环境。
#### 生成ignition文件
生成步骤如下：
```
# ls /root/ocp/install-config.yaml
/root/ocp/install-config.yaml
# openshift-install create manifests --dir=/root/ocp
# openshift-install create ignition-configs --dir=/root/ocp
```
命令运行后，install-config.yaml会被自动删除，同时会生成如下的文件：
```
/root/ocp
├── auth
│   ├── kubeadmin-password
│   └── kubeconfig
├── bootstrap.ign
├── master.ign
├── metadata.json
└── worker.ign
```
#### 修改文件权限
修改文件权限并为每个节点生成一个对应的拷贝:
```
# chmod 644 /root/ocp/*.ign
# mkdir /var/www/html/ocp
# cp /root/ocp/bootstrap.ign /var/www/html/ocp/192.168.122.30.ign
# cp /root/ocp/master.ign /var/www/html/ocp/192.168.122.31.ign
# cp /root/ocp/master.ign /var/www/html/ocp/192.168.122.32.ign
# cp /root/ocp/master.ign /var/www/html/ocp/192.168.122.33.ign
# cp /root/ocp/worker.ign /var/www/html/ocp/192.168.122.34.ign
# cp /root/ocp/worker.ign /var/www/html/ocp/192.168.122.35.ign
```
## 启动KVM虚拟机开始安装
### 准备磁盘镜像文件
为每个虚拟机准备一个磁盘镜像文件：
```
# cd /root
# gunzip rhcos-4.2.18-s390x-qemu.qcow2.gz
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/bootstrap.qcow2
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/master-0.qcow2
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/master-1.qcow2
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/master-2.qcow2
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/worker-0.qcow2
# cp rhcos-4.2.18-s390x-qemu.qcow2 /var/lib/libvirt/images/worker-1.qcow2
```
### 启动虚拟机开始安装
启动虚拟机开始安装:
```
# virt-install --name bootstrap --ram 16384 --vcpus 4 \
  --disk /var/lib/libvirt/images/bootstrap.qcow2 \
  --network mac=52:54:00:6A:00:30,network=default \
  --import --noautoconsole
# virt-install --name master-0 --ram 16384 --vcpus 4 \
  --disk /var/lib/libvirt/images/master-0.qcow2 \
  --network mac=52:54:00:6A:00:31,network=default \
  --import --noautoconsole
# virt-install --name master-1 --ram 16384 --vcpus 4 \
  --disk /var/lib/libvirt/images/master-1.qcow2 \
  --network mac=52:54:00:6A:00:32,network=default \
  --import --noautoconsole
# virt-install --name master-2 --ram 16384 --vcpus 4 \
  --disk /var/lib/libvirt/images/master-2.qcow2 \
  --network mac=52:54:00:6A:00:33,network=default \
  --import --noautoconsole
# virt-install --name worker-0 --ram 8192 --vcpus 4 \
  --disk /var/lib/libvirt/images/worker-0.qcow2 \
  --network mac=52:54:00:6A:00:34,network=default \
  --import --noautoconsole
# virt-install --name worker-1 --ram 8192 --vcpus 4 \
  --disk /var/lib/libvirt/images/worker-1.qcow2 \
  --network mac=52:54:00:6A:00:35,network=default \
  --import --noautoconsole
```
## 安装进度监控及后续配置
### 监控bootstrap进度
注意等待bootstrap完成后再继续下一步:
```
# openshift-install --dir=/root/ocp wait-for bootstrap-complete --log-level=info
```
### 监控集群访问状态
注意等待oc命令可以正常访问集群后再继续下一步：
```
# mkdir /root/.kube
# cp /root/ocp/auth/kubeconfig /root/.kube/config
# oc whoami
# oc get clusteroperators
```
### 配置持久卷供内部镜像源使用
步骤如下：
```
# cat <<EOF > config.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv4iro
spec:
  accessModes:
  - ReadWriteMany
  - ReadWriteOnce
  capacity:
    storage: 100Gi
  nfs:
    path: /openshift
    server: 192.168.122.1
  persistentVolumeReclaimPolicy: Recycle
  volumeMode: Filesystem
EOF
# oc create -f config.yaml
# oc patch configs.imageregistry.operator.openshift.io/cluster \
  --type merge --patch '{"spec":{"storage":{"pvc":{"claim":""}}}}'
```
### 监控集群安装进度并完成安装
监控集群安装进度并完成安装:
```
# openshift-install --dir=/root/ocp wait-for install-complete --log-level=info
```
### 确认集群运行状态
确认集群运行状态:
```
# oc get clusteroperators
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.2.20    True        False         False      60m
cloud-credential                           4.2.20    True        False         False      75m
cluster-autoscaler                         4.2.20    True        False         False      68m
console                                    4.2.20    True        False         False      65m
dns                                        4.2.20    True        False         False      74m
image-registry                             4.2.20    True        False         False      61m
ingress                                    4.2.20    True        False         False      69m
insights                                   4.2.20    True        False         False      75m
kube-apiserver                             4.2.20    True        False         False      73m
kube-controller-manager                    4.2.20    True        False         False      72m
kube-scheduler                             4.2.20    True        False         False      71m
machine-api                                4.2.20    True        False         False      75m
machine-config                             4.2.20    True        False         False      75m
marketplace                                4.2.20    True        False         False      69m
monitoring                                 4.2.20    True        False         False      66m
network                                    4.2.20    True        False         False      73m
node-tuning                                4.2.20    True        False         False      70m
openshift-apiserver                        4.2.20    True        False         False      71m
openshift-controller-manager               4.2.20    True        False         False      72m
openshift-samples                          4.2.20    True        False         False      70m
operator-lifecycle-manager                 4.2.20    True        False         False      73m
operator-lifecycle-manager-catalog         4.2.20    True        False         False      73m
operator-lifecycle-manager-packageserver   4.2.20    True        False         False      71m
service-ca                                 4.2.20    True        False         False      75m
service-catalog-apiserver                  4.2.20    True        False         False      71m
service-catalog-controller-manager         4.2.20    True        False         False      71m
storage                                    4.2.20    True        False         False      70m
```
确认集群运行正常（如上所示）后，可以尝试访问控制台:
- 访问网址：https://console-openshift-console.apps.ocp.example.com/
- 用户名：kubeadmin
- 密码：（密码可从`/root/ocp/auth/kubeadmin-password`文件中获取）

注意：因域名解析的问题，需要将宿主机作为浏览器的代理服务器才可以访问创建好的OCP环境
