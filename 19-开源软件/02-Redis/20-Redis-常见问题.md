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
## 主从同步问题
### 路由配置问题
启动从节点后，并没有形成主从关系，检查log报错如下：
```
25166:S 04 Aug 2023 15:21:11.799 * Connecting to MASTER 192.168.100.134:6379
25166:S 04 Aug 2023 15:21:11.799 * MASTER <-> REPLICA sync started
25166:S 04 Aug 2023 15:21:11.800 # Error condition on socket for SYNC: No route to host
```
防火墙限制导致无法访问到主节点，关闭防火墙或者开放端口，查看7369端口是否开启：
```
[root@redhat9 ~]# firewall-cmd --query-port=6379/tcp
no
```
开放7369端口：
```
[root@redhat9 ~]# firewall-cmd --add-port=6379/tcp --permanent
success
[root@redhat9 ~]# firewall-cmd --reload
success
[root@redhat9 ~]# firewall-cmd --query-port=6379/tcp
yes
```
## 集群问题
### 集群创建问题
执行命令创建集群：
```
[redis@redhat9a /]$ redis-cli -a 123456 --cluster create \
192.168.100.134:6500 192.168.100.133:6500 192.168.100.138:6500 \
192.168.100.134:6501 192.168.100.133:6501 192.168.100.138:6501 \
--cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.100.133:6501 to 192.168.100.134:6500
Adding replica 192.168.100.138:6501 to 192.168.100.133:6500
Adding replica 192.168.100.134:6501 to 192.168.100.138:6500
M: 0bf0fa53ab672cfa55ea6ec2c96600523f133a13 192.168.100.134:6500
   slots:[0-5460] (5461 slots) master
M: e06e084400dd990113028978fe8a39948f0703de 192.168.100.133:6500
   slots:[5461-10922] (5462 slots) master
M: c8d06a064ae676e7b6ebeda4a18d75af4c0ee6da 192.168.100.138:6500
   slots:[10923-16383] (5461 slots) master
S: 2a9fdfe038bb3b011dd46c94d13bac886764b5c5 192.168.100.134:6501
   replicates c8d06a064ae676e7b6ebeda4a18d75af4c0ee6da
S: 28c17b6a37679e45b9e1546e9aef7da03761acd9 192.168.100.133:6501
   replicates 0bf0fa53ab672cfa55ea6ec2c96600523f133a13
S: 611aa56cc74c14276326ee8effc2bd340c0948a3 192.168.100.138:6501
   replicates e06e084400dd990113028978fe8a39948f0703de
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.....................................
```
&#8195;&#8195;第一次配置，非常慢等了很久，日志也没啥，就一直点点点的走，也不报错。查了下需要开启总线端口，redis服务端口默认加10000，例如redis使用的6500端口和6501端口，命令如下：
```shell
firewall-cmd --add-port=16500/tcp --permanent
firewall-cmd --add-port=16501/tcp --permanent
firewall-cmd --reload
```
关掉redis服务，清空cluster创建的文件。重启redis服务，然后重新执行创建集群命令即可。
## 数据写入问题
### 集群数据写入问题
往集群写入数据时候其它主节点无法写入：
```
[redis@redhat8 ~]$ for i in {0..1999}
> do
>     redis-cli -h 192.168.100.134 -p 6500 -a 123456 set superhero_$i batman_$i
> done
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(error) MOVED 7331 192.168.100.133:6500
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(error) MOVED 11456 192.168.100.133:6500
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(error) MOVED 15585 192.168.100.138:6500
```
需要加`-c`参数去连接集群，示例：
```shell
for i in {0..1999}
do 
    redis-cli -c -h 192.168.100.134 -p 6500 -a 123456 set superhero_$i batman_$i
done
```
## Ruby使用问题
### redis-copy命令报错
示例如下：
```
[redis@centos82 redis-stable]$ /usr/local/bin/redis-copy --no-prompt :123456@172.26.9.154:6379/1 :123456@172.26.9.154:6501/10
Traceback (most recent call last):
        2: from /usr/local/bin/redis-copy:23:in `<main>'
        1: from /soft/ruby/lib/ruby/2.7.0/rubygems.rb:296:in `activate_bin_path'
/soft/ruby/lib/ruby/2.7.0/rubygems.rb:277:in `find_spec_for_exe': can't find gem redis-copy (>= 0.a) with executable redis-copy (Gem::GemNotFoundException)
```
原因是先用yum安装了2.5版本，后面编译安装了2.7.8版本。尝试卸载提示未安装：
```
[root@centos82 bin]# ./gem uninstall redis-copy
Gem 'redis-copy' is not installed
```
安装rvm的依赖包：
```
[root@centos82 ruby]# yum install curl gpg
```
下载并安装rvm：
```
[root@centos82 ruby]# \curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.29.12.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc
gpg: Signature made Sat 16 Jan 2021 02:46:22 AM CST
gpg:                using RSA key 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
gpg: Can't check signature: No public key
GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.12.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --keyserver hkp://keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
    command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -

In case of further problems with validation please refer to https://rvm.io/rvm/security
```
根据提示运行以下命令通过curl导入公钥：
```
[root@centos82 ruby]# command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
gpg: key 3804BB82D39DC0E3: 47 signatures not checked due to missing keys
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 3804BB82D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no ultimately trusted keys found
[root@centos82 ruby]# command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
gpg: key 105BD0E739499BDB: public key "Piotr Kuczynski <piotr.kuczynski@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```
再次执行安装：
```
[root@centos82 ruby]# \curl -sSL https://get.rvm.io | bash -s stable
......
Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:
......
```
查看rvm版本：
```
[root@centos82 bin]# ./rvm --version
rvm 1.29.12 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```
加载`rvm`到当前会话中：
```
[root@centos82 scripts]# source /usr/local/rvm/scripts/rvm
```
查看已安装ruby版本，显示没有：
```
[root@centos82 ~]# rvm list

# No rvm rubies installed yet. Try 'rvm help install'.
```
安装2.7.8版本，并启用此版本：
```
[root@centos82 ~]# rvm install 2.7.8 
......
ruby-2.7.8 - #extracting ruby-2.7.8 to /usr/local/rvm/src/ruby-2.7.8.....
ruby-2.7.8 - #configuring.............................................................
ruby-2.7.8 - #post-configuration..
ruby-2.7.8 - #compiling....................................................
ruby-2.7.8 - #installing..................
ruby-2.7.8 - #making binaries executable...
Installed rubygems 3.1.6 is newer than 3.0.9 provided with installed ruby, skipp                                              ing installation, use --force to force installation.
ruby-2.7.8 - #gemset created /usr/local/rvm/gems/ruby-2.7.8@global
ruby-2.7.8 - #importing gemset /usr/local/rvm/gemsets/global.gems.............
ruby-2.7.8 - #generating global wrappers.................
Error running 'run_gem_wrappers regenerate',
please read /usr/local/rvm/log/1691307518_ruby-2.7.8/gemset.wrappers.global.log
ruby-2.7.8 - #gemset created /usr/local/rvm/gems/ruby-2.7.8
```
有error，根据提示fource安装了一样报错，查看安装版本：
```
=* ruby-2.7.8 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
```
查看gem：
```
[root@centos82 ~]# whereis gem
gem: /usr/bin/gem /usr/local/rvm/rubies/ruby-2.7.8/bin/gem
```
安装`redis-copy`：
```
[root@centos82 ~]# gem install redis-copy
/usr/local/rvm/rubies/ruby-2.7.8/lib/ruby/site_ruby/2.7.0/rubygems/package.rb:509: warning: Using the last argument as keyword parameters is deprecated
Successfully installed redis-copy-1.0.0
Parsing documentation for redis-copy-1.0.0
Done installing documentation for redis-copy after 0 seconds
1 gem installed
```
查看依旧没有：
```
[root@centos82 ~]# gem list redis-copy

*** LOCAL GEMS ***
```
原因应该是安装时候gemset相关报错，查看日志：
```
/usr/local/rvm/log/1691307518_ruby-2.7.8/gemset.wrappers.global.log
```
里面有error：
```
ERROR:  While executing gem ... (Gem::CommandLineError)
    Unknown command wrappers
++ return 1
```
卸载掉ruby-2.7.8，查看版本还有2.5.9版本：
```
[root@centos82 bin]# ruby -v
ruby 2.5.9p229 (2021-04-05 revision 67939) [x86_64-linux]
```
卸载：
```
[root@centos82 bin]# yum remove ruby
[root@centos82 bin]# ruby -v
-bash: /usr/bin/ruby: No such file or directory
```
编译安装的也卸载：
```
[root@centos82 ruby-2.7.8]# pwd
/soft/ruby/ruby-2.7.8
[root@centos82 ruby-2.7.8]# make uninstall
```
换个版本安装3.1.4版本报错：
```
Error running 'env GEM_HOME=/usr/local/rvm/gems/ruby-3.1.4@global GEM_PATH= /usr/local/rvm/rubies/ruby-3.1.4/bin/ruby -d /usr/local/rvm/src/rubygems-3.0.9/setup.rb --no-document',
please read /usr/local/rvm/log/1691313659_ruby-3.1.4/rubygems.install.log
```
删掉了很多相关文件，重新安装3.1.4版本，报错`Error running 'run_gem_wrappers regenerate'`还是有，直接安装redis-copy：
```
[root@centos82 wrappers]# gem install redis-copy
Fetching redis-3.3.5.gem
Successfully installed redis-3.3.5
Fetching concurrent-ruby-1.2.2.gem
Successfully installed concurrent-ruby-1.2.2
Fetching tzinfo-2.0.6.gem
Successfully installed tzinfo-2.0.6
Fetching minitest-5.19.0.gem
Successfully installed minitest-5.19.0
Fetching i18n-1.14.1.gem
Successfully installed i18n-1.14.1
Fetching activesupport-7.0.6.gem
Successfully installed activesupport-7.0.6
Fetching implements-0.0.2.gem
Successfully installed implements-0.0.2
Fetching redis-copy-1.0.0.gem
Successfully installed redis-copy-1.0.0
......
8 gems installed
```
还是不行。还redis用户进行make安装2.7.8版本：
```
[redis@centos82 bin]$ ./gem install redis-copy
Fetching redis-3.3.5.gem
Fetching tzinfo-2.0.6.gem
Fetching i18n-1.14.1.gem
Fetching activesupport-7.0.6.gem
Fetching redis-copy-1.0.0.gem
Fetching implements-0.0.2.gem
Fetching concurrent-ruby-1.2.2.gem
Successfully installed concurrent-ruby-1.2.2
Successfully installed tzinfo-2.0.6
Successfully installed i18n-1.14.1
Successfully installed activesupport-7.0.6
Successfully installed implements-0.0.2
Successfully installed redis-3.3.5
Successfully installed redis-copy-1.0.0
...
7 gems installed
```
终于成功了：
```
[redis@centos82 bin]$ pwd
/soft/ruby/bin
[redis@centos82 bin]$ ls -l redis-copy
-rwxr-xr-x 1 redis redis 552 Aug  6 19:23 redis-copy
[redis@centos82 bin]$ ./redis-copy -v
redis-copy: Source and Destination must be specified

redis-copy v1.0.0 (with redis-rb 3.3.5)
```
改用redis用户就可以了，难道是用户原因？待后期验证
## 待补充