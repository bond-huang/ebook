# Redis-基础操作
## 安装启动
&#8195;&#8195;下载地址：[https://redis.io/download/](https://redis.io/download/)。下载最新稳定版本：redis-stable.tar.gz。下载后解压：
```
[root@centos82 soft]# tar -zxvf redis-stable.tar.gz
```
编译安装：
```
[root@centos82 soft]# cd redis-stable
[root@centos82 redis-stable]# make
[root@centos82 redis-stable]# make install
[root@centos82 redis-stable]# ./src/redis-server redis.conf
38550:C 01 Aug 2023 15:46:49.237 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
38550:C 01 Aug 2023 15:46:49.237 # Redis version=7.0.12, bits=64, commit=00000000, modified=0, pid=38550, just started
38550:C 01 Aug 2023 15:46:49.237 # Configuration loaded
38550:M 01 Aug 2023 15:46:49.237 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 7.0.12 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 38550
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
 ......
[root@centos82 /]# ps -ef |grep redis
root       38769   33539  0 16:13 pts/1    00:00:00 ./src/redis-server 127.0.0.                                               1:6379
root       38775   38600  0 16:13 pts/2    00:00:00 grep --color=auto redis
```
默认前台启动，一下参数设置为yes可以后台启动：
```
deamonize yes
```
## 基础操作
进入命令行：
```
[root@centos82 redis-stable]# ./src/redis-cli -p 6379 -h 127.0.0.1
127.0.0.1:6379> 
```
### 配置查看修改
查看内存配置：
```
127.0.0.1:6379> info memory
# Memory
used_memory:931720
...
used_memory_dataset_perc:97.28%
allocator_allocated:1312048
...
total_system_memory_human:1.78G
...
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
...
```
查看客户端信息：
```
127.0.0.1:6379> info clients
# Clients
connected_clients:1
cluster_connections:0
maxclients:10000
client_recent_max_input_buffer:20480
client_recent_max_output_buffer:0
blocked_clients:0
tracking_clients:0
clients_in_timeout_table:0
```
查看某项配置：
```
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "0"
```
在线修改配置：
```
127.0.0.1:6379> config set maxmemory 1073741824
OK
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "1073741824"
```
修改完成后立即修改配置文件，重启后以配置文件为准。关闭连接：
```
127.0.0.1:6379> quit
[root@centos82 redis-stable]#
```
### 关闭Redis
关闭redis：
```
[root@centos82 redis-stable]# ./src/redis-cli -p 6379 -h 127.0.0.1
127.0.0.1:6379> shutdown
not connected>quit
[root@centos82 redis-stable]#
```
查看端口使用情况：
```
[root@centos82 redis-stable]# netstat -tunpl |grep 6379
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      45800/./src/redis-s
tcp6       0      0 ::1:6379                :::*                    LISTEN      45800/./src/redis-s
```
### 从配置文件启动
创建conf目录，下面创建一个配置文件，名为redis_7012.conf，从配置文件启动：
```
[root@centos82 redis-stable]# chown -R redis.redis conf
[redis@centos82 redis-stable]$ ./src/redis-server ./conf/redis_7012.conf
[root@centos82 ~]# netstat -tunpl |grep 6379
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      47118/./src/redis-s
tcp        0      0 172.26.9.154:6379       0.0.0.0:*               LISTEN      47118/./src/redis-s
```
### 数据操作
设置键(key)为"username"，值(value)为"Thor"。然后使用GET命令获取该键的值：
```
127.0.0.1:6379> set username Thor
OK
127.0.0.1:6379> GET username
"Thor"
```
删除数据：
```
127.0.0.1:6379> DEL username
(integer) 1
```
清空当前库所有keys：
```
192.168.100.134:6500> info keyspace
# Keyspace
db0:keys=672,expires=0,avg_ttl=0
192.168.100.134:6500> flushdb
OK
192.168.100.134:6500> info keyspace
# Keyspace
```
注意：高危操作，需谨慎！
## 安全配置
### 密码配置
V6.0版本前，可以在配置文件中`requirepass`项设置用户访问密码，示例：
```ini
requirepass 123456
```
查看配置：
```
192.168.100.134:6500> config get requirepass
1) "requirepass"
2) "123456"
```
在线修改：
```
192.168.100.134:6500> config set requirepass "654321"
OK
192.168.100.134:6500> config get requirepass
1) "requirepass"
2) "654321"
```
在线修改不会写入到配置文件中。
### ACL机制配置
&#8195;&#8195;Redis6.0发布了权限管理功能`ACL`(access control list访问控制列表)，可以根据不同的用户设置不同的权限，限制用户访问命令和待访问的数据。查看用户权限列表：
```
192.168.100.134:6500> acl list
1) "user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all"
```
查看当前使用用户：
```
192.168.100.134:6500> acl whoami
"default"
```
创建一个用户示例：
```
192.168.100.134:6500> acl setuser batman on >123456 ~superh* +@string +get
OK
192.168.100.134:6500> acl list
1) "user batman on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~superh* resetchannels -@all +@string"
2) "user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all"
```
示例说明：
- `user`：表示是用户
- `batman`：用户名, `default`是系统默认用户名
- `on`：表示启用用户，默认为`off`(禁用)
- `#...`：用户密码, `nopass`表示不需要密码
- `~*`：可访问的数据Key(正则匹配)
     - `~superh*`表示可以访问`superh`开头的key
- `+@all`：表示用户的权限：
     - `+`添加权限，`-`删减权限;
     -  `@`为redis命令分类, 可以通过ACL CAT查询相关分类
     - `+@all`表示拥有所有权限
     - `+@string +get`：表示只对`string`类型的数据具有`get`权限

redis命令分类：
```
192.168.100.134:6500> acl cat
 1) "keyspace"
 2) "read"
 3) "write"
 4) "set"
 5) "sortedset"
 6) "list"
 7) "hash"
 8) "string"
 9) "bitmap"
10) "hyperloglog"
11) "geo"
12) "stream"
13) "pubsub"
14) "admin"
15) "fast"
16) "slow"
17) "blocking"
18) "dangerous"
19) "connection"
20) "transaction"
21) "scripting"
```
示例batman用户get相关数据：
```
192.168.100.134:6500> get supergod_1000
(error) NOPERM this user has no permissions to access one of the keys used as arguments
192.168.100.134:6500> get superhero_100
-> Redirected to slot [12369] located at 192.168.100.138:6500
"batman_100"
```
切换用户：
```
192.168.100.134:6500> auth batman 123456
OK
```
删除用户：
```
192.168.100.134:6500> acl deluser batman
(integer) 1
```
## 工具安装
### Windows管理平台
Windows下有一款Another Redis Desktop Manager工具，打开PowerShell：
```
PS C:\Users\admin> winget install qishibo.AnotherRedisDesktopManager
```
或者直接复制链接到浏览器进行下载：https://github.com/qishibo/AnotherRedisDesktopManager/releases/download/v1.5.9/Another-Redis-Desktop-Manager.1.5.9.exe

## 待补充
