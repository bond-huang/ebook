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
appendfilename "appendonly-133a.aof"
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
### 系统配置计划
计划使用系统信息如下：

系统名称|IP|系统版本|主从|端口
:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6379
redhat9|192.168.100.133|RHEL9.0|从|6379
redhat9a|192.168.100.138|RHEL9.0|从|6379

### 主从模式配置步骤
redhat8注释的配置放开：
```
masterauth 123456
```
redhat9和rehat9a配置文件`REPLICATION`项中分别加入：：
```
replicaof 192.168.100.134 6379
masterauth 123456
```
启动主节点：
```sh
/soft/redis/bin/redis-server /soft/redis/conf/redis-134a.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-133a.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-138a.conf
```
主节点查看主从关系：
```
192.168.100.134:6379> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.100.133,port=6379,state=online,offset=182,lag=0
slave1:ip=192.168.100.138,port=6379,state=online,offset=182,lag=1
master_failover_state:no-failover
master_replid:ebc73b7a8416af20bc46c811ecdfb1307f9fd9b1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:182
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:182
```
从节点查看主从关系配置：
```
192.168.100.133:6379> info Replication
# Replication
role:slave
master_host:192.168.100.134
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_read_repl_offset:917
slave_repl_offset:917
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:ebc73b7a8416af20bc46c811ecdfb1307f9fd9b1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:917
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:917
```
主节点写入一条数据：
```
192.168.100.134:6379> set superhero Batman
OK
192.168.100.134:6379> GET superhero
"Batman"
```
redhat9从节点查看数据：
```
192.168.100.133:6379> get superhero
"Batman"
```
redhat9a从节点查看数据：
```
192.168.100.138:6379> get superhero
"Batman"
```
从节点删除数据：
```
192.168.100.133:6379> del superhero
(error) READONLY You can't write against a read only replica.
```
主节点删除数据：
```
192.168.100.134:6379> del superhero
(integer) 1
192.168.100.134:6379> GET superhero
(nil)
```
## 哨兵模式配置
### 系统配置计划
配置一主两从三哨兵，在上面一主两从配置基础上进行配置，使用系统信息如下：

系统名称|IP|系统版本|主从|服务端口|哨兵端口
:---:|:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6379|6380
redhat9|192.168.100.133|RHEL9.0|从|6379|6380
redhat9a|192.168.100.138|RHEL9.0|从|6379|6380

### 哨兵模式配置步骤
分别创建三份配置文件，放在`/soft/redis/conf`目录下：
- sentinel-134a.conf
- sentinel-133a.conf
- sentinel-138a.conf

主节点配置文件`sentinel-134a.conf`内容示例：
```ini
port 6380
daemonize yes
logfile "/soft/redis/logs/sentinel-134a.log"
pidfile "/soft/redis/pid/sentinel-134a.pid"
dir "/soft/redis/sentinel_db"
sentinel deny-scripts-reconfig yes
sentinel monitor redisMaster 192.168.100.134 6379 2
sentinel down-after-milliseconds redisMaster 300
sentinel failover-timeout redisMaster 1500
sentinel auth-pass redisMaster 123456
```
&#8195;&#8195;其它节点配置文件更改log和pid文件名即可。注意目录是否存在，其它目录已在，三个节点分别创建"/soft/redis/sentinel_db"：
```
[redis@redhat9a ~]$ mkdir -p /soft/redis/sentinel_db
```
分别启动三个节点Redis服务，然后启动哨兵，命令分别如下：
```shell
/soft/redis/bin/redis-sentinel /soft/redis/conf/sentinel-134a.conf
/soft/redis/bin/redis-sentinel /soft/redis/conf/sentinel-133a.conf 
/soft/redis/bin/redis-sentinel /soft/redis/conf/sentinel-138a.conf 
```
启动后查看进程：
```
[redis@redhat8 ~]$ ps -ef |grep redis
redis     19339      1  0 07:15 ?        00:00:11 /soft/redis/bin/redis-server 192.168.100.134:6379
redis     21102      1  0 08:59 ?        00:00:00 /soft/redis/bin/redis-sentinel *:6380 [sentinel]
```
查看当前主从信息：
```
192.168.100.134:6379> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.100.133,port=6379,state=online,offset=54922,lag=0
slave1:ip=192.168.100.138,port=6379,state=online,offset=54922,lag=1
master_failover_state:no-failover
master_replid:ebc73b7a8416af20bc46c811ecdfb1307f9fd9b1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:55216
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:55216
```
手动停掉主节点，尝试切换：
```
192.168.100.134:6379> shutdown
not connected> quit
[redis@redhat8 ~]$ ps -ef |grep redis
redis     21102      1  0 08:59 ?        00:00:02 /soft/redis/bin/redis-sentinel *:6380 [sentinel]
redis     21190  17495  0 09:05 pts/1    00:00:00 ps -ef
redis     21191  17495  0 09:05 pts/1    00:00:00 grep --color=auto redis
```
主节点变成`192.168.100.138:6379`，查看信息：
```
192.168.100.138:6379> info Replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.100.133,port=6379,state=online,offset=106896,lag=0
master_failover_state:no-failover
master_replid:253b05b1572819283db74e1ad62b6452f83fc11e
master_replid2:5f0ad0b2d7370ae307c24d02f32158a9155833c2
master_repl_offset:107043
second_repl_offset:82309
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:183
repl_backlog_histlen:106861
```
写入一条数据：
```
192.168.100.138:6379> set superhero Batman
OK
192.168.100.138:6379> get superhero
"Batman"
```
重新启动`192.168.100.134:6379`，并查看数据:
```
[redis@redhat8 ~]$ /soft/redis/bin/redis-server /soft/redis/conf/redis-134a.conf
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6379 -a 123456
192.168.100.134:6379> get superhero
"Batman"
```
主节点还是`192.168.100.138:6379`，信息如下：
```
192.168.100.138:6379> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.100.133,port=6379,state=online,offset=145284,lag=0
slave1:ip=192.168.100.131,port=6379,state=online,offset=145284,lag=0
master_failover_state:no-failover
master_replid:253b05b1572819283db74e1ad62b6452f83fc11e
master_replid2:5f0ad0b2d7370ae307c24d02f32158a9155833c2
master_repl_offset:145284
second_repl_offset:82309
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:183
repl_backlog_histlen:145102
```
说明：
- 显示192.168.100.131是因为我的redhat8系统中网卡是双IP配置，跟192.168.100.134在同一个网卡。

关闭哨兵模式，分别执行：
```shell
redis-cli -h 192.168.100.134 -p 6380 shutdown
redis-cli -h 192.168.100.133 -p 6380 shutdown
redis-cli -h 192.168.100.138 -p 6380 shutdown
```
## 集群模式配置
### 系统配置计划
配置三主三从，使用三个系统，各运行两个redis节点，使用系统信息如下：

系统名称|IP|系统版本|主从|服务端口|配置文件
:---:|:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6500|redis-134b.conf
redhat8|192.168.100.134|RHEL8.0|从|6501|redis-134c.conf
redhat9|192.168.100.133|RHEL9.0|主|6500|redis-133b.conf
redhat9|192.168.100.133|RHEL9.0|从|6501|redis-133c.conf
redhat9a|192.168.100.138|RHEL9.0|主|6500|redis-138b.conf
redhat9a|192.168.100.138|RHEL9.0|从|6501|redis-138c.conf

### 集群模式配置步骤
每个系统上分别创建六个配置文件：
- redis-134b.conf,redis-134c.conf
- redis-133b.conf,redis-133c.conf
- redis-138b.conf,redis-138c.conf

为了和之前区别，集群相关目录修改下，每个节点分别创建目录，命令如下：
```shell
mkdir -p /soft/redis/cluster/{pid,logs,data}
```
配置文件每个差不多，注意事项：
- 端口与配置文件对应关系
- 密码
- 文件路径和文件名称
- 集群配置信息

每个配置文件中添加集群配置信息，138c为例：
```
######### CLUSTER #########
cluster-enabled yes
cluster-config-file nodes-138c.conf
cluster-node-timeout 15000
```
配置完成后启动6个节点：
```shell
/soft/redis/bin/redis-server /soft/redis/conf/redis-134b.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-134c.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-133b.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-133c.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-138b.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-138c.conf
```
启动后查看进程示例：
```
[redis@redhat8 ~]$ /soft/redis/bin/redis-server /soft/redis/conf/redis-134b.conf
[redis@redhat8 ~]$ /soft/redis/bin/redis-server /soft/redis/conf/redis-134c.conf
[redis@redhat8 ~]$ ps -ef |grep redis
redis     21979      1  0 10:10 ?        00:00:00 /soft/redis/bin/redis-server 192.168.100.134:6500 [cluster]
redis     21985      1  0 10:11 ?        00:00:00 /soft/redis/bin/redis-server 192.168.100.134:6501 [cluster]
redis     21993  17495  0 10:11 pts/1    00:00:00 grep --color=auto redis
```
创建集群，命令如下：
```shell
redis-cli -a 123456 --cluster create \
192.168.100.134:6500 192.168.100.133:6500 192.168.100.138:6500 \
192.168.100.134:6501 192.168.100.133:6501 192.168.100.138:6501 \
--cluster-replicas 1
```
说明：
- 前面三个是主，后面三个是从
- `--cluster-replicas 1`表示一主一丛，主从配置比为1

执行命令创建集群：
```
[redis@redhat8 ~]$ redis-cli -a 123456 --cluster create \
> 192.168.100.134:6500 192.168.100.133:6500 192.168.100.138:6500 \
> 192.168.100.134:6501 192.168.100.133:6501 192.168.100.138:6501 \
> --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.100.133:6501 to 192.168.100.134:6500
Adding replica 192.168.100.138:6501 to 192.168.100.133:6500
Adding replica 192.168.100.134:6501 to 192.168.100.138:6500
M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots:[0-5460] (5461 slots) master
M: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots:[5461-10922] (5462 slots) master
M: e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500
   slots:[10923-16383] (5461 slots) master
S: ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501
   replicates e9451337dd0bdd3923adda404089953065fa6712
S: fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501
   replicates d26627581602fc00c2b5f249c31dc831cee01217
S: b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501
   replicates 6541e11615d81e677f6ebba0b4d4d3df0772ca81
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.....
>>> Performing Cluster Check (using node 192.168.100.134:6500)
M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501
   slots: (0 slots) slave
   replicates d26627581602fc00c2b5f249c31dc831cee01217
M: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501
   slots: (0 slots) slave
   replicates e9451337dd0bdd3923adda404089953065fa6712
S: b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501
   slots: (0 slots) slave
   replicates 6541e11615d81e677f6ebba0b4d4d3df0772ca81
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
很快创建完成，查看集群状态：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 -c cluster info
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:77
cluster_stats_messages_pong_sent:28
cluster_stats_messages_sent:105
cluster_stats_messages_ping_received:23
cluster_stats_messages_pong_received:77
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:105
total_cluster_links_buffer_limit_exceeded:0
```
查看集群节点信息：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 -c cluster node
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(error) ERR unknown subcommand 'node'. Try CLUSTER HELP.
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 -c cluster nodes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500@16500 master - 0 1691147189799 3 connected 10923-16383
fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501@16501 slave d26627581602fc00c2b5f249c31dc831cee01217 0 1691147187000 1 connected
6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500@16500 master - 0 1691147188000 2 connected 5461-10922
ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501@16501 slave e9451337dd0bdd3923adda404089953065fa6712 0 1691147187783 3 connected
b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501@16501 slave 6541e11615d81e677f6ebba0b4d4d3df0772ca81 0 1691147188791 2 connected
d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500@16500 myself,master - 0 1691147185000 1 connected 0-5460
```
检查集群状态：
```
[redis@redhat8 ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.100.134:6500)
M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501
   slots: (0 slots) slave
   replicates d26627581602fc00c2b5f249c31dc831cee01217
M: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501
   slots: (0 slots) slave
   replicates e9451337dd0bdd3923adda404089953065fa6712
S: b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501
   slots: (0 slots) slave
   replicates 6541e11615d81e677f6ebba0b4d4d3df0772ca81
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```