# Redis-配置文件详解
## 基础配置参数

## 多节点模式参数
### 主从复制模式配置
主从模式配置示例及说明：
```ini
######### REPLICATION #########
masterauth 123456  ## 主节点密码
replica-serve-stale-data yes  
## 当slave与master失去通讯时，slave是否继续为客户端提供服务，yes表示继续，no表示终止
replica-read-only yes  ## slave节点仅读取
repl-diskless-sync no
## 是否开启无磁盘交互模式，默认为no。no时则Redis从库会将数据文件放入磁盘中，然后再读入内存
repl-diskless-sync-delay 5  ## 复制延迟时间（秒）
repl-disable-tcp-nodelay no
## 设置成yes，则redis会合并小的TCP包从而节省带宽，但会增加同步延迟（40ms），造成master与slave数据不一致。
## 设置成no，则redis master会立即发送同步数据，没有延迟
```
### 哨兵模式配置
配置示例：
```ini
port 6380  ## 端口
daemonize yes ## 后台启动
logfile "/soft/redis/logs/sentinel-134a.log" ## 日志位置
pidfile "/soft/redis/pid/sentinel-134a.pid" ## Pid位置
dir "/soft/redis/sentinel_db" ## 数据存放目录
sentinel deny-scripts-reconfig yes ## 拒绝重复执行脚本
sentinel monitor redisMaster 192.168.100.134 6379 2 
## redisMaster是自定义名称，下同。IP是Master IP，2表示至少两个哨兵
sentinel down-after-milliseconds redisMaster 300
## 主节点宕机多长时间后执行切换
sentinel failover-timeout redisMaster 60000
## 切换等待时间
sentinel auth-pass redisMaster 123456
## 配置密码，根据配置文件的reqirepass值配置
replica-priority 100
## 这是replicas节点通过INFO接口给出的信息，默认值为100。当master节点无法正常工作后Redis Sentinel通过这个值来决定将哪个replicas节点提升为master节点
## 这个数值越小表示越优先进行提升。这个值为0表示replica节点永远不能被提升为master节点
```
### 集群模式参数
基础配置示例：
```ini
######### CLUSTER #########
cluster-enabled yes   ## 开启集群
cluster-config-file nodes-133b.conf  ## 集群配置文件，默认在data目录中自动创建
cluster-node-timeout 15000  ## 集群节点超时时间
```
集群更多配置：
```ini
cluster-replica-validity-factor 10 
## 子节点是否过载，是否能成为主节点的可能性
cluster-migration-barrier 1 
## 实际从节点小于设置值时，主从切换不会触发
cluster-require-full-coverage yes
## 默认值yes，集群完整时，方可对外提供服
cluster-replica-no-failover no 
## 默认配置为no。配置成yes时，在master宕机时，slave不会故障转移升为master
```
