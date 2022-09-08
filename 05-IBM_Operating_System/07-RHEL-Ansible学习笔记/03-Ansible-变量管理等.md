# Ansible-变量&机密&事实管理
## 管理变量
### Ansible变量
&#8195;&#8195;Ansible支持利用变量来存储值，并在Ansible项目的所有文件中重复使用这些值。这可以简化项目的创建和维护，并减少错误的数量。通过变量，可以轻松地在Ansible项目中管理给定环境的动态值。变量可能包含下面这些值：
- 要创建的用户
- 要安装的软件包
- 要重新启动的服务
- 要删除的文件
- 要从互联网检索的存档

### 命名变量
变量的名称必须以字母开头，并且只能含有字母、数字和下划线。示例如下表：

无效的变量名称|有效的变量名称
:---|:---
db2 server|db2_server
remote.file|remote_file
1st file|file_1 or file1
remoteserver$1|remote_server_1 or remote_server1

### 定义变量
&#8195;&#8195;可在Ansible项目中的多个位置定义变量。如果在两个位置设置了同名变量，并且变量值不同，则通过优先级来决定要使用哪个值。
- 可以设置会影响一组主机的变量，也可以设置只会影响个别主机的变量
- 有些变量是Ansible可以根据系统配置来设置的事实
- 有些变量则可在`playbook`中设置，然后影响该`playbook`中的一个`play`，或者仅影响该`play`中的一项任务
- 可通过`--extra-vars`或`-e`选项并指定变量值来在`ansible-playbook`命令行上设置额外变量 ，这些值将覆盖相应变量名称的所有其他值

定义变量的方法如下，按优先级从低到高排列：
- 在清单中定义的组变量
- 在清单或`Playbook`所在目录的`group_vars`子目录中定义的组变量
- 在清单中定义的主机变量
- 在清单或`Playbook`所在目录的`host_vars`子目录中定义的主机变量
- 在运行时中发现的主机事实
- (`vars`和`vars_files`)`playbook`中的`Play`变量
- 任务变量
- 在命令行中定义的额外变量

&#8195;&#8195;建议选择全局唯一的变量名称，以避免考虑优先级规则。如果在多个级别上定义了相同名称的变量，则采用优先级别最高的变量。更详细、更精准的变量优先级说明，见Ansible文档：[Variable precedence: Where should I put a variable?](https://docs.ansible.com/ansible/2.9/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
### Playbook中的变量
#### Playbook中定义变量
&#8195;&#8195;编写`playbook`时，可以定义自己的变量，然后在任务中调用这些值。`Playbook`变量可以通过多种方式定义。一种常见方式是将变量放在`playbook`开头的`vars`块中：
```yaml
- hosts: all
  vars:
    user: huang
    home: /home/huang
```
&#8195;&#8195;也可以在外部文件中定义`playbook`变量，使用`vars_files`指令，后面跟上相对于`playbook`位置的外部变量文件名称列表：
```yaml
- hosts: all
  vars_files:
    - vars/users.yml
```
可以使用YAML格式在这些文件中定义`playbook`变量，示例：
```yaml
user: huang
home: /home/huang
```
#### 在Playbook中使用变量
&#8195;&#8195;声明变量后，若需要使用，可以将变量名称放在双花括号`{{ }}`内。在任务执行时，Ansible会将变量替换为其值。示例：
```yaml
vars:
  user: huang

tasks:
  # This line will read: Creates the user joe
  - name: Creates the user {{ user }}
    user:
      # This line will create the user named Joe
      name: "{{ user }}"
```
&#8195;&#8195;当变量用作开始一个值的第一元素时，必须使用引号。这可以防止Ansible将变量引用视为YMAL 字典的开头。如果缺少引号，将显示下列消息：
```yaml
yum:
     name: {{ service }}
            ^ here
We could be wrong, but this one looks like it might be an issue with
missing quotes.  Always quote template expression brackets when they
start a value. For instance:

    with_items:
      - {{ foo }}

Should be written as:

    with_items:
      - "{{ foo }}"
```
### 主机变量和组变量
直接应用于主机的清单变量分为两大类：
- 主机变量，应用于特定主机
- 组变量，应用于一个主机组或一组主机组中的所有主机
- 主机变量优先于组变量，但`playbook`中定义的变量的优先级比这两者更高

&#8195;&#8195;若要定义主机变量和组变量，一种方法是直接在清单文件中定义。这种方法比较老，不容易使用，它将主机和主机组的所有清单信息和变量设置都放在了一个文件中。
- 定义`web1.big1000.com`的`ansible_user`主机变量：
    ```yaml
    [servers]
    web1.big1000.com  ansible_user=ansible
    ```
- 定义`servers`主机组的`user`组变量：
    ```yaml
    [servers]
    web1.big1000.com
    web2.big1000.com

    [servers:vars]
    user=ansible
    ```
- 定义`servers`组的`user`组变量，该组由两个主机组组成，每个主机组有两个服务器：
    ```yaml
    [servers1]
    web1.big1000.com
    web2.big1000.com

    [servers2]
    db2.big1000.com
    oracle.big1000.com

    [servers:children]
    servers1
    servers2

    [servers:vars]
    user=ansible
    ```

#### 使用目录填充主机和组变量
&#8195;&#8195;定义主机和主机组的变量的首选做法是在与清单文件或`playbook`相同的工作目录中，创建 `group_vars`和`host_vars`两个目录。
- 这两个目录分别包含用于定义组变量和主机变量的文件
- 建议使用`host_vars`和`group_vars`目录定义清单变量，而不直接在清单文件中定义它们

&#8195;&#8195;为了定义用于`servers`组的组变量，需要创建名为`group_vars/servers`的YAML文件，然后该文件的内容将使用与`playbook`相同的语法将变量设置为值：
```yaml
user: ansible
```
&#8195;&#8195;为了定义用于特定主机的主机变量，在`host_vars`目录中创建名称与主机匹配的文件来存放主机变量。例如需要管理两个数据中心，并在`~/project/inventory`清单文件中定义数据中心主机：
```
[ansible@redhat9 ~]$ cat ~/project/inventory
[datacenter1]
web1.big1000.com
web2.big1000.com

[datacenter2]
web3.big1000.com
web4.big1000.com

[datacenters:children]
datacenter1
datacenter2
```
如需为两个数据中心的所有服务器定义一个通用值，可以为`datacenters`主机组设置一个组变量：
```
[ansible@redhat9 ~]$ cat ~/project/group_vars/datacenters
package: httpd
```
如果要为每个数据中心定义不同的值，可以为每个数据中心主机组设置组变量：
```
[ansible@redhat9 ~]$ cat ~/project/group_vars/datacenter1
package: httpd
[ansible@redhat9 ~]$ cat ~/project/group_vars/datacenter2
package: apache
```
如果要为每一数据中心的各个主机定义不同的值，则在单独的主机变量文件中定义变量：
```
[ansible@redhat9 host_vars]$ cat web1.big1000.com
package: httpd
[ansible@redhat9 host_vars]$ cat web2.big1000.com
package: apache
[ansible@redhat9 host_vars]$ cat web3.big1000.com
package: oracle-server
[ansible@redhat9 host_vars]$ cat web4.big1000.com
package: db2-server
```
示例项目`project`的目录结构如果包含上面的所有示例文件：
```
[ansible@redhat9 ~]$ tree project
project
├── ansible.cfg
├── group_vars
│   ├── datacenter1
│   ├── datacenter2
│   └── datacenters
├── host_vars
│   ├── web1.big1000.com
│   ├── web2.big1000.com
│   ├── web3.big1000.com
│   └── web4.big1000.com
├── inventory
└── playbook.yml

2 directories, 10 files
```
注意事项：
- Ansible会查找与清单和`playbook`相对的`host_vars`和`group_vars`子目录
- 如果清单和`playbook`恰好在同一目录中，则比较简单，Ansible可以在相应目录中查找这两个子目录
- 如果清单和`playbook`在不同目录中，则Ansible需要从两个位置查找`host_vars`和`group_vars`子目录。`playbook`子目录的优先级更高

### 从命令行覆盖变量
&#8195;&#8195;清单变量可被`playbook`中设置的变量覆盖，这两种变量又可通过在命令行中传递参数到 `ansible`或`ansible-playbook`命令来覆盖。在命令行上设置的变量称为额外变量。当需要覆盖一次性运行的 `playbook`的变量的已定义值时，额外变量非常有用。例如：
```
[ansible@redhat9 ~]$ ansible-playbook main.yml -e "package=apache"
```
### 使用数组作为变量
&#8195;&#8195;除了将与同一元素相关的配置数据(软件包列表、服务列表和用户列表等)分配到多个变量外，管理员也可以使用数组。好处在于，数组是可以浏览的。例如有下列代码片段：
```yaml
user1_first_name: Bob
user1_last_name: Jones
user1_home_dir: /users/bjones
user2_first_name: Anne
user2_last_name: Cook
user2_home_dir: /users/acook
```
可以改写成名为`users`的数组：
```yaml
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```
可以使用以下变量来访问用户数据：
```yaml
# Returns 'Bob'
users.bjones.first_name

# Returns '/users/acook'
users.acook.home_dir
```
由于变量被定义为Python字典，因此可以使用替代语法：
```yaml
# Returns 'Bob'
users['bjones']['first_name']

# Returns '/users/acook'
users['acook']['home_dir']
```
注意事项：
- 如果键名称与Python方法或属性的名称(如discard、copy和add等)相同，点表示法可能会造成问题。使用方括号表示法有助于避免冲突和错误
- 两种语法都有效，但为了方便故障排除，建议在任何给定Ansible项目的所有文件中一致地采用一种语法

### 使用已注册变量捕获命令输出
&#8195;&#8195;可以使用`register`语句捕获命令输出。输出保存在一个临时变量中，稍后在`playbook`中可用于调试用途或者达成其他目的，例如基于命令输出的特定配置。示例`playbook`如何为调试用途捕获命令输出：
```yaml
---
- name: Installs a package and prints the result
  hosts: redhat8
  tasks:
    - name: Install the package
      yum:
        name: httpd
        state: installed
      register: install_result

    - debug:
        var: install_result
```
&#8195;&#8195;在运行`playbook`时，`debug`模块用于将`install_result`注册变量的值转储到终端。上面`playbook`运行结果示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook test2.yml

PLAY [Installs a package and prints the result] **************

TASK [Gathering Facts] **************************************
ok: [redhat8]

TASK [Install the package] *********************************                                                            
ok: [redhat8]

TASK [debug] ********************************************
ok: [redhat8] => {
    "install_result": {
        "changed": false,
        "failed": false,
        "msg": "Nothing to do",
        "rc": 0,
        "results": []
    }
}

PLAY RECAP ******************************************* 
redhat8                    : ok=3    changed=0    unreachabl                                                                  e=0    failed=0    skipped=0    rescued=0    ignored=0
```
## 管理机密
### Ansible Vault
&#8195;&#8195;Ansible可能需要访问密码或API密钥等敏感数据，以便能配置受管主机。使用Ansible随附的 `Ansible Vault`可以加密和解密任何由Ansible使用的结构化数据文件。
- 使用`Ansible Vault`，可通过`ansible-vault`命令行工具创建、编辑、加密、解密和查看文件
- `Ansible Vault`可以加密任何由Ansible使用的结构化数据文件：
    - 可能包括清单变量、`playbook`中含有的变量文件、在执行`playbook`时作为参数传递的变量文件
    - 或者Ansible角色中定义的变量
- `Ansible Vault`并不实施自有的加密函数，而是使用外部Python工具集。文件通过利用`AES256`的对称加密(将密码用作机密密钥)加以保护

#### 创建加密的文件
&#8195;&#8195;要创建新的加密文件，可使用`ansible-vault create filename`命令。该命令将提示输入新的`vault`密码，然后利用默认编辑器`vi`打开文件。可以设置和导出`EDITOR`环境变量，将默认编辑器设为`nano`，可设为`export EDITOR=nano`。
```
[ansible@redhat9 ~]$ ansible-vault create secret.yml
New Vault password:
Confirm New Vault password:
```
&#8195;&#8195;可以使用`vault`密码文件来存储`vault`密码，而不是通过标准输入途径输入`vault`密码。需要使用文件权限和其他方式来严密保护该文件：
```
[ansible@redhat9 ~]$ ansible-vault create --vault-password-file=vault-pass secret.yml
```
#### 查看加密的文件
&#8195;&#8195;可以使用`ansible-vault view filename`命令查看`Ansible Vault`加密的文件，而不必打开它进行编辑。示例如下：
```
[ansible@redhat9 ~]$ ansible-vault view secret1.yml
Vault password:
```
#### 编辑现有的加密文件
&#8195;&#8195;`ansible-vault edit filename`命令可以编辑现有的加密文件。此命令将文件解密为一个临时文件，并允许编辑该文件。保存时，它将复制其内容并删除临时文件：
```
[ansible@redhat9 ~]$ ansible-vault edit secret.yml
Vault password:
```
&#8195;&#8195;`edit`子命令始终重写文件，因此只应在进行更改时使用它。这在文件保管在版本控制下时有影响。要查看文件的内容而不进行更改，始终应使用`view`子命令。
#### 加密现有的文件
&#8195;&#8195;使用`ansible-vault encrypt filename`命令加密已存在的文件。此命令可取多个欲加密文件的名称作为参数。示例：
```
[ansible@redhat9 ~]$ ansible-vault encrypt secret2.yml secret3.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
&#8195;&#8195;可以用`--output=OUTPUT_FILE`选项，可将加密文件保存为新的名称。只能通过`--output`选项使用一个输入文件。
#### 解密现有的文件
&#8195;&#8195;可以通过`ansible-vault decrypt filename`命令对现有加密文件永久解密。在解密单个文件时，可使用`--output`选项以其他名称保存解密的文件。示例如下：
```
[ansible@redhat9 ~]$ ansible-vault decrypt secret2.yml
Vault password:
Decryption successful
```
#### 更改加密文件的密码
&#8195;&#8195;使用`ansible-vault rekey filename`命令更改加密文件的密码。此命令可一次性更新多个数据文件的密钥。它将提示提供原始密码和新密码。示例如下：
```
[ansible@redhat9 ~]$ ansible-vault rekey secret3.yml
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful
```
在使用`vault`密码文件时，使用`--new-vault-password-file`选项：
```
[ansible@redhat9 ~]$ ansible-vault rekey \
> --new-vault-password-file=NEW_VAULT_PASSWORD_FILE secret.yml
```
### Playbook和Ansible Vault
&#8195;&#8195;要运行可访问通过`Ansible Vault`加密的文件的`playbook`，需要向`ansible-playbook`命令提供加密密码。如果不提供密码，`playbook`将返回错误，示例如下：
```
ERROR: A vault password must be specified to decrypt vars/xxxx.yml
```
为`playbook`提供`vault`密码，使用`--vault-id`选项。例如，要以交互方式提供`vault`密码，使用下例中所示的`--vault-id @prompt`：
```
[ansible@redhat9 ~]$ ansible-playbook --vault-id @prompt site.yml
Vault password (default):
...output omitted...
```
更多密码提供方式：
- 使用的Ansible版本是2.4之前的，则需要使用`--ask-vault-pass`选项以交互方式提供`vault`密码。如果`playbook`使用的所有`vault`加密文件都使用同一密码加密，则仍可使用该选项
- 也可使用`--vault-password-file`选项指定以纯文本存储加密密码的文件。密码应当在该文件中存储为一行字符串。由于该文件包含敏感的纯文本密码，因此务必要通过文件权限和其他安全措施对其加以保护。命令示例：
    ```
    [ansible@redhat9 ~]$ ansible-playbook \
    >--vault-password-file=vault-pw-file site.yml
    ```
- 也可以使用`ANSIBLE_VAULT_PASSWORD_FILE`环境变量，指定密码文件的默认位置

&#8195;&#8195;从Ansible 2.4开始，可以通过`ansible-playbook`使用多个`Ansible Vault`密码。要使用多个密码，请将多个`--vault-id`或`--vault-password-file`选项传递给`ansible-playbook`命令。示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook \
> --vault-id one@prompt --vault-id two@prompt site.yml
Vault password (one):
Vault password (two):
...output omitted...
```
示例说明：
- `@prompt`前面的vault ID `one`和`two`可以是任何字符，甚至可以完全省略它们
- 如果在使用`ansible-vault`命令加密文件时使用`--vault-id id `选项，则在运行`ansible-playbook`时，将最先尝试匹配ID的密码。如果不匹配，接下来将尝试您提供的其他密码
- 没有ID的vault ID `@prompt`实际上是`default@prompt`的简写，这意味着提示输入`vault ID default`的密码

#### 变量文件管理的推荐做法
&#8195;&#8195;若要简化管理，务必要设置Ansible项目，使敏感变量和所有其他变量保存在相互独立的文件中。然后，包含敏感变量的文件可通过`ansible-vault`命令进行保护。管理组变量和主机变量的首选方式是在 `playbook`级别上创建目录：
- `group_vars`目录通常包含名称与它们所应用的主机组匹配的变量文件
- `host_vars`目录通常包含名称与它们所应用的受管主机名称匹配的变量文件

&#8195;&#8195;除了使用`group_vars`或`host_vars`中的文件外，也可对每一主机组或受管主机使用目录。这些目录而后可包含多个变量文件，它们都由该主机组或受管主机使用。例如`playbook.yml`的项目目录：
```
.
├── ansible.cfg
├── group_vars
│   └── webservers
│       └── vars
├── host_vars
│   └── demo.example.com
│       ├── vars
│       └── vault
├── inventory
└── playbook.yml
```
示例项目目录说明：
- 在`playbook.yml`的以下项目目录中，`webservers`主机组的成员将使用`group_vars/webservers/vars`文件中的变量
- 而`demo.example.com`将使用`host_vars/demo.example.com/vars`和`host_vars/demo.example.com/vault`中的变量
- 示例中，`host_vars/demo.example.com`目录内使用的文件名没有什么特别之处。该目录可以包含更多文件，一些由`Ansible Vault`加密，另一些则不加密：
    - 在这种情景中，其好处在于用于`demo.example.com`的大部分变量可以放在`vars`文件中，敏感变量则可单独放在`vault`文件中保密
    - 然后，管理员可以使用`ansible-vault`加密`vault`文件，而将`vars`文件保留为纯文本

&#8195;&#8195;如果在`playbook`中使用多个`vault`密码，确保为每个加密文件分配一个`vault ID`，并在运行`playbook`时输入具有该`vault ID`的匹配密码。
## 管理事实
### Ansible事实
&#8195;&#8195;Ansible事实是Ansible在受管主机上自动检测到的变量。事实中含有与主机相关的信息，可以像 `play`中的常规变量、条件、循环或依赖于从受管主机收集的值的任何其他语句那样使用。事实可能包括：
- 主机名称
- 内核版本
- 网络接口
- IP 地址
- 操作系统版本
- 各种环境变量
- CPU 数量
- 提供的或可用的内存
- 可用磁盘空间

借助事实，可以方便地检索受管主机的状态，并根据该状态确定要执行的操作。例如：
- 可以根据含有受管主机当前内核版本的事实运行条件任务，以此来重新启动服务器
- 可以根据通过事实报告的可用内存来自定义MySQL配置文件
- 可以根据事实的值设置配置文件中使用的IPv4地址

&#8195;&#8195;通常，每个`play`在执行第一个任务之前会先自动运行`setup`模块来收集事实。在Ansible 2.3中为`Gathering Facts`任务，在更早版本的中为`setup`。默认情况下，无需具有在`play`中运行`setup`的任务，通常会自动运行。    
&#8195;&#8195;查看为受管主机收集的事实的一种方式是运行一个收集事实并使用`debug`模块显示`ansible_facts`变量值的简短`playbook`。示例`playbook`如下：
```yaml
- name: Fact dump
  hosts: redhat8
  tasks:
    - name: Print all facts
      debug:
        var: ansible_facts
```
运行结果示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook setup.yml
PLAY [Fact dump] ************************************************************
TASK [Gathering Facts] ************************************************************
ok: [redhat8]

TASK [Print all facts] ************************************************************
ok: [redhat8] => {
    "ansible_facts": {
        "all_ipv4_addresses": [
            "192.168.100.130",
            "192.168.100.131"
        ],
        "all_ipv6_addresses": [
            "fe80::bcd7:31a:c4ac:9c09"
        ],
        "architecture": "x86_64",
        "bios_date": "07/29/2019",
        ...output omitted...
        "virtualization_type": "VMware"
    }
}

PLAY RECAP **************************************************************
redhat8                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
&#8195;&#8195;`Playbook`以`JSON`格式显示`ansible_facts`变量的内容，作为变量的散列/字典。可以浏览输出来查看收集了哪些事实，并查找可能要在`play`中使用的事实。下表是可能从受管节点收集的并可在`playbook`中使用的一些事实：

事实|变量
:---|:---
短主机名|`ansible_facts['主机名称']`
完全限定的域名|`ansible_facts['fqdn']`
主要IPv4地址(基于路由)|`ansible_facts['default_ipv4']['address']`
所有网络接口的名称列表|`ansible_facts['interfaces']`
`/dev/vda1`磁盘分区的大小|`ansible_facts['devices']['vda']['partitions']['vda1']['size']`
DNS服务器列表|`ansible_facts['dns']['nameservers']`
当前运行的内核的版本|`ansible_facts['kernel']`

如果变量的值为散列/字典，则可使用两种语法来检索该值。从上表中举两例：
- `ansible_facts['default_ipv4']['address']`也可写成`ansible_facts.default_ipv4.address`
- `ansible_facts['dns']['nameservers']`也可写成`ansible_facts.dns.nameservers`

在`playbook`中使用事实时，Ansible将事实的变量名动态替换为对应的值：
```yaml
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: >
        The default IPv4 address of {{ ansible_facts.fqdn }}
        is {{ ansible_facts.default_ipv4.address }}
```
&#8195;&#8195;下面示例演示了Ansible如何查询受管节点，并且动态使用系统信息来更新变量。也可使用事实来创建符合特定标准的动态主机组：
```
PLAY ***********************************************************************

TASK [Gathering Facts] *****************************************************
ok: [demo1.example.com]

TASK [Prints various Ansible facts] ****************************************
ok: [demo1.example.com] => {
    "msg": "The default IPv4 address of demo1.example.com is
            172.25.250.10"
}

PLAY RECAP *****************************************************************
demo1.example.com    : ok=2    changed=0    unreachable=0    failed=0
```
### Ansible事实作为变量注入
&#8195;&#8195;在Ansible2.5之前，事实是作为前缀为字符串`ansible_`的单个变量注入，而不是作为`ansible_facts`变量的一部分注入。例如`ansible_facts['distribution']`事实可以称为`ansible_distribution`。    
&#8195;&#8195;较旧的`playbook`仍然使用作为变量注入的事实，而不是在`ansible_facts`变量下创建命名空间的新语法。可以使用临时命令来运行`setup`模块，以此形式显示所有事实的值。以下示例中使用一个临时命令在受管主机`redhat8`上运行`setup`模块：
```
[ansible@redhat9 ~]$ ansible redhat8 -m setup
redhat8 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.100.130",
            "192.168.100.131"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::bcd7:31a:c4ac:9c09"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
    ...output omitted...
    },
    "changed": false
}
```
旧的和新的事实名称对比如下表：

ansible_facts形式|旧事实变量形式
:---|:---
`ansible_facts['主机名称']`|ansible_hostname
`ansible_facts['fqdn']`|ansible_fqdn
`ansible_facts['default_ipv4']['address']`|`ansible_default_ipv4['address']`
`ansible_facts['interfaces']`|ansible_interfaces
`ansible_facts['devices']['vda']['partitions']['vda1']['size']`|`ansible_devices['vda']['partitions']['vda1']['size']`
`ansible_facts['dns']['nameservers']`|`ansible_dns['nameservers']`
`ansible_facts['kernel']`|ansible_kernel

&#8195;&#8195;目前Ansible同时识别新的事实命名系统和旧的命名系统。将Ansible配置文件的`[default]`部分中的 `inject_facts_as_vars`参数设置为`false`，可关闭旧命名系统。默认设置目前为`true`。如果更改为`false`，尝试通过旧命名空间引用事实将导致以下错误：
```
...output omitted...
TASK [Show me the facts] *************************************************
fatal: [demo.example.com]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible_distribution' is undefined\n\nThe error appears to have been in
 '/home/student/demo/playbook.yml': line 5, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n    - name: Show me the facts\n      ^ here\n"}
...output omitted...
```
### 关闭事实收集
不想为`play`收集事实的原因可能有：
- 不准备使用任何事实，并且希望加快`play`速度或减小`play`在受管主机上造成的负载
- 受管主机因为某种原因而无法运行`setup`模块，或者需要安装一些必备软件后再收集事实

为`play`禁用事实收集可将`gather_facts`关键字设为`no`：
```yaml
---
- name: This play gathers no facts automatically
  hosts: large_farm
  gather_facts: no
```
&#8195;&#8195;即使`play`设置了`gather_facts: no`，也可以随时使用`setup`模块的任务来手动收集事实。示例如下：
```
  tasks:
    - name: Manually gather facts
      setup:
...output omitted...
```
### 创建自定义事实
&#8195;&#8195;管理员可以创建自定义事实，将其本地存储在每个受管主机上。这些事实整合到`setup`模块在受管主机上运行时收集的标准事实列表中。它们让受管主机能够向Ansible提供任意变量，以用于调整`play`的行为。
- 自定义事实可以在静态文件中定义，格式可为`INI`文件或采用`JSON`。它们也可以是生成`JSON`输出的可执行脚本
- 借助自定义事实，管理员可以为受管主机定义特定的值，供`play`用于填充配置文件或有条件地运行任务。动态自定义事实允许在`play`运行时以编程方式确定这些事实的值，甚至还可以确定提供哪些事实
- 默认情况下，`setup`模块从各受管主机的`/etc/ansible/facts.d`目录下的文件和脚本中加载自定义事实：
    - 各个文件或脚本的名称必须以`.fact`结尾才能被使用
    - 动态自定义事实脚本必须输出`JSON`格式的事实，而且必须是可执行文件

&#8195;&#8195;示例采用`INI`格式编写的静态自定义事实文件。`INI`格式的自定义事实文件包含由一个部分定义的顶层值，后跟用于待定义的事实的键值对：
```ini
[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = huang
user2 = ansible
```
&#8195;&#8195;也以`JSON`格式提供同样事实。以下`JSON`事实等同于以上示例中`INI`格式指定的事实。`JSON`数据可以存储在静态文本文件中，或者通过可执行脚本输出到标准输出：
```json
{
  "packages": {
    "web_package": "httpd",
    "db_package": "mariadb-server"
  },
  "users": {
    "user1": "huang",
    "user2": "ansible"
  }
}
```
&#8195;&#8195;自定义事实由`setup`模块存储在`ansible_facts['ansible_local']`变量中。事实按照定义它们的文件的名称来整理。例如，假设前面的自定义事实由受管主机上保存为`/etc/ansible/facts.d/custom.fact`的文件生成。这时，`ansible_facts['ansible_local']['custom']['users']['user1']`的值为`huang`。可以利用临时命令在受管主机上运行`setup`模块来检查自定义事实的结构，官方示例：
```
[user@demo ~]$ ansible demo1.example.com -m setup
demo1.example.com | SUCCESS => {
    "ansible_facts": {
...output omitted...
        "ansible_local": {
            "custom": {
                "packages": {
                    "db_package": "mariadb-server",
                    "web_package": "httpd"
                },
                "users": {
                    "user1": "joe",
                    "user2": "jane"
                }
            }
        },
...output omitted...
    },
    "changed": false
}
```
自定义事实的使用方式与`playbook`中的默认事实相同，官方示例：
```
[user@demo ~]$ cat playbook.yml
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: >
           The package to install on {{ ansible_facts['fqdn'] }}
           is {{ ansible_facts['ansible_local']['custom']['packages']['web_package'] }}

[user@demo ~]$ ansible-playbook playbook.yml
PLAY ***********************************************************************

TASK [Gathering Facts] *****************************************************
ok: [demo1.example.com]

TASK [Prints various Ansible facts] ****************************************
ok: [demo1.example.com] => {
    "msg": "The package to install on demo1.example.com  is httpd"
}

PLAY RECAP *****************************************************************
demo1.example.com    : ok=2    changed=0    unreachable=0    failed=0
```
### 使用魔法变量
&#8195;&#8195;一些变量并非事实或通过`setup`模块配置，但也由Ansible自动设置。这些魔法变量也可用于获取与特定受管主机相关的信息。最常用的有四个：
- `hostvars`：包含受管主机的变量，可以用于获取另一台受管主机的变量的值。如果还没有为受管主机收集事实，则它不会包含该主机的事实
- `group_names`：列出当前受管主机所属的所有组
- `groups`：列出清单中的所有组和主机
- `inventory_hostname`：包含清单中配置的当前受管主机的主机名称。这可能因为各种原因而与事实报告的主机名称不同

&#8195;&#8195;另外还有许多其他的`魔法变量`。有关更多信息参考[Variable precedence: Where should I put a variable?](https://docs.ansible.com/ansible/2.9/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)。深入了解它们的值，可以使用`debug`模块报告特定主机的`hostvars`变量的内容：
```
[ansible@redhat9 ~]$ ansible localhost -m debug -a 'var=hostvars["localhost"]'
localhost | SUCCESS => {
    "hostvars[\"localhost\"]": {
        "ansible_check_mode": false,
        "ansible_config_file": "/home/ansible/ansible.cfg",
        "ansible_connection": "local",
        "ansible_diff_mode": false,
        "ansible_facts": {},
        "ansible_forks": 5,
        "ansible_inventory_sources": [
            "/home/ansible/inventory"
        ],
        "ansible_playbook_python": "/usr/bin/python3.9",
        "ansible_python_interpreter": "/usr/bin/python3.9",
        "ansible_verbosity": 0,
        "ansible_version": {
            "full": "2.12.2",
            "major": 2,
            "minor": 12,
            "revision": 2,
            "string": "2.12.2"
        },
        "group_names": [],
        "groups": {
            "all": [
                "redhat8",
                "192.168.100.131"
            ],
            "testhosts": [
                "redhat8",
                "192.168.100.131"
            ],
            "ungrouped": []
        },
        "inventory_hostname": "localhost",
        "inventory_hostname_short": "localhost",
        "omit": "__omit_place_holder__b264b0e13db6af9e4a57204f1d159fb7d8e2eb95",
        "playbook_dir": "/home/ansible"
    }
}
```