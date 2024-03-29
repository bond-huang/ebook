# Ansible-常见问题
## 权限问题
### 提示需要root用户
运行`ansible-playbook`报错示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook -C site.yml
PLAY [Install and start Apache HTTPD] ***********************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [redhat8]

TASK [httpd package is present] ***********************************************************************
fatal: [redhat8]: FAILED! => {"changed": false, "msg": "This command has to be run under the root user.", "results": []}

PLAY RECAP ************************************************************
redhat8  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```
&#8195;&#8195;没有root权限，用户也不能使用`sudo`，在ansible.cfg文件中`become`相关项，检查配置文件。通常配置如下所示：
```ini
[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```
### 提示没有权限
报错示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook -C site.yml

PLAY [Install and start Apache HTTPD] ****************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
fatal: [redhat8]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: user@redhat8: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}

PLAY RECAP *******************************************************************************************************************
redhat8                    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
```
运行ping模块同样报错：
```
[ansible@redhat9 ~]$ ansible redhat8 -m ping
redhat8 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: user@redhat8: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).",
    "unreachable": true
}
```
&#8195;&#8195;用户没有做ssh验证，或者是做了，但是在ansible.cfg文件中`remote_user = `项的用户写错了，检查ssh验证，检查配置文件。
## 网络问题
### 网络不通
报错示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook loop_register.yml
PLAY [Loop Register Test] ***************************************************************************
TASK [Looping Echo Task] ***************************************************************************
failed: [redhat8] (item=one) => {"ansible_loop_var": "item", "item": "one", "msg": "Failed to connect to the host via ssh: ssh: connect to host redhat8 port 22: No route to host", "unreachable": true}
failed: [redhat8] (item=two) => {"ansible_loop_var": "item", "item": "two", "msg": "Failed to connect to the host via ssh: ssh: connect to host redhat8 port 22: No route to host", "unreachable": true}
fatal: [redhat8]: UNREACHABLE! => {"changed": false, "msg": "All items completed", "results": [{"ansible_loop_var": "item", "item": "one", "msg": "Failed to connect to the host via ssh: ssh: connect to host redhat8 port 22: No route to host", "unreachable": true}, {"ansible_loop_var": "item", "item": "two", "msg": "Failed to connect to the host via ssh: ssh: connect to host redhat8 port 22: No route to host", "unreachable": true}]}

PLAY RECAP *******************************************************************************************************************
redhat8                    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
```
&#8195;&#8195;上面报错原因通常是到目标host的网络不通导致的，或者`/etc/hosts`里面写入了错误的IP地址导致网络不能通信。更改为正确的即可。
## 语法问题
### 语法使用不当
示例报错如下：
```
ERROR! 'file' is not a valid attribute for a Play

The error appears to be in '/home/ansible/file2.yml': line 1, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

- name: Touch a file and set permissions
  ^ here
```
检查Play的语法或模块使用方法，确保正确编写Playbook。
## 模块问题
### sefcontext模块使用问题
报错示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook file3.yml
ERROR! couldn't resolve module/action 'sefcontext'. 
This often indicates a misspelling, missing collection, or incorrect module path.
```
&#8195;&#8195;查询需要安装`libselinux-python`和`policycoreutils-python`，控制节点和受控节点上都需要安装。
### synchronize模块问题
报错示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook synchronize.yml
ERROR! couldn't resolve module/action 'synchronize'. 
This often indicates a misspelling, missing collection, or incorrect module path.
```
`rsync`工具必须同时安装在本地和远程主机上。
## 待补充