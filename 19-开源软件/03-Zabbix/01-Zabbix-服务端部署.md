# Zabbix-服务端部署
## 服务端安装部署
### 环境准备
下载依次选择：
- ZABBIX版本：6.0 LTS
- OS分布：CentOS
- OS版本：8 Stream
- ZABBIX：Server,Frontent,Agent
- 数据库：MySQL
- WEB SERVER：Apache

检查防火墙和selinux状态：
```
[root@centos82 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor>
   Active: inactive (dead)
     Docs: man:firewalld(1)

[root@centos82 ~]# getenforce
Disabled
```
### 安装Zabbix
安装Zabbix仓库：
```
[root@centos82 ~]# rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/za                                               bbix-release-6.0-4.el8.noarch.rpm
Retrieving https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-                                               4.el8.noarch.rpm
warning: /var/tmp/rpm-tmp.dvaPG0: Header V4 RSA/SHA512 Signature, key ID a14fe5                                               91: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:zabbix-release-6.0-4.el8         ################################# [100%]
[root@centos82 ~]# dnf clean all
27 files removed
```
安装Zabbix server，Web前端，Agent等：
```
[root@centos82 ~]# dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
......
Installed:
  OpenIPMI-libs-2.0.31-3.el8.x86_64
  apr-1.6.3-12.el8.x86_64
  apr-util-1.6.1-6.el8.x86_64
  apr-util-bdb-1.6.1-6.el8.x86_64
  apr-util-openssl-1.6.1-6.el8.x86_64
  centos-logos-httpd-85.8-2.el8.noarch
  dejavu-fonts-common-2.35-7.el8.noarch
  dejavu-sans-fonts-2.35-7.el8.noarch
  fping-4.2-2.el8.x86_64
  httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch
  httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  libtool-ltdl-2.4.6-25.el8.x86_64
  mariadb-connector-c-3.1.11-2.el8_3.x86_64
  mariadb-connector-c-config-3.1.11-2.el8_3.noarch
  mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64
  net-snmp-libs-1:5.8-22.el8.x86_64
  php-bcmath-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-gd-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-json-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-ldap-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-mbstring-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-mysqlnd-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-pdo-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-xml-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  unixODBC-2.3.7-1.el8.x86_64
  zabbix-agent-6.0.20-release1.el8.x86_64
  zabbix-apache-conf-6.0.20-release1.el8.noarch
  zabbix-selinux-policy-6.0.20-release1.el8.x86_64
  zabbix-server-mysql-6.0.20-release1.el8.x86_64
  zabbix-sql-scripts-6.0.20-release1.el8.noarch
  zabbix-web-6.0.20-release1.el8.noarch
  zabbix-web-deps-6.0.20-release1.el8.noarch
  zabbix-web-mysql-6.0.20-release1.el8.noarch

Complete!
```
发现没有MySQL命令，安装少了几个包：
```
[root@centos82 ~]# dnf install zabbix-server  mariadb-server
Installed:
  mariadb-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  mariadb-backup-3: -1.module_el8.3.0+757+d382997d.x86_64
  mariadb-common-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  mariadb-errmsg-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  mariadb-gssapi-server-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  mariadb-server-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  mariadb-server-utils-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64
  perl-DBD-MySQL-4.046-3.module_el8.3.0+419+c2dec72b.x86_64
  perl-DBI-1.641-3.module_el8.3.0+413+9be2aeb5.x86_64
  perl-Math-BigInt-1:1.9998.11-7.el8.noarch
  perl-Math-Complex-1.59-420.el8.noarch

Complete!
```
### MySql配置
启动数据库服务：
```
[root@centos82 ~]# systemctl start mariadb
[root@centos82 ~]# systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[root@centos82 ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor pr>
   Active: inactive (dead)
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 10166 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─10166 /usr/libexec/mysqld --basedir=/usr

Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: See the MariaDB Knowledg>
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: MySQL manual for more in>
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: Please report any proble>
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: The latest information a>
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: You can find additional >
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: http://dev.mysql.com
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: Consider joining MariaDB>
Aug 16 11:41:26 centos82 mysql-prepare-db-dir[10061]: https://mariadb.org/get->
Aug 16 11:41:26 centos82 mysqld[10166]: 2023-08-16 11:41:26 0 [Note] /usr/libe>
Aug 16 11:41:26 centos82 systemd[1]: Started MariaDB 10.3 database server.
```
初始化数据库，设置密码，步骤全部Y：
```
[root@centos82 ~]# mysql_secure_installation
......
Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
继续配置数据库：
```
[root@centos82 ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 26
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all on zabbix.* to zabbix@localhost identified by "passw0rd";
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> set global log_bin_trust_function_creators = 1;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> quit;
Bye
```
导入初始架构和数据，系统将提示输入新创建的密码。
```
[root@centos82 ~]# zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8 -uzabbix -p zabbix
Enter password:
```
导入数据库架构后禁用`log_bin_trust_function_creators`选项：
```
[root@centos82 ~]# mysql -uroot -p
Enter password:
MariaDB [(none)]> set global log_bin_trust_function_creators = 0;
Query OK, 0 rows affected (0.000 sec)
MariaDB [(none)]> quit;
Bye
```
修改最大连接数，修改配置文件`/etc/my.cnf` ，在mysqld下添加：
```ini
[mysqld]
max_connections=4096
```
### zabbix配置
备份zabbix配置文件：
```
[root@centos82 zabbix]# cp zabbix_server.conf zabbix_server.conf.bak
[root@centos82 zabbix]# pwd
/etc/zabbix
```
&#8195;&#8195;根据需求写入配置文件。调整mariadb 启动文件，文件路径`/usr/lib/systemd/system/mariadb.service`。添加示例如下：
```
[Service]
LimitNOFILE=10000
LimitNPROC=10000
```
完成后启动Server：
```
[root@centos82 zabbix]# systemctl restart zabbix-server
[root@centos82 zabbix]# systemctl enable zabbix-server
Created symlink /etc/systemd/system/multi-user.target.wants/zabbix-server.service → /usr/lib/systemd/system/zabbix-server.service.
[root@centos82 zabbix]# systemctl status zabbix-server
● zabbix-server.service - Zabbix Server
   Loaded: loaded (/usr/lib/systemd/system/zabbix-server.service; enabled; ven>
   Active: active (running) since Wed 2023-08-16 10:52:52 CST; 30s ago
 Main PID: 6731 (zabbix_server)
    Tasks: 1 (limit: 11494)
   Memory: 3.1M
   CGroup: /system.slice/zabbix-server.service
           └─6731 /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf

Aug 16 10:52:52 centos82 systemd[1]: Starting Zabbix Server...
Aug 16 10:52:52 centos82 systemd[1]: Started Zabbix Server.
```
更改时区：

```shell
sed -i s'/.*date.timezone.*/\tphp_value date\.timezone Asia\/Shanghai/' /etc/httpd/conf.d/zabbix.conf
sed -i '/^date.timezone/d' /etc/php.ini
sed -i '/^; date.timezone.*/a\date.timezone = Asia/Shanghai' /etc/php.ini
```
启动httpd服务：
```
[root@centos82 zabbix]# systemctl start httpd
[root@centos82 zabbix]# systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```
启动成功后可以访问Zabbix主页：[http://121.43.191.157:82/zabbix/setup.php](http://121.43.191.157:82/zabbix/setup.php)

配置完成后默认用户Admin，默认密码zabbix。
## 工具安装
安装zabbix-get：
```
[root@centos82 ~]# dnf install zabbix-get
```
使用示例：
```
[root@centos82 ~]# zabbix_get -s 172.26.9.154 -p 10050 -k "system.uname"
Linux centos82 4.18.0-193.14.2.el8_2.x86_64 #1 SMP Sun Jul 26 03:54:29 UTC 2020 x86_64
```
使用说明：
- `-s`：客户端的IP  
- `-p`：客户端端口，默认10050     
- `-k`：监控项的关键字

## 待补充