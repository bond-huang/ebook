# Zabbix-Agent部署
## 安装Agent
安装Agent端软件：
```
[root@centos82 ~]# dnf install zabbix-agent
Last metadata expiration check: 1:43:50 ago on Wed 16 Aug 2023 01:15:32 PM CST.
Package zabbix-agent-6.0.20-release1.el8.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
备份`agentd`配置文件：
```
[root@centos82 zabbix]# pwd
/etc/zabbix
[root@centos82 zabbix]# cp zabbix_agentd.conf zabbix_agentd.conf.bak
```
配置文件示例：
```ini
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize= 1
DebugLevel=3
ListenPort=10050
ListenIP=0.0.0.0
Server=172.26.9.154
ServerActive=172.26.9.154
Hostname=centos82

EnableRemoteCommands=1
LogRemoteCommands=0
BufferSize=100
Timeout=30
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```
启动Agent：
```
[root@centos82 ~]# systemctl start zabbix-agent
[root@centos82 ~]# systemctl enable zabbix-agent
Created symlink /etc/systemd/system/multi-user.target.wants/zabbix-agent.service → /usr/lib/systemd/system/zabbix-agent.service.
[root@centos82 ~]# systemctl status zabbix-agent
● zabbix-agent.service - Zabbix Agent
   Loaded: loaded (/usr/lib/systemd/system/zabbix-agent.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-08-16 15:21:49 CST; 10s ago
 Main PID: 60606 (zabbix_agentd)
......
```
## 软件Agent配置
### 监控Nginx
检查确认是否有`with-http_stub_status_module`模块：
```
[root@centos82 ~]# nginx -V 2>&1 |grep -o with-http_stub_status_module
with-http_stub_status_module
```
编辑Nginx配置文件，添加如下内容：
```ini
location /status {
	stub_status;
	access_log off;
	allow 127.0.0.1;
	allow ::1;
	deny all;
}
```
监控脚本配置：
```
[root@centos82 nginx]# cat /etc/zabbix/zabbix_agentd.d/userparameter_nginx.conf
UserParameter=nginx.status[*],/bin/bash /opt/shell/devops/zabbix/scripts/status_nginx.sh $1
[root@centos82 nginx]# vim /opt/shell/devops/zabbix/scripts/status_nginx.sh
```
使用脚本检查Nginx线程数量示例：
```
[root@centos82 nginx]# /bin/bash /opt/shell/devops/zabbix/scripts/status_nginx.sh check
2
```
重启Zabbix-Agent，之前配置了自动注册，会自动监控Nginx。     
命令行获取值测试（开启被动模式）：
```
[root@centos82 nginx]# systemctl restart zabbix-agent
[root@centos82 nginx]# zabbix_get -s 172.26.9.154 -p 10050 -k nginx.status[check]
2
```
### 监控Redis
监控脚本配置：
```
[root@centos82 shell]# cat /etc/zabbix/zabbix_agentd.d/userparameter_redis.conf
UserParameter=Redis.Info[*],/bin/bash /opt/shell/devops/zabbix/scripts/redismonitor.sh $1 $2

UserParameter=Redis.Status,status1=`/soft/redis-stable/src/redis-cli -p 6379 -a 123456 ping 2>/dev/null|grep -c PONG` && echo $status1

UserParameter=Redis.Sentinel,status=`ps -ef |grep redis-sentinel |grep -v grep |wc -l` && echo $status
```
运行脚本检查示例：
```
[root@centos82 ~]# sh /opt/shell/devops/zabbix/scripts/redismonitor.sh version
7.0.12
[root@centos82 ~]# sh /opt/shell/devops/zabbix/scripts/redismonitor.sh ping
1
[root@centos82 ~]# sh /opt/shell/devops/zabbix/scripts/redismonitor.sh uptime
1686
```
zabbix_get命令检查验证：
```
[root@centos82 ~]# zabbix_get -s 172.26.9.154 -p 10050 -k Redis.Info[used_memory]
1387184
[root@centos82 ~]# zabbix_get -s 172.26.9.154 -p 10050 -k Redis.Info[db0,keys]
2000
```
默认没有Redis模板，添加一个模板：
- 进入主页左侧导航栏
- 选择`配置`
- 然后选择`模板`
- 右上角点击`导入`：
  - 浏览导入的文件
  - 勾选需求的规则
  - 点击`导入`确认，模板名未RedisMontior，群组名为RedisMontior
- 点击`导入`确认
- 添加一个自动注册动作：
  - 名称为Atuo-Register-Redis
  - 条件是：主机元数据包含redis-single
  - 操作：链接到模板RedisMontior
- 点击`添加`确认添加

Redis所在的主机已经监控了，选中主机：
- 右键，选择`配置`
- 添加模板RedisMontior
- 添加群组RedisMontior
- 点击`更新`确认

重启zabbix-agent服务，会自动添加监控项目，主机之前有121个监控项目，目前已经变成了140个了。

## 待补充
