# Redis-服务配置
## 模式说明
### 单节点模式
优点：
- 架构简单，部署方便；性价比高；高性能

缺点：
- 数据没冗余不可靠；
- 不适用于数据可靠性高的业务

### 主从模式
优点：
- 读写分离，效率高；数据有冗余，有多个副本

缺点：
- 主节点故障，集群无法工作，需要人工干预；
- 主节点只有一个，写的压力比较大

### 哨兵模式
优点：
- 对节点进行监控，自动切换

缺点：
- 切换等待时间可能比较长；
- 只有一个主节点对外提供服务，没法支持很高并发；
- 单个主节点内存不易设置过大，可能会导致持久化文件过大，影响数据恢复及主从同步

### 集群模式
优点：
- 无中心架构，多个master节点，写压力分散；
- 数据按照slot存储分布在多个节点，节点间数据共享，可以动态调整数据分布；
- 可扩展性，可线性扩展到一千多个节点；
- 高可用性，部分节点不可用时，集群依然可用，能够实现自动failover

缺点：
- 如果主节点A和其从节点A1都宕机了，集群将无法提供服务

参考链接：[https://blog.csdn.net/m0_45406092/article/details/116171758](https://blog.csdn.net/m0_45406092/article/details/116171758)

## 系统环境软件安装
计划使用系统信息如下：

系统名称|IP|系统版本
:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0
redhat9|192.168.100.133|RHEL9.0
redhat9a|192.168.100.138|RHEL9.0

用户均为redis，分别创建目录，并更改所属：
```
[root@redhat9 ~]#  mkdir -p /soft/redis
[root@redhat9 ~]# chown -R redis:redis /soft/redis
```
切换到redis用户，分别创建配置文件、数据存储、日志目录、PID以及安装目录：
```
[root@redhat9 ~]# su - redis
[redis@redhat9 ~]$ mkdir -p /soft/redis/conf
mkdir -p /soft/redis/data
mkdir -p /soft/redis/logs
mkdir -p /soft/redis/pid
mkdir -p /soft/redis/install/redis
[redis@redhat9 ~]$ ls -l  /soft/redis
total 0
drwxr-xr-x. 2 redis redis  6 Aug  3 22:39 conf
drwxr-xr-x. 2 redis redis  6 Aug  3 22:39 data
drwxr-xr-x. 3 redis redis 19 Aug  3 22:39 install
drwxr-xr-x. 2 redis redis  6 Aug  3 22:39 logs
drwxr-xr-x. 2 redis redis  6 Aug  3 22:39 pid
```
上传安装包到`/soft/redis/install/redis`下，再改下所属：
```
[root@redhat9 ~]# chown -R redis:redis /soft/redis
```
redis用户解压安装包：
```
[redis@redhat9 ~]$ cd /soft/redis/install/redis
[redis@redhat9 redis]$ tar zxvf redis-stable.tar.gz
```
redis用户执行编译安装：
```
[redis@redhat9 redis]$ cd redis-stable
[redis@redhat9 redis-stable]$ make && make install PREFIX=/soft/redis
```
用root用户设置一个链接或者添加环境变量，方便调用redis命令：
```
[root@redhat9 ~]# ln -s /soft/redis/bin/redis-cli /usr/local/bin/
```
分别创建三份配置文件，放在`/soft/redis/conf`目录下以系统IP尾数加字母命名：
- redis-134a.conf
- redis-133a.conf
- redis-138a.conf

`redis-134a.conf`配置示例：
```ini
######### NETWORK #########
bind 192.168.100.133
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
######### GENERAL #########
daemonize yes
supervised no
pidfile /soft/redis/pid/redis-133a.pid
loglevel notice
logfile "/soft/redis/logs/redis-133a.log"
databases 16
always-show-logo yes
######### SNAPSHOTTING #########
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump-133a.rdb
dir /soft/redis/data
######### REPLICATION #########
# masterauth {password}   
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
######### SECURITY #########
requirepass 123456
######### CLIENTS #########
######### MEMORY MANAGEMENT #########
maxmemory 1024m   
maxmemory-policy volatile-lru
# maxmemory-policy volatile-lfu
# maxmemory-policy noeviction
maxmemory-samples 5
# replica-ignore-maxmemory yes
######### LAZY FREEING #########     
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
######### APPEND ONLY MODE #########
appendonly yes
appendfilename "appendonly-6379.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
######### LUA SCRIPTING #########
lua-time-limit 5000
######### SLOW LOG #########
slowlog-log-slower-than 10000
slowlog-max-len 128
######### LATENCY MONITOR #########
latency-monitor-threshold 0
######### Event notification #########
notify-keyspace-events ""
######### ADVANCED CONFIG #########
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
```
其它两个系统的配置文件修改IP和文件名即可。使用redis用户启动服务，命令分别如下：
```sh
/soft/redis/bin/redis-server /soft/redis/conf/redis-134a.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-133a.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-138a.conf
```
分别验证是否正常，示例：
```
[redis@redhat9 ~]$ redis-cli -h 192.168.100.133 -p 6379 -a 123456 config get maxmemory
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "maxmemory"
2) "1024000000"
```
关闭redis，命令分别如下：
```sh
redis-cli -h 192.168.100.134 -p 6379 -a 123456 shutdown
redis-cli -h 192.168.100.133 -p 6379 -a 123456 shutdown
redis-cli -h 192.168.100.138 -p 6379 -a 123456 shutdown
```
## 单节点模式配置
单节点配置简单，上面三个独立节点就是。或者参考[Redis-基础配置]。
## 主从模式配置
计划使用系统信息如下：

系统名称|IP|系统版本|主从|端口
:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6379
redhat9|192.168.100.133|RHEL9.0|从|6379
redhat9a|192.168.100.138|RHEL9.0|从|6379



## 哨兵模式配置
## 集群模式配置