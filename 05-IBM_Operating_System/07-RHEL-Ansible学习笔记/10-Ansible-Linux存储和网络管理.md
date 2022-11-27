# Ansible-Linux存储和网络管理
## 管理存储
### 使用Ansible模块配置存储
&#8195;&#8195;红帽Ansible自动化平台提供了一组模块，可用于在受管主机上配置存储设备。这些模块支持对设备进行分区，创建逻辑卷，以及创建和挂载文件系统。
#### parted模块
&#8195;&#8195;`parted`模块支持块设备的分区。该模块包含有`parted`命令的功能，允许创建具有特定大小、标志和对齐的分区。`parted`模块的一些参数如下表：

参数名称|描述
:---:|:---
align|配置分区对齐
device|块设备
flags|分区的标志
number|分区编号
part_end|以parted支持单位指定的从磁盘开头开始的分区大小
state|创建或删除分区
unit|分区信息的大小单位

示例创建了一个10GB的新分区：
```yml
- name: New 10GB partition
  parted:
    device: /dev/vdb # 使用vdb作为要分区的块设备
    number: 1 # 创建分区号1
    state: present # 确保该分区可用
    part_end: 10GB # 将分区大小设置为10GB
```
#### lvg和lvol模块
&#8195;&#8195;`lvg`和`lvol`模块支持创建逻辑卷，包括物理卷和卷组的配置。`lvg`将块设备取为参数，将其配置为卷组的后端物理卷。下表是`lvg`模块的一些参数：

参数名称|描述
:---:|:---
pesize|物理区块的大小。必须是2的幂，或128KiB的倍数
pvs|逗号分隔的设备列表，这些设备将配置为卷组的物理卷
vg|卷组的名称
state|创建或删除卷

示例使用块设备作为后端，创建具有特定物理区块大小的卷组：
```yml
- name: Creates a volume group
  lvg:
    vg: vg1 # 卷组名称是vg1
    pvs: /dev/vda1 # 使用/dev/vda1作为卷组的后端物理卷
    pesize: 32 # 将物理区块大小设置为32
```
&#8195;&#8195;下面示例中，如果`vg1`卷组已可用且将`/dev/vdb1`用作物理卷，则通过添加使用`/dev/vdc1`的一个新物理卷来扩大该卷：
```yml
- name: Resize a volume group
  lvg:
    vg: vg1
    pvs: /dev/vdb1,/dev/vdc1
```
&#8195;&#8195;`lvol`模块可创建逻辑卷，并且支持放大和收缩这些卷的大小，以及其上的文件系统。此模块还支持为逻辑卷创建快照。下表是`lvol`模块的一些参数：

参数名称|描述
:---:|:---
lv|逻辑卷的名称
resizefs|调整逻辑卷的文件系统大小
shrink|启用逻辑卷收缩
size|逻辑卷的大小
snapshot|逻辑卷快照的名称
state|创建或删除逻辑卷
vg|逻辑卷的父卷组

示例创建一个2GB的逻辑卷：
```yml
- name: Create a logical volume of 2GB
  lvol:
    vg: vg1 # 父卷组名称vg1
    lv: lv1 # 逻辑卷名称lv1
    size: 2g # 逻辑卷大小2GB
```
#### filesystem模块
&#8195;&#8195;`filesystem`模块支持创建文件系统和调整文件系统大小。此模块支持调整`ext2`、`ext3`、`ext4`、`ext4dev`、`f2fs`、`lvm`、`xfs`和`vfat`的文件系统大小。下表是`filesystem`模块的一些参数：

参数名称|描述
:---:|:---
dev|块设备名称
fstype|文件系统类型
resizefs|将文件系统大小增加到块设备大小

示例在分区中创建一个文件系统：
```yml
- name: Create an XFS filesystem
  filesystem:
    fstype: xfs # 使用XFS文件系统
    dev: /dev/vdb1 # 使用/dev/vdb1设备
```
#### mount模块
`mount`模块支持在`/etc/fstab`上配置挂载点。下表是`mount`模块的一些参数：

参数名称|描述
:---:|:---
fstype|文件系统类型
opts|挂载选项
path|挂载点路径
src|要挂载的设备
state|指定挂载状态。如果设为mounted，系统将挂载设备并使用该挂载信息配置/etc/fstab。要取消挂载设备并将它从/etc/fstab移除，可使用absent

示例使用特定ID挂载设备：
```yml
- name: Mount device with ID
  mount:
    path: /data # 使用/data作为挂载点路径
    src: UUID=a8063676-44dd-409a-b584-68be2c9f5570 # 使用此ID挂载设备
    fstype: xfs # 使用XFS文件系统
    state: present # 挂载设备并相应地配置/etc/fstab
```
示例将`172.25.250.100:/share`上可用的NFS 共享挂载到受管主机上的`/nfsshare`目录：
```yml
- name: Mount NFS share
  mount:
    path: /nfsshare
    src: 172.25.250.100:/share
    fstype: nfs
    opts: defaults
    dump: '0'
    passno: '0'
    state: mounted
```
#### 使用模块配置交换空间
&#8195;&#8195;红帽Ansible自动化平台目前不包含管理交换内存的模块。要通过Ansible使用逻辑卷向系统添加交换内存，步骤如下：
- 需要使用`lvg`和`lvol`模块创建新的卷组和逻辑卷
- 准备就绪后，需要使用`command`模块及`mkswap`命令将新逻辑卷格式化为交换空间
- 最后需要使用`command`模块及`swapon`命令激活新的交换设备。

&#8195;&#8195;红帽Ansible引擎包含了`ansible_swaptotal_mb`变量，其含有总交换内存大小。当交换内存不足时，可以使用此变量来触发交换配置和启用。以下任务为交换内存创建卷组和逻辑卷，将该逻辑卷格式化为交换，并将它激活：
```yml
- name: Create new swap VG
  lvg:
    vg: vgswap
    pvs: /dev/vda1
    state: present

- name: Create new swap LV
  lvol:
    vg: vgswap
    lv: lvswap
    size: 10g

- name: Format swap LV
  command: mkswap /dev/vgswap/lvswap
  when: ansible_swaptotal_mb < 128

- name: Activate swap LV
  command: swapon /dev/vgswap/lvswap
  when: ansible_swaptotal_mb < 128
```
### 存储配置的Ansible事实
&#8195;&#8195;Ansible使用事实向控制节点检索有关受管主机配置的信息。可以使用`setup` Ansible模块来检索受管主机的所有Ansible事实：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.100.134",
            "192.168.100.131"
        ],
    ...output omitted...
    },
    "changed": false
}
```
&#8195;&#8195;`setup`模块的`filter`选项支持根据shell样式的通配符进行精细过滤。`ansible_devices`元素包括受管主机上可用的所有存储设备。每个存储设备的元素包括分区或总大小等其他信息。示例显示受管主机的`ansible_devices`元素：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup -a 'filter=ansible_devices'
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_devices": {
            ...output omitted...
            "sr0": {
                "holders": [],
                "host": "SATA controller: VMware SATA AHCI controller",
                "links": {
                    "ids": [
                        "ata-VMware_Virtual_SATA_CDRW_Drive_01000000000000000001"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VMware SATA CD01",
                "partitions": {},
                "removable": "1",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "mq-deadline",
                "sectors": "2097151",
                "sectorsize": "512",
                "size": "1024.00 MB",
                "support_discard": "0",
                "vendor": "NECVMWar",
                "virtual": 1
            }
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
&#8195;&#8195;`ansible_device_links`元素包括各个存储设备的所有可用链接。示例显示受管主机的 `ansible_device_links`元素，该主机具有多个存储设备，其带有关联的ID：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup -a 'filter=ansible_device_links'
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_device_links": {
            "ids": {
                "dm-0": [
                    "dm-name-rhel-root",
                    "dm-uuid-LVM-SXS9jcQoZbu7xedMrnq1CMpgws2FHpb9loJnjfuQzExiMJCC18pox2Af8JKItLXi"
                ],
                ...output omitted...
                "nvme0n1": [
                    "nvme-nvme.15ad-564d57617265204e564d455f30303030-564d77617265205669727475616c204e564d65204469736b-00000001"
                ],
                ...output omitted...
                "sr0": [
                    "ata-VMware_Virtual_SATA_CDRW_Drive_01000000000000000001"
                ]
            },
            "labels": {},
            "masters": {
                "nvme0n1p2": [
                    "dm-0",
                    "dm-1"
                ],
                "nvme0n1p5": [
                    "dm-0"
                ]
            },
            "uuids": {
                "dm-0": [
                    "de085a1b-1289-450c-b62b-c95f7a0918d3"
                ],
                ...output omitted...
                "nvme0n2p5": [
                    "f3ad8520-7468-4ec3-938c-39a2fee46fef"
                ]
            }
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
&#8195;&#8195;`ansible_mounts`元素包括受管主机上当前挂载的设备的信息，如挂载的设备、挂载点和选项。以下输出显示受管主机的`ansible_mounts`元素，该主机具有多个活跃挂载：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup -a 'filter=ansible_mounts'
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_mounts": [
            {
                "block_available": 5911875,
                "block_size": 4096,
                "block_total": 9433600,
                "block_used": 3521725,
                "device": "/dev/mapper/rhel-root",
                "fstype": "xfs",
                "inode_available": 18693331,
                "inode_total": 18872320,
                "inode_used": 178989,
                "mount": "/",
                "options": "rw,seclabel,relatime,attr2,inode64,noquota",
                "size_available": 24215040000,
                "size_total": 38640025600,
                "uuid": "de085a1b-1289-450c-b62b-c95f7a0918d3"
            },
            ...output omitted...
        ],
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
## 管理网络配置
### 使用网络系统角色配置网络
&#8195;&#8195;RHEL8中包含一系列Ansible系统角色，可用于配置基于RHEL的系统。`rhel-system-roles`软件包可安装这些系统角色，这些角色可以支持时间同步或网络等配置。可以使用`ansible-galaxy list`命令列出当前安装的系统角色：
```
[ansible@redhat9 ~]$ ansible-galaxy list
# /usr/share/ansible/roles
- linux-system-roles.certificate, (unknown version)
- linux-system-roles.cockpit, (unknown version)
...output omitted...
- rhel-system-roles.vpn, (unknown version)
# /etc/ansible/roles
[WARNING]: - the configured path /home/ansible/.ansible/roles does not exist.
```
&#8195;&#8195;这些角色位于`~/usr/share/ansible/roles`目录中：以`linux-system-roles`开头的角色是相匹配的`rhel-system-roles`角色的符号链接。     
&#8195;&#8195;网络系统角色支持在受管主机上配置网络。此角色支持以太网接口、网桥接口、绑定接口、VLAN 接口、MacVLAN接口和Infiniband接口的配置。网络角色通过`network_provider`和`network_connections`这两个变量配置：
```yml
---
network_provider: nm
network_connections:
  - name: ens4
    type: ethernet
    ip:
      address:
        - 172.25.250.30/24
```
&#8195;&#8195;`network_provider`变量配置后端提供程序，即`nm`(NetworkManager)或`initscripts`。在RHEL 8中，网络角色使用`nm`(NetworkManager)`作为默认的网络提供程序。`initscripts`提供程序用于 RHEL 6系统，需要`network`服务处于可用状态。`network_connections`变量使用接口名称作为连接名称来配置指定为字典列表的不同连接。

下表是`network_connections`变量的选项：

选项名称|描述
:---:|:---
name|标识连接配置集
state|连接配置集的运行时状态。如果连接配置集为活跃，状态为up，否则为down
persistent_state|标识连接配置集是否持久。如果连接配置集持久，则为present，否则为absent
type|标识连接类型。有效的值有ethernet、bridge、bond、team、vlan、macvlan和infiniband
autoconnect|确定连接是否自动启动
mac|将连接限制为在具有特定MAC地址的设备上使用
interface_name|将连接配置集显示为供特定接口使用
zone|为接口配置FirewallD区域
ip|确定连接的IP配置。支持诸如address(指定静态IP地址)或 dns(配置DNS服务器)等选项

示例使用上述部分选项：
```yml
network_connections:
- name: eth0 # 使用eth0作为连接名称
   persistent_state: present # 使连接持久。这是默认值
   type: ethernet  # 将连接类型设置为ethernet
   autoconnect: yes  # 在引导时自动启动连接。这是默认值
   mac: 00:00:5e:00:53:5d  # 限制为具有此MAC地址的设备使用该连接
   ip:
     address:
       - 172.25.250.40/24  # 为连接配置172.25.250.40/24 IP地址
   zone: external  # 将external区域配置为连接的FirewallD区域
```
使用网络系统角色，需要在Playbook的roles子句下指定角色名称，如下所示：
```yml
- name: NIC Configuration
  hosts: webservers
  vars:
    network_connections:
      - name: ens4
        type: ethernet
        ip:
          address:
            - 172.25.250.30/24
  roles:
    - rhel-system-roles.network
```
&#8195;&#8195;可以使用`vars`子句指定网络角色的变量，如上例中所示，也可在`group_vars`或`host_vars`目录下创建包含这些变量的YAML文件，具体取决于使用情形。
### 使用模块配置网络
&#8195;&#8195;作为`network`系统角色的替代选择，Ansible包含了支持系统上网络配置的一系列模块。`nmcli`模块支持管理网络连接和设备。此模块支持配置网络接口组合和绑定，以及IPv4和IPv6寻址。下表是`nmcli`模块的一些参数：

参数名称|描述
:---:|:---
conn_name|配置连接名称
autoconnect|启用在引导时自动激活连接
dns4|配置IPv4的DNS服务器(最多3个)
gw4|为接口配置IPv4网关
ifname|要绑定到连接的接口
ip4|接口的IP地址(IPv4)
state|启用或禁用该网络接口
type|设备或网络连接的类型

示例为网络连接和设备配置静态IP配置：
```yml
- name: NIC configuration
  nmcli:
    conn_name: ens4-conn # 配置ens4-conn作为连接名称
    ifname: ens4 # 将ens4-conn连接绑定到ens4网络接口
    type: ethernet # 将网络接口配置为ethernet
    ip4: 172.25.250.30/24 # 在接口上配置172.25.250.30/24 IP地址
    gw4: 172.25.250.1 # 将网关设置为172.25.250.1
    state: present # 确保连接可用
```
&#8195;&#8195;`hostname`模块可在不修改`/etc/hosts`文件的前提下设置受管主机的主机名。此模块使用 `name`参数来指定新的主机名，如下所示：
```yml
- name: Change hostname
  hostname:
    name: managedhost1
```
#### firewalld模块
&#8195;&#8195;`firewalld`模块支持受管主机上的`FirewallD`管理。此模块支持为服务和端口配置`FirewallD`规则。它还支持区域管理，包括将网络接口和规则与特定区域关联。示例为默认区域(public)中的`http`服务创建`FirewallD`规则。该任务将规则配置为永久规则，并确保它处于活动状态：
```yml
- name: Enabling http rule
  firewalld:
    service: http
    permanent: yes
    state: enabled
```
示例在`external FirewallD`区域中配置eth0：
```yml
- name: Moving eth0 to external
  firewalld:
    zone: external
    interface: eth0
    permanent: yes
    state: enabled
```
下表是`firewalld`模块的一些参数：

参数名称|描述
:---:|:---
interface|要使用FirewallD管理的接口名称
port|端口或端口范围。使用端口/协议或端口-端口/协议格式
rich_rule|FirewallD的富规则
service|要使用FirewallD管理的服务名称
source|要使用FirewallD管理的来源网络
zone|FirewallD区域
state|启用或禁用FirewallD配置
type|设备或网络连接的类型

### 网络配置的Ansible事实
&#8195;&#8195;Ansible使用事实向控制节点检索有关受管主机配置的信息。可以使用`setup` Ansible模块来检索受管主机的所有Ansible事实。示例参考之前的。    
&#8195;&#8195;受管主机的所有网络接口位于`ansible_interfaces`元素下。可以使用`setup`模块的`gather_subset=network`参数，将事实限制为`network`子集中包含的事实。`setup`模块的`filter`选项支持根据`shell`样式的通配符进行精细过滤：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup \
> -a 'gather_subset=network filter=ansible_interfaces'
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_interfaces": [
            "lo",
            "ens160"
        ],
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
&#8195;&#8195;上面示例显示了受管主机`redhat8`上可用的两个网络接口：`lo`、和`ens160`。可以使用`setup`模块的`ansible_NIC_name`过滤器检索有关网络接口配置的其他信息。例如，要检索置`ens160`网络接口的配置，可使用`ansible_ens160`过滤器：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup \
> -a 'gather_subset=network filter=ansible_ens160'
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_ens160": {
            "active": true,
            "device": "ens160",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                ...output omitted...
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "192.168.100.134",
                "broadcast": "192.168.100.255",
                "netmask": "255.255.255.0",
                "network": "192.168.100.0"
            },
            "ipv4_secondaries": [
                {
                    "address": "192.168.100.131",
                    "broadcast": "192.168.100.255",
                    "netmask": "255.255.255.0",
                    "network": "192.168.100.0"
                }
            ],
            "ipv6": [
                {
                    "address": "fe80::bcd7:31a:c4ac:9c09",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "00:0c:29:ee:ed:0e",
            "module": "vmxnet3",
            "mtu": 1500,
            "pciid": "0000:03:00.0",
            "promisc": false,
            "speed": 10000,
            "timestamping": [
                "rx_software",
                "software"
            ],
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
&#8195;&#8195;上面示例显示其他配置详细信息，如IPv4和IPv6的IP地址配置、关联的设备及其类型。下表是`network`子集的一些可用事实：

事实名称|描述
:---:|:---
ansible_dns|包括DNS服务器IP地址和搜索域
ansible_domain|包括受管主机的子域
ansible_all_ipv4_addresses|包括受管主机上配置的所有IPv4地址
ansible_all_ipv6_addresses|包括受管主机上配置的所有IPv6地址
ansible_fqdn|包括受管主机的FQDN
ansible_hostname|包括非限定主机名，即FQDN中第一个句点之前的字符串
ansible_nodename|包括系统报告的受管主机的主机名

注意：
- Ansible还提供`inventory_hostname`变量，它可包含Ansible清单文件中配置的主机名

## 练习