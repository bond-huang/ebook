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