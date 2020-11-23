# LinuxONE-OpenStack on KVM安装手册2
学习IBM官方云最佳实践视频做的笔记，步骤基本摘自官方手册，手册地址：[LinuxONE高密度云最佳实践成长之路 (KVM版）](https://csc.cn.ibm.com/roadmap/index/e96159c6-cf9b-47cb-bb13-17cb5cecdaf7?eventId=)
## 安装nova服务
Nova 是OpenStack 的计算模块，旨在于管理每个compute节点的虚拟机，比如虚拟机的start/stop/resize等。
### 控制节点执行
#### 创建相关数据库
配置参考及步骤如下：
```
[root@controller ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 29
Server version: 10.3.17-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> CREATE DATABASE nova_api;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> CREATE DATABASE nova;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> CREATE DATABASE nova_cell0;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
    ->   IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> \q
Bye
```
#### 在OpenStack中添加nova用户
创建步骤如下：
```
[root@controller ~]#  openstack user create --domain default --password-prompt nova
User Password: openstack
Repeat User Password: openstack
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | f2e10eac7b4e4ef88b074e7669605609 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
为nova用户添加admin角色:
```
[root@controller ~]# openstack role add --project service --user nova admin
```
添加nova service
```
[root@controller ~]# openstack service create --name nova \
--description "OpenStack Compute" compute

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | dd4e9da63e774665958fa409efa65f30 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```
创建endpoint:
```
[root@controller ~]# openstack endpoint create --region RegionOne \
compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 53a71290cc214a06a6665d081cfa0782 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | dd4e9da63e774665958fa409efa65f30 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
[root@controller ~]#  openstack endpoint create --region RegionOne \
compute internal http://controller:8774/v2.1

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | af529989702a44249f581d2454d91219 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | dd4e9da63e774665958fa409efa65f30 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
[root@controller ~]#
[root@controller ~]# openstack endpoint create --region RegionOne \
compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9c2ac43f49e74b56aa1042291668044f |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | dd4e9da63e774665958fa409efa65f30 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```
#### 安装nova相关包
命令如下：
```
[root@controller ~]# yum install openstack-nova-api openstack-nova-conductor \
openstack-nova-novncproxy openstack-nova-scheduler -y
```
#### 修改配置文件
参考如下：
```
[root@controller ~]# vim /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:openstack@controller:5672
my_ip = 172.16.36.177
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[database]
connection = mysql+pymysql://nova:openstack@controller/nova
[api_database]
connection = mysql+pymysql://nova:openstack@controller/nova_api

[api]
auth_strategy = keystone
[keystone_authtoken]
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = openstack
[vnc]
enabled = false
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = openstack
```
#### 同步及启动
同步数据库：
```
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
29f46528-001d-48c4-a243-717d449581ba
[root@controller ~]# su -s /bin/sh -c "nova-manage db sync" nova
	/usr/lib/python3.6/site-packages/pymysql/cursors.py:165: Warning: (1831, 'Duplicate index `block_device_mapping_instance_uuid_virtual_name_device_name_idx`. This is deprecated and will be disallowed in a future release')
  result = self._query(query)
/usr/lib/python3.6/site-packages/pymysql/cursors.py:165: Warning: (1831, 'Duplicate index `uniq_instances0uuid`. This is deprecated and will be disallowed in a future release')
  result = self._query(query)
[root@controller ~]#  su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
|  名称 |                 UUID                 |              Transport URL               |                    数据库连接                   | Disabled |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                  none:/                  | mysql+pymysql://nova:****@controller/nova_cell0 |  False   |
| cell1 | 29f46528-001d-48c4-a243-717d449581ba | rabbit://openstack:****@controller:5672/ |    mysql+pymysql://nova:****@controller/nova    |  False   |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
[root@controller ~]#
```
启动服务:
```
[root@controller ~]# systemctl enable \
openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
[root@controller ~]# systemctl start \
openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
[root@controller ~]#
```
### 计算节点执行
#### 安装及配置
安装nova-compute 包：
```
[root@compute ~]# dnf module disable virt
[root@compute ~]# yum install openstack-nova-compute -y
```
修改配置文件:
```
[root@compute ~]# vim /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:openstack@controller:5672
my_ip =172.16.36.176
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
compute_driver = libvirt.LibvirtDriver
config_drive_format = iso9660
force_config_drive = True
flat_injected=true
[api]
auth_strategy = keystone
[keystone_authtoken]
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = openstack

[vnc]
enabled=false
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = openstack

[libvirt]
virt_type = kvm
cpu_mode = none
use_usb_tablet = False
inject_partition = -2
```
启动服务:
```
[root@compute ~]# systemctl restart libvirtd.service openstack-nova-compute.service \
&& systemctl enable  libvirtd.service openstack-nova-compute.service
```
### 验证nova模块是否正常
列出的服务都是up即为正常:
```
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
|  5 | nova-compute | compute | nova | enabled | up    | 2020-04-21T09:10:26.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 29f46528-001d-48c4-a243-717d449581ba
Checking host mapping for compute host 'compute': 68fd1fa7-5074-48c6-8f6a-d6afd490dadc
Creating host mapping for compute host 'compute': 68fd1fa7-5074-48c6-8f6a-d6afd490dadc
Found 1 unmapped computes in cell: 29f46528-001d-48c4-a243-717d449581ba
[root@controller ~]#

[root@controller ~]# nova service-list
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| Id                                   | Binary         | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason | Forced down |
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| a1d58917-55eb-4936-b59f-f7a01e5062cb | nova-conductor | controller | internal | enabled | up    | 2020-04-21T09:11:28.000000 | -               | False       |
| 70577e8a-9352-4304-a322-570903d9c70f | nova-scheduler | controller | internal | enabled | up    | 2020-04-21T09:11:29.000000 | -               | False       |
| 98931cb3-19ee-487e-9e1f-45872c519749 | nova-compute   | compute    | nova     | enabled | up    | 2020-04-21T09:11:36.000000 | -               | False       |
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
[root@controller ~]#
```
## 安装neutron服务

