# RHEL-分析服务器及获取支持
## 分析和管理远程服务器
### Web控制台描述
&#8195;&#8195;Web控制台是适用于RHEL8 的基于Web型管理界面，专为管理和监控您的服务器而设计。它的基础是开源Cockpit服务。可以使用 Web 控制台监控系统日志并查看系统性能图表。
### 启用Web控制台
&#8195;&#8195;除了最小安装外，RHEL8的所有安装版本中都默认安装Web控制台。可以使用`yum`命令安装Web控制台：
```
[root@redhat8 ~]# yum install cockpit
```
&#8195;&#8195;启用并启动`cockpit.socket`服务，它会运行一个Web服务器。如果需要通过Web界面连接系统，必须执行这个步骤：
```
[root@redhat8 ~]# systemctl enable --now cockpit.socket
Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/
systemd/system/cockpit.socket.
```
&#8195;&#8195;如果使用的是自定义防火墙配置集，需要将`cockpit`服务添加到`firewalld`，以在防火墙中开启端口`9090`：
```
[root@redhat8 ~]# firewall-cmd --add-service=cockpit --permanent
Warning: ALREADY_ENABLED: cockpit
success
[root@redhat8 ~]# firewall-cmd --reload
success
```
### 登录Web控制台
&#8195;&#8195;Web控制台提供自己的Web服务器。可以使用系统上任何本地帐户的用户名和密码登录，包括root 用户在内。在Web浏览器中打开地址：
```
https://servername:9090
```
说明：
- 其中`servername`是服务器的主机名或IP地址
- 连接将受到TLS会话的保护。默认情况下，系统安装有一个自签名的 TLS 证书，首次连接时，Web浏览器可能会显示安全警告。
- 在登录屏幕中输入用户名和密码
- 选择`Reuse my password for privileged tasks`选项。这将允许使用`sudo`特权执行命令，从而执行诸如修改系统信息或配置新帐户之类的任务
- 登录后Web控制台在标题栏右侧显示用户名。如果选择了`Reuse my password for privileged tasks`选项，用户名的左侧会显示`Privileged`图标
- 如果以某个非特权用户身份登录，则不显示`Privileged`图标

### 更改密码
&#8195;&#8195;特权和非特权用户都可在登录Web控制台期间更改自己的密码。单击导航栏中的`Accounts`。单击帐户标签，以打开帐户详细信息页面：
- 作为非特权用户时，只能设置或重置自己的密码，或者更改SSH公钥
- 若要设置或重置密码，单击`Set Password`

### 使用Web控制台进行故障排除
&#8195;&#8195;Web控制台是一个功能强大的故障排除工具。可以实时监控基本系统统计信息，检查系统日志，并快速切换到Web控制台中的终端会话，以便从命令行界面中收集其他信息。
#### 实时监控系统统计信息
