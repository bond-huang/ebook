# LinuxONE-OpenStack on KVM安装手册
学习IBM官方云最佳实践视频做的笔记，步骤基本摘自官方手册，手册地址：[LinuxONE高密度云最佳实践成长之路 (KVM版）](https://csc.cn.ibm.com/roadmap/index/e96159c6-cf9b-47cb-bb13-17cb5cecdaf7?eventId=)

## 部署环境
&#8195;&#8195;本文档部署OpenStack Train 版本，基于LinuxONE的硬件环境，每个Lpar有2个网卡，一个网卡为管理网，为OpenStack提供api交互，另外一块网卡为虚拟机业务网，次网卡不需要配置ip地址，网卡属性需要设置为bridge_role=primary。本次yum源使用rdo提供的源。OpenStack平台内部的用户比较多，为了方便读者部署，所有OpenStack的用户的密码均设置为openstack。

hostname|IP|OS|kernal|Hardware|Nic
:---:|:---:|:---:|:---:|:---:|:---:
controller|172.16.36.177|RHEL release8.0|4.18.0-80.el8.s390x|2c4G50G|管理enc3000业务enc3100
compute|172.16.36.176|RHEL release8.0|4.18.0-80.el8.s390x|2c4G50G|管理enc3000业务enc3100

下图为部署的基本架构图：    
![部署基本架构图](OpenStack-3.png)

## 前期准备
### 配置YUM源
配置参考及步骤如下：
```
[root@controller ~]# cat /etc/yum.repos.d/openstack.repo
[rdo-train-upstream]
name=rdo-train-upstream
baseurl=https://trunk.rdoproject.org/centos8-train/puppet-passed-ci/
enabled=1
gpgcheck=0
[rdo-train-linuxone-deps]
name=rdo-train-linuxone-deps
baseurl=http://linuxone.cloud.marist.edu:8080/repos/rdo/rhel8.0/deps/
enabled=1
gpgcheck=0
[root@controller ~]#
```
### 关闭防火墙和selinux
配置参考及步骤如下：
```
[root@controller ~]# systemctl stop firewalld
[root@controller ~]# setenforce 0
setenforce: SELinux is disabled
[root@controller ~]# sed -ri '/^[^#]*SELINUX=/s#=.+$#=disabled#'  /etc/selinux/config
```
### 安装/配置 ntp 服务
在controller上配置配置ntp server：
```
[root@controller ~]# yum install chrony -y
[root@controller ~]# vim /etc/chrony.conf
allow 172.16.36.0/24
[root@controller ~]# systemctl restart chronyd.service &&  systemctl enable chronyd.service
```
在compute节点配置ntp server为controller节点的ip地址:
```
[root@compute ~]# vim /etc/chrony.conf
server 172.16.36.177 iburst
```
重启服务:
```
[root@compute ~]#  systemctl restart chronyd.service &&  systemctl enable chronyd.service
```
配置hosts 解析:
```
[root@controller ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.36.177 controller
172.16.36.176 compute
[root@controller ~]#
[root@compute ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.36.177 controller
172.16.36.176 compute
[root@compute ~]#
```
### 安装数据库
OpenStack 使用Mariadb数据库存储内部数据，此步骤只需要在controller节点执行:
```
[root@controller ~]# yum install mariadb mariadb-server python2-PyMySQL -y
```
添加配置文件:
```
[root@controller ~]# cat /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 172.16.36.177
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
启动服务:
```
[root@controller ~]# systemctl enable mariadb.service && systemctl start mariadb.service
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[root@controller ~]#
```
### 安装rabbitmq 服务
Rabbitmq 服务为OpenStack提供了消息队列服务，用于各模块间的异步调用，此步骤只需要在controller节点执行:
```
[root@controller ~]# yum install rabbitmq-server -y
```
启动服务:
```
[root@controller ~]# systemctl enable rabbitmq-server.service && systemctl start rabbitmq-server.service
[root@controller ~]#
```
为rabbitmq添加用户，并赋予权限:
```
[root@controller ~]# rabbitmqctl add_user openstack openstack
warning: the VM is running with native name encoding of latin1 which may cause Elixir to malfunction as it expects utf8. Please ensure your locale is set to UTF-8 (which can be verified by running "locale" in your shell)
Adding user "openstack" ...
[root@controller ~]#
[root@controller ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
warning: the VM is running with native name encoding of latin1 which may cause Elixir to malfunction as it expects utf8. Please ensure your locale is set to UTF-8 (which can be verified by running "locale" in your shell)
Setting permissions for user "openstack" in vhost "/" ...
[root@controller ~]#
```
### 安装memcached 服务
Memcache 服务为OpenStack 提供缓存服务，此步骤只需要在controller节点执行:
```
[root@controller ~]# yum install memcached python-memcached -y
```
编辑配置文件:
```
[root@controller ~]# cat /etc/sysconfig/memcached
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1,controller"
[root@controller ~]#
```
启动服务，并配置开机自启:
```
[root@controller ~]# systemctl enable memcached.service && systemctl start memcached.service
```
### 安装etcd服务
安装：
```
[root@controller ~]# yum install memcached python-memcached -y
```
编辑配置文件:
```
[root@controller ~]# cat /etc/sysconfig/memcached
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1,controller"
[root@controller ~]#
```
启动服务，并配置开机自启:
```
[root@controller ~]# systemctl enable memcached.service && systemctl start memcached.service
```
跟刚才一模一样，回头研究下是否有误。
### 安装OpenStack客户端
安装命令如下：
```
[root@controller ~]# yum install python3-openstackclient -y
```
## 安装keystone 认证服务
Keystone 为OpenStack提供认证服务，此组件安装到controller节点。
### 创建keystone数据库
配置参考及步骤如下：
```
[root@controller ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.3.17-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> CREATE DATABASE keystone;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost'  IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%'  IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> \q
Bye
[root@controller ~]#
```
### 安装并配置keystone服务
安装命令：
```
[root@controller ~]# yum install openstack-keystone httpd mod_wsgi -y
```
修改配置：
```
[root@controller ~]# vim /etc/keystone/keystone.conf
...
[database]
...
connection = mysql+pymysql://keystone:openstack@controller/keystone
...
[token]
provider = fernet
```
同步数据库:
```
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone
```
生成fernet key:
```
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```
创建用户:
```
[root@controller ~]# keystone-manage bootstrap --bootstrap-password openstack  \
--bootstrap-admin-url http://controller:5000/v3/ \
--bootstrap-internal-url http://controller:5000/v3/ \
--bootstrap-public-url http://controller:5000/v3/ \
--bootstrap-region-id RegionOne
```
修改httpd配置文件:
```
[root@controller ~]# vim /etc/httpd/conf/httpd.conf
ServerName controller
```
创建软连接:
```
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```
启动服务:
```
[root@controller ~]# systemctl enable httpd.service && systemctl start httpd.service
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```
创建认证的配置文件，里面包含着认证信息，当我们需要操作OpenStack内部资源时，需要执行下这个文件:
```
[root@controller ~]# cat admin.rc
export OS_USERNAME=admin
export OS_PASSWORD=openstack
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```
执行环境变量，做OpenStack的认证信息:
```
[root@controller ~]# source  admin.rc
```
创建user 的role，以后如果做多租户会需要用到:
```
[root@controller ~]# openstack role create user
```
### 验证keystone服务是否可用
验证示例如下,若有类似的输出则证明安装成功：
```
[root@controller ~]# openstack user list
+----------------------------------+--------+
| ID                               | Name   |
+----------------------------------+--------+
| 45085728834a40e489d74b444da4a3e8 | admin  |
+----------------------------------+--------+
```
## 安装glance 服务
