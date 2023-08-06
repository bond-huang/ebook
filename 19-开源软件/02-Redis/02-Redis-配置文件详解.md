# Redis-配置文件详解
## 哨兵模式配置
配置示例：
```ini
port 6380  ## 端口
daemonize yes ## 后台启动
logfile "/soft/redis/logs/sentinel-134a.log" ## 日志位置
pidfile "/soft/redis/pid/sentinel-134a.pid" ## Pid位置
dir "/soft/redis/sentinel_db" ## 数据存放目录
sentinel deny-scripts-reconfig yes ## 拒绝重复执行脚本
## redisMaster是自定义名称，下同。IP是Master IP，2表示至少两个哨兵
sentinel monitor redisMaster 192.168.100.134 6379 2 
## 主节点宕机多长时间后执行切换
sentinel down-after-milliseconds redisMaster 300
## 切换等待时间
sentinel failover-timeout redisMaster 60000
## 配置密码，根据配置文件的reqirepass值配置
sentinel auth-pass redisMaster 123456
```
