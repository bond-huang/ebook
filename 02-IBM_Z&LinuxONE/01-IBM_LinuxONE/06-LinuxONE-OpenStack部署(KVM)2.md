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
Neutron 服务为OpenStack提供SDN 服务，为虚拟机提供虚拟网络。
### 控制节点执行
#### 创建neutron数据库
配置参考及步骤如下：
```
[root@controller ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 101
Server version: 10.3.17-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>  CREATE DATABASE neutron;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.001 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%'    IDENTIFIED BY 'openstack';
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]>\q
```
#### 用户创建
创建neutron用户并赋予admin权限：
```
[root@controller ~]# openstack user create --domain default --password-prompt neutron
User Password: openstack
Repeat User Password: openstack
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 94ac1191b9a84ef581fa1ac65e2b40a0 |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
[root@controller ~]# openstack role add --project service --user neutron admin
```
创建neutron service:
```
[root@controller ~]# openstack service create --name neutron \
--description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 5fd0d75fcd0f470fa26202dbb34dc0e2 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```
创建endpoint:
```
[root@controller ~]# openstack endpoint create --region RegionOne \
network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 700d0ba978654bdc94374da2dcca9554 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5fd0d75fcd0f470fa26202dbb34dc0e2 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
[root@controller ~]# openstack endpoint create --region RegionOne \
network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | f1e236c9fe7d41bea80235aade1f6c51 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5fd0d75fcd0f470fa26202dbb34dc0e2 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
[root@controller ~]# openstack endpoint create --region RegionOne \
network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 96610eaf1ad14d828035b91457b06f47 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5fd0d75fcd0f470fa26202dbb34dc0e2 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
[root@controller ~]#
```
安装neutron相关包:
```
[root@controller ~]# yum install openstack-neutron-ml2 \
openstack-neutron-openvswitch openstack-neutron python3-neutronclient -y
```
修改配置文件:
```
[root@controller ~]# vim /etc/neutron/neutron.conf
[DEFAULT]
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
core_plugin = ml2
auth_strategy = keystone
transport_url =  rabbit://openstack:openstack@controller
bind_host=172.16.36.177
[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = openstack
[database]
connection = mysql+pymysql://neutron:openstack@controller/neutron
[nova]
# ...
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = openstack
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

vim /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = local,flat,vlan
tenant_network_types = flat,vlan
mechanism_drivers =openvswitch
[ml2_type_flat]
flat_networks =*
[ml2_type_vlan]
network_vlan_ranges =vlan
[ml2_type_gre]
[ml2_type_vxlan]
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
[l2pop]
[database]
[agent]
l2_population=False
polling_interval=2
arp_responder=False
[ovs]
enable_tunneling=False
integration_bridge=br-int
bridge_mappings=vlan:br-ex

[root@controller ~]# cat  /etc/neutron/plugins/ml2/openvswitch_agent.ini
[ml2]
type_drivers = local,flat,vlan
tenant_network_types = flat,vlan
mechanism_drivers =openvswitch
[ml2_type_flat]
flat_networks =*
[ml2_type_vlan]
network_vlan_ranges =vlan
[ml2_type_gre]
[ml2_type_vxlan]
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
[l2pop]
[database]
[agent]
l2_population=False
polling_interval=2
arp_responder=False
[ovs]
enable_tunneling=False
integration_bridge=br-int
bridge_mappings=vlan:br-ex
[root@controller ~]# vim /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = openstack

[root@controller ~]# vim /etc/nova/nova.conf
[neutron]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = openstack
service_metadata_proxy = true
metadata_proxy_shared_secret =openstack
```
创建软连接:
```
[root@controller~]#ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
同步数据库:
```
[root@controller ~]# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
```
正在对 neutron 运行 upgrade...
```INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> kilo
INFO  [alembic.runtime.migration] Running upgrade kilo -> 354db87e3225
INFO  [alembic.runtime.migration] Running upgrade 354db87e3225 -> 599c6a226151
INFO  [alembic.runtime.migration] Running upgrade b12a3ef66e62 -> 97c25b0d2353
INFO  [alembic.runtime.migration] Running upgrade 97c25b0d2353 -> 2e0d7a8a1586
INFO  [alembic.runtime.migration] Running upgrade 2e0d7a8a1586 -> 5c85685d616d
```
#### 网络配置
创建网桥：
```
[root@controller ~]# systemctl start openvswitch
[root@controller ~]# systemctl enable openvswitch
[root@controller ~]# ovs-vsctl add-br br-ex
[root@controller ~]# ovs-vsctl add-port br-ex enc3100   #此网卡为业务网卡
[root@controller ~]# ovs-vsctl show
f8a27ca8-e2f9-4b9a-9208-a25efd0b8fff
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "enc3100"
            Interface "enc3100"
    ovs_version: "2.11.0"
[root@controller ~]#
```
配置网卡为bridge_role，若不配置虚拟机网络会不通:
```
[root@controller ~]#chzdev  0.0.3100 bridge_role=primary
[root@controller ~]# chzdev -e 0.0.3100
```
#### 修改配置文件
修改系统的配置文件：
```
[root@controller ~]# vim /etc/sysctl.conf
net.ipv4.tcp_keepalive_intvl = 1
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_time = 5
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.ip_forward = 1
[root@controller ~]# sysctl  -p
```
#### 启动服务
启动步骤如下：
```
[root@controller ~]# systemctl start  neutron-server.service  neutron-openvswitch-agent.service   neutron-metadata-agent.service neutron-openvswitch-agent.service
[root@controller ~]# systemctl enable   neutron-server.service  neutron-openvswitch-agent.service   neutron-metadata-agent.service neutron-openvswitch-agent.service
Created symlink /etc/systemd/system/multi-user.target.wants/neutron-server.service → /usr/lib/systemd/system/neutron-server.service.
Created symlink /etc/systemd/system/multi-user.target.wants/neutron-openvswitch-agent.service → /usr/lib/systemd/system/neutron-openvswitch-agent.service.
Created symlink /etc/systemd/system/multi-user.target.wants/neutron-metadata-agent.service → /usr/lib/systemd/system/neutron-metadata-agent.service.
[root@controller ~]#
```
### 在计算节点执行
安装neutron-openvswitch 服务:
```
[root@compute ~]# yum install openstack-neutron-ml2 openstack-neutron-openvswitch ebtables ipset -y
```
改配置文件，可以复用控制节点的配置文件:
```
[root@compute ml2]# pwd
/etc/neutron/plugins/ml2
[root@compute neutron]# scp controller:/etc/neutron/neutron.conf .
The authenticity of host 'controller (172.16.36.177)' can't be established.
ECDSA key fingerprint is SHA256:aZZ6eV1fv07ZBk1A3qjv/dlmGeSIsmWThYlebsRIy/A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'controller' (ECDSA) to the list of known hosts.
root@controller's password:
neutron.conf                                              100%   39KB   1.9MB/s   00:00
```
把ip改成本节点的ip地址:
```
[root@compute neutron]# vim neutron.conf
bind_host=172.16.36.176
[root@compute ml2]# scp controller:/etc/neutron/plugins/ml2/ml2_conf.ini .
[root@compute ml2]# scp controller:/etc/neutron/plugins/ml2/openvswitch_agent.ini .
```
修改nova.conf:
```
[root@compute ml2]# vim /etc/nova/nova.conf
[neutron]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = openstack
```
创建网桥:
```
[root@compute ml2]# systemctl start openvswitch
[root@compute ml2]# ovs-vsctl add-br br-ex
[root@compute ml2]# ovs-vsctl add-port br-ex enc3100
```
配置网卡为bridge_role:
```
[root@controller ~]# chzdev 0.0.3100 bridge_role=primary
[root@controller ~]# chzdev -e 0.0.3100
```
启动服务:
```
[root@compute ml2]# systemctl enable neutron-openvswitch-agent.service openvswitch
[root@compute ml2]# systemctl restart  neutron-openvswitch-agent.service  openstack-nova-compute.service  openstack-nova-api
[root@compute ml2]#
```
### 验证服务
若是笑脸则证明服务正常：
```
[root@controller ml2]# neutron agent-list
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| 1816c0ed-c633-45f4-b7b9-12a8a553aabc | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-agent    |
| 1ca845c3-85d6-44cc-9a8e-10bedd1828ea | Open vSwitch agent | controller |                   | :-)   | True           | neutron-openvswitch-agent |
| e8bd8275-db99-48f2-9af9-0c659ba83d9f | Open vSwitch agent | compute    |                   | :-)   | True           | neutron-openvswitch-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
[root@controller ml2]#
```
## 安装dashboard
dashboard 是为OpenStack提供web管理界面，更方便的管理Openstack，降低了使用成本。
### 安装配置
安装命令如下：
```
[root@controller ~]# yum install openstack-dashboard -y
```
修改配置文件:
```
[root@controller ~]# vim /etc/openstack-dashboard/local_settings
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*', ]
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
  'identity': 3,
   "image": 2,
   "volume": 3,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    'enable_distributed_router': False,
    'enable_firewall': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_quotas': True,
    'enable_security_group': True,
    'enable_vpn': False,
    'profile_support': None,
}
TIME_ZONE = "UTC"
[root@controller ~]# vim /etc/httpd/conf.d/openstack-dashboard.conf
WSGIApplicationGroup %{GLOBAL}
```
重启服务:
```
[root@controller ~]# systemctl restart httpd.service memcached.service
```
&#8195;&#8195;此时使用浏览器访问，在浏览器中访问`http://ip/dashboard`若此时正常访问则证明搭建成功，若访问不到，则需要在`/etc/openstack-dashboard/local_settings`最后添加:
```
WEBROOT = '/dashboard/'
LOGIN_URL = '/dashboard/auth/login/'
LOGOUT_URL = '/dashboard/auth/logout/'
LOGIN_REDIRECT_URL = '/dashboard/'
```
添加完后，重启httpd和memcached服务再重试使用web登录。
