# Ansible-自动执行Linux管理任务
## 管理软件和订阅
### 使用Ansible管理软件包
&#8195;&#8195;`yum Ansible`模块在受管主机上使用Yum软件包管理器来处理软件包操作。下例中的Playbook可将`httpd`软件包安装到`servera.lab.example.com`受管主机上：
```yml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the httpd packages
      yum:
        name: httpd
        state: present
```
- `yum`中`name `关键字提供要安装的软件包的名称
- `state`关键字指明该软件包在受管主机上的预期状态：
    - `present`：如果尚不存在，Ansible将安装该软件包
    - `absent`：如果已安装，Ansible将删除该软件包
    - `latest`：如果软件包还不是最新的可用版本，Ansible将更新该软件包。如果软件包尚未安装，则 Ansible会安装它

`yum Ansible`模块和同等作用的`yum`命令的一些用法进行对比：
- `yum install httpd`，对应Ansible任务：
    ```yml
    - name: Install httpd
      yum:
        name: httpd
        state: present
    ```
- `yum update httpd`或`yum install httpd`（未安装软件包的情况下），对应Ansible任务：
    ```yml
    - name: Install or update httpd
      yum:
        name: httpd
        state: latest
    ```
- `yum update`，对应Ansible任务：
    ```yml
    - name: Update all packages
      yum:
        name: '*'
        state: latest
    ```
- `yum remove httpd`，对应Ansible任务：
    ```yml
    - name: Remove httpd
      yum:
        name: httpd
        state: absent
    ```
- `yum group install "Development Tools"`，对应Ansible任务：
    ```yml
    - name: Install Development Tools
      yum:
        name: '@Development Tools' 
        state: present
    ```
    使用`yum Ansible`模块时，必须在组名称前面添加前缀`@`。请记住，可以使用`yum group list`命令来检索组列表
- `yum group remove "Development Tools"`，对应Ansible任务：
    ```yml
    - name: Remove Development Tools
      yum:
        name: '@Development Tools'
        state: absent
    ```
- `yum module install perl:5.26/minimal`，对应Ansible任务：
    ```yml
    - name: Inst perl AppStream module
      yum:
        name: '@perl:5.26/minimal'
        state: present
    ```
    若要管理`Yum AppStream`模块，请在其名称前加上前缀`@`。语法与使用`yum`命令相同。例如，可以省略配置集部分以使用默认配置集：`@perl:5. 26`。请记住，可以使用`yum module list`命令列出可用的`Yum AppStream`模块

运行`ansible-doc yum`命令查看其他参数和Playbook示例。
#### 优化多软件包安装
&#8195;&#8195;如要对几个软件包进行操作，`name`关键字可接受列表。下例演示了一个Playbook，它可将三个软件包安装到`servera.lab.example.com`上：
```yml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the packages
      yum:
        name:
          - httpd
          - mod_ssl
          - httpd-tools
        state: present
```
&#8195;&#8195;通过利用这种语法，Ansible可在单个`Yum`事务中安装这些软件包。此操作等同于运行`yum install httpd mod_ssl httpd-tools`命令。此任务的常见版本是使用循环，但效率较低且速度较慢(避免使用此方法，因为它需要模块执行三个单独的事务，每个事务对应一个软件包)：
```yml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the packages
      yum:
        name: "{{ item }}""
        state: present
      loop:
        - httpd
        - mod_ssl
        - httpd-tools
```
#### 收集有关已安装软件包的事实
&#8195;&#8195;`package_facts Ansible`模块收集受管主机上已安装软件包的详细信息。它使用软件包详细信息来设置`ansible_facts.packages`变量。    
&#8195;&#8195;以下Playbook调用`package_facts`模块，再调用`debug`模块来显示`ansible_facts.packages`变量的内容，然后再次调用`debug`模块来查看已安装的`NetworkManager`软件包的版本：
```yml
---
- name: Display installed packages
  hosts: redhat8
  tasks:
    - name: Gather info on installed packages
      package_facts:
        manager: auto

    - name: List installed packages
      debug:
        var: ansible_facts.packages

    - name: Display NetworkManager version
      debug:
        msg: "Version {{ansible_facts.packages['NetworkManager'][0].version}}"
      when: "'NetworkManager' in ansible_facts.packages"
```
运行时，该Playbook将显示软件包列表和`NetworkManager`软件包的版本
```
[ansible@redhat9 ~]$ ansible-playbook lspackages.yml
PLAY [Display installed packages] **********************************************************************
TASK [Gathering Facts] ***********************************************************************
ok: [redhat8]
TASK [Gather info on installed packages] ***********************************************************************
ok: [redhat8]
TASK [List installed packages] ***********************************************************************
ok: [redhat8] => {
    "ansible_facts.packages": {
        "ModemManager": [
            {
                "arch": "x86_64",
                "epoch": null,
                "name": "ModemManager",
                "release": "1.el8",
                "source": "rpm",
                "version": "1.8.0"
            }
        ],
        "NetworkManager": [
            {
                "arch": "x86_64",
                "epoch": 1,
                "name": "NetworkManager",
                "release": "14.el8",
                "source": "rpm",
                "version": "1.14.0"
            }
        ],
        ...output omitted...
    }
}

TASK [Display NetworkManager version] ***********************************************************************
ok: [redhat8] => {
    "msg": "Version 1.14.0"
}
PLAY RECAP ************************************************************
redhat8 : ok=4 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```
#### 查看用于管理软件包的其他模块
&#8195;&#8195;`yum Ansible`模块适用于使用Yum软件包管理器的受管主机。对于其他软件包管理器，Ansible 通常提供一个专用模块。例如：
- `dnf`模块可管理利用DNF软件包管理器的操作系统（如Fedora）中的软件包
- `apt`模块使用Debian或Ubuntu中可用的APT软件包工具
- `win_package`模块可以在Microsoft Windows系统中安装软件

以下Playbook使用条件在由RHEL和Fedora系统组成的环境中选择适当的模块：
```yml
---
- name: Install the required packages on the web servers
  hosts: redhat8
  tasks:
    - name: Install httpd on RHEL
      yum:
        name: httpd
        state: present
      when: "ansible_distribution == 'RedHat'"

    - name: Install httpd on Fedora
      dnf:
        name: httpd
        state: present
      when: "ansible_distribution == 'Fedora'"
```
&#8195;&#8195;作为替代选择，通用的`package`模块可自动检测并使用受管主机上可用的软件包管理器。使用`package`模块时，可以按照如下所示重写上面的Playbook：
```yml
---
- name: Install the required packages on the web servers
  hosts: webservers
  tasks:
    - name: Install httpd
      package:
        name: httpd
        state: present
```
&#8195;&#8195;请注意，`package`模块不支持更为专业的模块所提供的所有功能。此外，操作系统通常对它们提供的软件包使用不同的名称。例如，用于安装`Apache HTTP Server`的软件包在RHEL中名为`httpd`，在Ubuntu中则名为`apache2`。在这种情况下，仍需要一个条件来选择正确的软件包名称，具体取决于受管主机的操作系统。
### 使用红帽订阅管理注册和管理系统
&#8195;&#8195;为了让用户的新RHEL系统获得产品订阅权利，Ansible提供了`redhat_subscription`和 `rhsm_repository`模块。这些模块与受管主机上的红帽订阅管理工具进行接口。
#### 注册和订阅新系统
&#8195;&#8195;使用红帽订阅管理工具时，执行的前两个任务通常是注册新系统和附加可用订阅。如果没有Ansible，可以使用`subscription-manager`命令来执行这些任务：
```
[user@host ~]$ subscription-manager register --username=yourusername \
> --password=yourpassword
[user@host ~]$ subscription-manager attach --pool=poolID
```
&#8195;&#8195;请记住，可通过`subscription-manager list --available`命令列出帐户中可用的池。
`redhat_subscription Ansible`模块可在一个任务中执行注册和订阅：
```yml
- name: Register and subscribe the system
  redhat_subscription:
    username: yourusername
    password: yourpassword
    pool_ids: poolID
    state: present
```
`state`关键字设为`present`时表示注册并订阅系统。如果设为`absent`，模块将取消注册系统。
#### 启用红帽软件存储库
&#8195;&#8195;订阅之后，下一任务是在新系统上启用红帽软件存储库。如果没有Ansible，要执行`subscription-manager`命令来实现这一目标：
```
[user@host ~]$ subscription-manager repos \
> --enable "rhel-8-for-x86_64-baseos-rpms" \
> --enable "rhel-8-for-x86_64-baseos-debug-rpms"
```
&#8195;&#8195;可以使用`subscription-manager repos --list`命令列出可用的存储库。对于Ansible，请使用`rhsm_repository`模块：
```yml
- name: Enable Red Hat repositories
  rhsm_repository:
    name:
      - rhel-8-for-x86_64-baseos-rpms
      - rhel-8-for-x86_64-baseos-debug-rpms
    state: present
```
### 配置Yum存储库
为了在受管主机上启用对第三方存储库的支持，Ansible提供了`yum_repository`模块。
#### 声明Yum存储库
运行以下Playbook时，它将声明一个位于`servera.lab.example.com`的新存储库：
```yml
---
- name: Configure the company Yum repositories
  hosts: servera.lab.example.com
  tasks:
    - name: Ensure Example Repo exists
      yum_repository:
        file: example
        name: example-internal
        description: Example Inc. Internal YUM repo
        baseurl: http://materials.example.com/yum/repository/
        enabled: yes
        gpgcheck: yes
        state: present
```
示例说明：
- `file: example`：`file`关键字提供要在`/etc/yum.repos.d/`目录下创建的文件的名称。模块将自动为该名称添加`.repo`扩展名
- `gpgcheck: yes`：通常，软件提供商使用GPG密钥对RPM软件包进行数字签名。通过将`gpgcheck`关键字设为 `yes`，RPM系统将确认软件包已由相应的GPG密钥签名以验证软件包的完整性。如果GPG签名不匹配，它将拒绝安装软件包。使用`rpm_key Ansible`模块安装所需的GPG公钥
- `state: present`：当`state`关键字设为`present`时，Ansible会创建或更新`.repo`文件。当`state`设为`absent`时，Ansible会删除该文件

`servera.lab.example.com`上生成的`/etc/yum.repos.d/example.repo`文件如下所示：
```ini
[example-internal]
baseurl = http://materials.example.com/yum/repository/
enabled = 1
gpgcheck = 1
name = Example Inc. Internal YUM repo
```
&#8195;&#8195;`yum_repository`模块将大多数Yum存储库配置参数公开为关键字。运行`ansible-doc yum_repository`命令查看其他参数和Playbook示例。   
&#8195;&#8195;一些第三方存储库将一个配置文件和GPG公钥作为RPM软件包的一部分提供，该软件包可以使用 `yum install`命令来下载和安装。例如，Extra Packages for Enterprise Linux (EPEL) 项目提供了 `https://dl.fedoraproject.org/pub/epel/epel-release-latest-VER.noarch.rpm`软件包，该软件包会部署`/etc/yum.repos.d/epel.repo`配置文件。对于此存储库，请使用`yum Ansible`模块来安装`EPEL`软件包，而不要使用`yum_repository`模块。
#### 导入RPM GPG密钥
&#8195;&#8195;当`yum_repository`模块中的`gpgcheck`关键字设为`yes`时，还需要在受管主机上安装GPG密钥。下例中的`rpm_key`模块会在`servera.lab.example.com`上部署托管于远程Web服务器的`GPG`公钥：
```yml
---
- name: Configure the company Yum repositories
  hosts: servera.lab.example.com
  tasks:
    - name: Deploy the GPG public key
      rpm_key:
        key: http://materials.example.com/yum/repository/RPM-GPG-KEY-example
        state: present

    - name: Ensure Example Repo exists
      yum_repository:
        file: example
        name: example-internal
        description: Example Inc. Internal YUM repo
        baseurl: http://materials.example.com/yum/repository/
        enabled: yes
        gpgcheck: yes
        state: present
```
## 管理用户和身份验证
### user模块
&#8195;&#8195;Ansible `user`模块允许管理远程主机上的用户帐户。可以管理许多参数，包括删除用户、设置主目录、设置系统帐户的UID，以及管理密码和关联的分组。若要创建可以登录计算机的用户，需要为`password`参数提供哈希密码。`user`模块示例：
```yml
- name: Add new user to the development machine and assign the appropriate groups.
  user:
    name: devops_user
    shell: /bin/bash
    groups: sys_admins, developers
    append: yes
```
示例说明：
- `name: devops_user`：`name`参数是`user`模块中的唯一必要参数，通常是服务帐户或用户帐户
- `shell: /bin/bash`：`shell`参数用于设置用户的shell（可选）。在其他操作系统上，默认shell由正在使用的工具决定
- `groups: sys_admins, developers`：`groups`参数及`append`参数告知计算机，要为此用户附加`sys_asmins`和`developers`组。如果不使用`append`参数，则组将覆盖现有的组

&#8195;&#8195;创建用户时，可以将其指定为`generate_ssh_key`。这不会覆盖现有的SSH密钥。使用user模块生成ssh密钥的示例：
```yml
- name: Create a SSH key for user1
  user:
    name: user1
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_my_rsa
```
注意：
- `user`模块也提供一些返回值。Ansible模块可以取用返回值并将它们注册到变量中
- 可通过`ansible-doc`或在主要文档网站上了解更多信息

一些常用的参数：

参数|注释
:---:|:---
comment|(可选)设置用户帐户的描述
group|(可选)设置用户的主要组
groups|多个组的列表。设置为空值时，将删除除主组之外的所有组
home|(可选)设置用户的主目录
create_home|取布尔值yes或no。如果值设置为yes，将为用户创建主目录
system|在创建state=present的帐户时，将此参数设为yes可使该用户成为系统帐户。无法对现有用户更改此设置
uid|设置用户的UID

### group模块
&#8195;&#8195;`group`模块允许管理（添加、删除或修改）受管主机上的组。需要具有`groupadd`、`groupdel`或`groupmod`。对于Windows目标使用`win_group`模块。`group`模块示例：
```yml
- name: Verify that auditors group exists
  group:
    name: auditors
    state: present
```
`group`模块参数：

参数|注释
:---:|:---
gid|为组设置的可选GID
local|强制在实施它的平台上使用“本地”命令替代选择
name|要管理的组的名称
state|远程主机上是否应存在该组
system|如果设置为yes，则表示创建的组是系统组

### 已知主机模块
&#8195;&#8195;如果要管理大量主机密钥，则需要使用`known_hosts`模块。`known_hosts`模块允许在受管主机上的`known_hosts`文件中添加或删除主机密钥。`known_hosts`任务示例：
```yml
- name: copy host keys to remote servers
  known_hosts:
    path: /etc/ssh/ssh_known_hosts
    name: host1
    key: "{{ lookup('file', 'pubkeys/host1') }}" #lookup插件允许Ansible访问外部来源的数据
```
### authorized_key模块
&#8195;&#8195;`authorized_key`模块允许为各个用户帐户添加或删除SSH授权密钥。在向大量服务器添加和减少用户时，需要能够管理ssh密钥。`authorized_key`任务示例：
```yml
- name: Set authorized key
  authorized_key:
    user: user1
    state: present
    key: "{{ lookup('file', '/home/user1/.ssh/id_rsa.pub') }}" # 也可以从url(https://github.com/user1.keys)获取密钥
```
## 管理引导过程和调度的进程
### 使用at模块进行调度
&#8195;&#8195;通过`at`模块来进行一次性快速调度。可以将作业创建为在将来的某个时间运行，它将等待到该时间执行。此模块附带六个参数。分别是`command`、`count`、`script_file`、`state`、`unique`和`units`。`at`模块示例：
```yml
- name: remove tempuser.
  at:
    command: userdel -r tempuser
    count: 20
    units: minutes
    unique: yes
```
六个参数说明：

参数|选项|注释
:---:|:---:|:---		
command|Null|计划运行的命令
count|Null|单位数。(必须与units一起运行)
script_file|Null|要在将来执行的现有脚本文件
state|absent、present|添加或删除命令或脚本的状态
unique|yes、no|如果作业已在运行，则不会再次执行
units|minutes/hours/days/weeks|时间名称

### 使用cron模块附加命令
&#8195;&#8195;在设置作业调度任务时，可使用`cron`模块。`cron`模块将命令直接附加到指定用户的 `crontab`中。`cron`模块示例：
```yml
- cron:
    name: "Flush Bolt"
    user: "root"
    minute: 45
    hour: 11
    job: "php ./app/nut cache:clear"
```
示例说明
- 此play在清空`Bolt`缓存后立即使用`cache:clear`命令，在每天上午的`11:45`删除缓存的文件以及`CMS`服务器的`directories.flushes`缓存
- Ansible将使用用户声明的正确语法将play写入到`crontab`中
- 通过检查`crontab`来验证它是否已附加。

`cron`模块的一些常用参数有：

参数|选项|注释
:---:|:---:|:---
special_time|reboot、yearly、annually、monthly、 weekly、daily、hourly|一系列重复出现时间
state|absent、present|如果设为present，它将创建命令。设为absent则会删除命令
cron_file|Null|如果有大量的服务器需要维护，那么有时最好有一个预先编写好的crontab文件
backup|yes、no|在编辑之前备份crontab文件

### 使用systemd和service模块管理服务
为管理服务或重新加载守护进程，Ansible提供了`systemd`和`service`模块：
- `service`提供了一组基本选项，即`start`、`stop`、`restart`和`enable`
- `systemd`模块提供更多配置选项
- `Systemd`允许您执行守护进程重新加载，`service`模块则不会

`service`模块示例：
```yml
- name: start nginx
  service:
    name: nginx
    state: started"
```
`systemd`模块示例：
```yml
- name: reload web server
  systemd:
    name: apache2
    state: reload
    daemon-reload: yes
```
### reboot模块
&#8195;&#8195;另一个很好用的Ansible系统模块是`reboot`。这被认为比使用shell模块来发起关机更加安全。在运行play期间，`reboot`模块将关闭受管主机，再等待它恢复运行，然后再继续执行play。`reboot`模块示例：
```yml
- name: "Reboot after patching"
  reboot:
    reboot_timeout: 180

- name: force a quick reboot
  reboot:
```
### shell和command模块
&#8195;&#8195;与`service`和`systemd`模块一样，`shell`和`command`也可以交换一些任务。`command`模块被认为更安全，但某些环境变量不可用。此外，流操作符将无法运作。如果需要流式传输命令，那么`shell`模块就可以。`shell`模块示例：
```yml
- name: Run a templated variable (always use quote filter to avoid injection)
    shell: cat {{ myfile|quote }} # 要清理变量，建议使用{{ var | quote }}而非{{ var }}
```
`command`模块示例：
```yml
- name: This command only
  command: /usr/bin/scrape_logs.py arg1 arg2
  args: # 可以将参数传递到表单中以提供选项
    chdir: scripts/
    creates: /path/to/script
```
&#8195;&#8195;通过在受管主机上收集事实，可以访问环境变量。有一个名为`ansible_env`的子列表，其中包含所有的环境变量：
```yml
---
- name:
  hosts: webservers
  vars:
    local_shell:  "{{ ansible_env }}" # 可以使用lookup插件确定要返回的变量。msg: "{{ lookup('env','USER','HOME','SHELL') }}"
  tasks:
    - name: Printing all the environment variables in Ansible
      debug:
        msg: "{{ local_shell }}"
```
## 管理存储
### 使用Ansible模块配置存储
