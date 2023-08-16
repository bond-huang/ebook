# MySQL-安装与卸载
## 卸载MySQL
### 完全卸载MySQL
卸载不干净可能导致重新安装后服务服务启动。使用dnf安装的，卸载MySql：
```
[root@centos82 ~]# dnf remove mariadb
```
查找相关目录：
```
[root@centos82 ~]# find / -name mysql
/var/lib/selinux/targeted/active/modules/100/mysql
/var/lib/mysql
/var/lib/mysql/mysql
/usr/share/bash-completion/completions/mysql
/usr/share/selinux/targeted/default/active/modules/100/mysql
/usr/share/zabbix-sql-scripts/mysql
```
删除相关文件或目录：
```
[root@centos82 ~]# rm -rf /var/lib/mysql
[root@centos82 ~]# rm -rf /var/lib/mysql/mysql
[root@centos82 ~]# rm -rf /etc/my.cnf
[root@centos82 ~]# rm -rf /var/log/mysqld.log
```
再次安装后成功启动：
```
[root@centos82 ~]# dnf module install mariadb:10.5
[root@centos82 ~]# systemctl start mariadb.service
[root@centos82 ~]# systemctl status mariadb.service
● mariadb.service - MariaDB 10.5 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor p>
   Active: active (running) since Wed 2023-08-16 13:45:53 CST; 3s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 22088 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, >
  Process: 21994 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mariadb.servic>
  Process: 21969 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, st>
 Main PID: 22074 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 13 (limit: 11494)
   Memory: 75.1M
   CGroup: /system.slice/mariadb.service
           └─22074 /usr/libexec/mysqld --basedir=/usr
```
参考链接：[https://www.jianshu.com/p/ef58fb333cd6](https://www.jianshu.com/p/ef58fb333cd6)
## 待补充