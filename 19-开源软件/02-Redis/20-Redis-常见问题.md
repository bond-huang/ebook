# Redis-常见问题
## 启停问题
### 无法关闭redis
无法关闭,root用户也不行，报错示例：
```
[redis@centos82 redis-stable]$ ./src/redis-cli -p 6379 -h 127.0.0.1
127.0.0.1:6379> shutdown
(error) ERR Errors trying to SHUTDOWN. Check logs.
```
前台任务用ctrl+C关闭也报错：
```
^C45800:signal-handler (1690951446) Received SIGINT scheduling shutdown...
45800:M 02 Aug 2023 12:44:06.301 # User requested shutdown...
45800:M 02 Aug 2023 12:44:06.301 * Saving the final RDB snapshot before exiting.
45800:M 02 Aug 2023 12:44:06.301 # Failed opening the temp RDB file temp-45800.rdb (in server root dir /soft/redis-stable) for saving: Permission denied
45800:M 02 Aug 2023 12:44:06.301 # Error trying to save the DB, can't exit.
45800:M 02 Aug 2023 12:44:06.301 # Errors trying to shut down the server. Check the logs for more information.
```
提示没有权限，原因是使用redis用户启动的，主目录/soft/redis-stable属于root，修改所属：
```
[root@centos82 soft]# chown redis.redis redis-stable
```
### 启动问题
#### IP或端口原因导致启动失败
使用配置文件启动时候报错：
```
[redis@centos82 logs]$ cat redis_6379.log
46303:C 02 Aug 2023 13:13:24.717 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
46303:C 02 Aug 2023 13:13:24.717 # Redis version=7.0.12, bits=64, commit=00000000, modified=0, pid=46303, just started
46303:C 02 Aug 2023 13:13:24.717 # Configuration loaded
46303:M 02 Aug 2023 13:13:24.718 * monotonic clock: POSIX clock_gettime
46303:M 02 Aug 2023 13:13:24.718 # Warning: Could not create server TCP listening socket 121.43.191.157:6379: bind: Cannot assign requested address
46303:M 02 Aug 2023 13:13:24.718 # Failed listening on port 6379 (TCP), aborting.
```
可能原因：
- 6379端口被占用，使用命令`netstat -tunpl |grep 6379`查看
- IP没有配置，使用命令`ifconfig -a`查看

## 连接问题
### 连上后没权限
能连上但是提示没有权限示例：
```
[redis@centos82 redis-stable]$ ./src/redis-cli -p 6379 -h 127.0.0.1
127.0.0.1:6379> info
NOAUTH Authentication required.
127.0.0.1:6379> info memory
Error: Server closed the connection
```
因为在配置文件中指定了密码，加上密码即可：
```
[redis@centos82 redis-stable]$ ./src/redis-cli -a 123456 -p 6379 -h 127.0.0.1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
127.0.0.1:6379> info memory
# Memory
used_memory:931376
...
```
相关配置选项：`protected-mode yes`，`requirepass 123456`

## 待补充