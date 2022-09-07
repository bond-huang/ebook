# Ansible-官方实例参考
## 常用软件安装
### Apache Web服务安装
&#8195;&#8195;安装Apache Web服务器并开启相关的端口以使其服务可被访问。`Playbook`查询Web服务器以确认其已设好并在运行。创建`playbook`并在`vars`部分中定义以下变量：

变量|描述
:---:|:---
web_pkg|要安装的 Web 服务器软件包
firewall_pkg|要安装的防火墙软件包
web_service|要管理的 Web 服务
firewall_service|要管理的防火墙服务
python_pkg 	uri|模块所需的软件包
rule|要打开的服务

完整`playbook`内容参考如下：
```yaml
---
- name: Deploy and start Apache HTTPD service
  hosts: webserver
  vars:
    web_pkg: httpd
    firewall_pkg: firewalld
    web_service: httpd
    firewall_service: firewalld
    python_pkg: python3-PyMySQL
    rule: http

  tasks:
    - name: Required packages are installed and up to date
      yum:
        name:
          - "{{ web_pkg  }}"
          - "{{ firewall_pkg }}"
          - "{{ python_pkg }}"
        state: latest

    - name: The {{ firewall_service }} service is started and enabled
      service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started

    - name: The {{ web_service }} service is started and enabled
      service:
        name: "{{ web_service }}"
        enabled: true
        state: started

    - name: Web content is in place
      copy:
        content: "Example web content"
        dest: /var/www/html/index.html

    - name: The firewall port for {{ rule }} is open
      firewalld:
        service: "{{ rule }}"
        permanent: true
        immediate: true
        state: enabled

- name: Verify the Apache service
  hosts: localhost
  become: false
  tasks:
    - name: Ensure the webserver is reachable
      uri:
        url: http://servera.lab.example.com
        status_code: 200
```
### 待补充