# Zabbix-常见问题
## 问题排查步骤
如有host监控信息不更新，状态异常，可以按照以下步骤进行问题排查：
- 检查主机配置：确保被监控主机的Zabbix Agent配置正确。检查配置文件是否包含正确的Zabbix服务器地址、端口和主机名称等信息。
- 检查防火墙和端口：确保Zabbix Agent的端口（通常是10050）在被监控主机上是开放的，并且防火墙允许Zabbix服务器与Agent之间的通信。
- 检查Zabbix Agent运行状态：确认Zabbix Agent是否正在运行。你可以通过查看进程列表或日志文件来确认。
- 检查Zabbix服务器配置：检查Zabbix服务器的配置，确保主机被正确地添加到监控系统中，且监控项和触发器等设置正确。
- 检查监控项和触发器配置：确保你为被监控主机配置了正确的监控项和触发器。如果设置不正确，可能导致数据无法正确显示。
- 检查触发器状态：检查触发器是否已启用。如果触发器没有启用，可能会导致数据无法触发报警。
- 检查日志信息：查看Zabbix Agent和Zabbix服务器的日志，以获取更多有关通信和错误的信息。
- 测试连接：在被监控主机上执行一些手动测试，如使用telnet命令测试Zabbix Agent的端口连接，确保通信正常。
- 尝试重启服务：有时，重启Zabbix Agent和Zabbix服务器服务可能会解决通信问题。
- 检查主机状态：确保被监控主机本身的网络和系统状态正常，以免影响数据传输。

## Zabbix-server问题
### Zabbix服务不停重启
查看Zabbix服务状态如下：
```
[root@centos82 zabbix]# systemctl status zabbix-server
● zabbix-server.service - Zabbix Server
   Loaded: loaded (/usr/lib/systemd/system/zabbix-server.service; enabled; vendor preset: disabled)
   Active: activating (auto-restart) (Result: exit-code) since Wed 2023-08-16 14:08:26 CST; 7s ago
  Process: 50916 ExecStop=/bin/kill -SIGTERM $MAINPID (code=exited, status=1/FAILURE)
  Process: 50639 ExecStart=/usr/sbin/zabbix_server -c $CONFFILE (code=exited, status=0/SUCCESS)
 Main PID: 50641 (code=exited, status=0/SUCCESS)
```
查看日志：
```
[root@centos82 zabbix]# pwd
/var/log/zabbix
[root@centos82 zabbix]# tail -n 100 zabbix_server.log
......
 53984:20230816:141022.518 Cannot connect to the database. Exiting...
 53987:20230816:141022.518 [Z3001] connection to database 'zabbix' failed: [1040] Too many connections
 53987:20230816:141022.518 Cannot connect to the database. Exiting...
 53747:20230816:141022.519 HA manager has been paused
 53747:20230816:141022.577 HA manager has been stopped
 53746:20230816:141022.580 syncing trend data...
 53746:20230816:141022.580 syncing trend data done
 53746:20230816:141022.582 Zabbix Server stopped. Zabbix 6.0.20 (revision b60e269feb4).
```
数据库连接池太少了，修改配置文件`/etc/my.cnf` ，在mysqld下添加：
```
[mysqld]
max_connections=4096
```
查看确认：
```
[root@centos82 etc]# mysql -uroot -p
Enter password:
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
MariaDB [(none)]> show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 4096  |
+-----------------+-------+
1 row in set (0.001 sec)
```
## 服务启动问题
### 启动httpd服务报错
报错示例：
```
[root@centos82 zabbix]# systemctl start httpd
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xe" for details.
```
原因是80端口被Nginx已经使用了，换个端口监听，修改配置文件：
```
[root@centos82 zabbix]# vi /etc/httpd/conf/httpd.conf
```
### 启动zabbix-server服务报错
报错示例：
```
[root@centos82 zabbix]# systemctl start zabbix-server
Job for zabbix-server.service failed because the service did not take the steps required by its unit configuration.
See "systemctl status zabbix-server.service" and "journalctl -xe" for details.
```
查看日志`/var/log/zabbix/zabbix_server.log`提示数据库版本不支持：
```
 10744:20230816:121103.107 Unable to start Zabbix server due to unsupported MariaDB database version (10.03.28).
 10744:20230816:121103.107 Must be at least (10.05.00).
```
查看当前数据库版本：
```
[root@centos82 zabbix]# mysql -V
mysql  Ver 15.1 Distrib 10.3.28-MariaDB, for Linux (x86_64) using readline 5.1
```
安装前完全卸载之前版本，安装指定版本：
```
[root@centos82 ~]# dnf module list mariadb
Last metadata expiration check: 2:54:02 ago on Wed 16 Aug 2023 10:14:37 AM CST.
CentOS-8 - AppStream
Name         Stream           Profiles                       Summary
mariadb      10.3 [d][e]      client, galera, server [d]     MariaDB Module
mariadb      10.5             client, galera, server [d]     MariaDB Module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[root@centos82 ~]# dnf remove  mariadb
[root@centos82 ~]# dnf module reset mariadb
[root@centos82 ~]# dnf module list mariadb
Last metadata expiration check: 3:00:14 ago on Wed 16 Aug 2023 10:14:37 AM CST.
CentOS-8 - AppStream
Name          Stream        Profiles                        Summary
mariadb       10.3 [d]      client, galera, server [d]      MariaDB Module
mariadb       10.5          client, galera, server [d]      MariaDB Module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[root@centos82 ~]# dnf module install mariadb:10.5
......
Installed:
  mariadb-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-backup-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-common-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-errmsg-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-gssapi-server-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-server-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mariadb-server-utils-3:10.5.9-1.module_el8.4.0+801+647c4915.x86_64
  mysql-selinux-1.0.2-4.el8.noarch
  perl-DBD-MySQL-4.046-3.module_el8.3.0+419+c2dec72b.x86_64
  perl-DBI-1.641-3.module_el8.3.0+413+9be2aeb5.x86_64
  perl-Math-BigInt-1:1.9998.11-7.el8.noarch
  perl-Math-Complex-1.59-420.el8.noarch

Complete!
[root@centos82 ~]# mysql -V
mysql  Ver 15.1 Distrib 10.5.9-MariaDB, for Linux (x86_64) using  EditLine wrapper
```
### MySql启动问题
MySql启动报错：
```
[root@centos82 mariadb]# systemctl status mariadb
● mariadb.service - MariaDB 10.5 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2023-08-16 14:29:31 CST; 3min 19s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 22074
   CGroup: /system.slice/mariadb.service

Aug 16 14:29:31 centos82 systemd[1]: mariadb.service: Found left-over process 22074 (mysqld) in control group while starting unit. Ign>
Aug 16 14:29:31 centos82 systemd[1]: This usually indicates unclean termination of a previous run, or service implementation deficienc>
Aug 16 14:29:31 centos82 systemd[1]: Starting MariaDB 10.5 database server...
Aug 16 14:29:31 centos82 mysql-check-socket[58794]: Socket file /var/lib/mysql/mysql.sock exists.
Aug 16 14:29:31 centos82 mysqld[22074]: 2023-08-16 14:29:31 22593 [Warning] Access denied for user 'UNKNOWN_MYSQL_USER'@'localhost'
Aug 16 14:29:31 centos82 mysql-check-socket[58794]: Is another MySQL daemon already running with the same unix socket?
Aug 16 14:29:31 centos82 mysql-check-socket[58794]: Please, stop the process using the socket /var/lib/mysql/mysql.sock or remove the >
Aug 16 14:29:31 centos82 systemd[1]: mariadb.service: Control process exited, code=exited status=1
Aug 16 14:29:31 centos82 systemd[1]: mariadb.service: Failed with result 'exit-code'.
Aug 16 14:29:31 centos82 systemd[1]: Failed to start MariaDB 10.5 database server.
```
检查是否有已经运行的进程：
```
[root@centos82 mariadb]# ps -ef |grep mysql
mysql      22074       1  1 13:45 ?        00:00:31 /usr/libexec/mysqld --basedir=/usr
root       58874   13438  0 14:32 pts/3    00:00:00 grep --color=auto mysql
[root@centos82 mariadb]# kill -9 22074
```
## Zabbix连接host问题
### 连接访问受限
错误示例：
```
127.0.0.1:10050 不可用	
Received empty response from Zabbix Agent at [127.0.0.1]. Assuming that agent dropped connection because of access permissions.
```
更改成IP地址和主机名称后一样不行，查看server日志（/var/log/zabbix/zabbix_server.log）：
```
 59263:20230816:151148.272 housekeeper [deleted 0 hist/trends, 0 items/triggers, 0 events, 0 problems, 0 sessions, 0 alarms, 0 audit, 0 records in 0.008169 sec, idle for 1 hour(s)]
 59666:20230816:152149.401 cannot send list of active checks to "172.26.9.154": host [centos82] not found
 59666:20230816:152349.417 cannot send list of active checks to "172.26.9.154": host [centos82] not found
```
重启Agent，状态恢复正常。
## 待补充