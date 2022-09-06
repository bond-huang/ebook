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