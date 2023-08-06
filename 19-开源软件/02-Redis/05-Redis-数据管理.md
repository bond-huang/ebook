# Redis-数据管理
## 相关软件安装
### Ruby安装
官方网站：[https://www.ruby-lang.org/zh_cn/](https://www.ruby-lang.org/zh_cn/)    
下载地址：[https://www.ruby-lang.org/zh_cn/downloads/](https://www.ruby-lang.org/zh_cn/downloads/)     
软件下载版本列表：[https://www.ruby-lang.org/en/downloads/releases/](https://www.ruby-lang.org/en/downloads/releases/)

下载版本2.7.8，名称ruby-2.7.8.tar.gz。上传到soft目录，安装依赖包：
```shell
yum install openssl openssl-devel gdbm gdbm-devel readline readline-devel gcc gcc-c++
```
解压安装包：
```
[root@centos82 ruby]# tar -zxvf ruby-2.7.8.tar.gz
[root@centos82 ruby]# cd ruby-2.7.8/
```
配置安装：
```
[root@centos82 ruby-2.7.8]# ./configure --prefix=/soft/ruby/
[root@centos82 ruby-2.7.8]# make && make install
```
直接yum安装也可以，此次安装版本比较低：
```
[root@centos82 ~]# yum install ruby
......
Installed:
  ruby-2.5.9-107.module_el8.4.0+847+ee687b6c.x86_64
  ruby-irb-2.5.9-107.module_el8.4.0+847+ee687b6c.noarch
  ruby-libs-2.5.9-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-bigdecimal-1.3.4-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-did_you_mean-1.2.0-107.module_el8.4.0+847+ee687b6c.noarch
  rubygem-io-console-0.4.6-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-json-2.1.0-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-openssl-2.1.2-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-psych-3.0.2-107.module_el8.4.0+847+ee687b6c.x86_64
  rubygem-rdoc-6.0.1.1-107.module_el8.4.0+847+ee687b6c.noarch
  rubygems-2.7.6.3-107.module_el8.4.0+847+ee687b6c.noarch

Complete!
```
还是用2.7.8版本，更新到国内源：
```
[root@centos82 ruby-3.2.2]# cd /soft/ruby/bin
[root@centos82 bin]# ./gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
source https://gems.ruby-china.com/ already present in the cache
source https://rubygems.org/ not present in cache
```
安装redis-copy命令
```
[root@centos82 bin]# ./gem install redis-copy
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
安装redis-dump命令
```
[root@centos82 bin]# ./gem install uri-redis -v 0.4.2
[root@centos82 bin]# ./gem install redis-dump
[redis@centos82 bin]$ ./gem install redis-dump
......
Successfully installed drydock-0.6.9
Successfully installed connection_pool-2.4.1
Successfully installed redis-client-0.15.0
Successfully installed redis-5.0.6
Building native extensions. This could take a while...
Successfully installed yajl-ruby-1.4.3
Successfully installed redis-dump-0.4.0
......
Done installing documentation for drydock, connection_pool, redis-client, redis, yajl-ruby, redis-dump after 1 seconds
6 gems installed
```
安装成功后查看：
```
[redis@centos82 bin]$ ls -l
......
-rwxr-xr-x 1 redis redis      552 Aug  6 19:23 redis-copy
-rwxr-xr-x 1 redis redis      552 Aug  6 19:34 redis-dump
-rwxr-xr-x 1 redis redis      552 Aug  6 19:34 redis-load
-rwxr-xr-x 1 redis redis      556 Aug  6 19:34 redis-report
```
## 数据备份与迁移
### 环境及数据准备
使用在阿里云服务器下配置的一主一从，启动服务：
```
[redis@centos82 redis-stable]$ ./src/redis-server ./conf/redis_7012.conf
[redis@centos82 redis-stable]$ ./src/redis-server ./conf/redis_6500.conf
[redis@centos82 redis-stable]$ ./src/redis-server ./conf/redis_6501.conf
[redis@centos82 redis-stable]$ ps -ef |grep redis
......
redis     117376  117375  0 12:27 ?        00:00:00 /usr/libexec/openssh/sftp-server
redis     117504       1  0 12:38 ?        00:00:00 ./src/redis-server 172.26.9.154:6379
redis     117511       1  0 12:38 ?        00:00:00 ./src/redis-server 172.26.9.154:6500
redis     117518       1  0 12:38 ?        00:00:00 ./src/redis-server 172.26.9.154:6501
```
主从关系如下：
```
172.26.9.154:6379> info Replication
# Replication
role:master
connected_slaves:1
slave0:ip=172.26.9.154,port=6500,state=online,offset=280,lag=1
master_failover_state:no-failover
master_replid:177f3b7e15a1af0f22b595d17030d12ffb15e1dc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:280
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:104857600
repl_backlog_first_byte_offset:1
repl_backlog_histlen:280
```
节点`172.26.9.154:6501`是一个单独的主节点。往`172.26.9.154:6379`里面0号库写入一点数据：
```shell
for i in {0..1999}
do 
    redis-cli -h 172.26.9.154 -p 6379 -a 123456 set superhero_$i batman_$i
done
```
往1号库写入一点数据：
```shell
for i in {0..1999}
do 
    redis-cli -h 172.26.9.154 -p 6379 -a 123456 -n 1 set supergod_$i Thor_$i
done
```
写入后查看信息：
```
172.26.9.154:6379> info Keyspace
# Keyspace
db0:keys=2000,expires=0,avg_ttl=0
db1:keys=2000,expires=0,avg_ttl=0
172.26.9.154:6379> get superhero_1000
"batman_1000"
172.26.9.154:6379[1]> get supergod_1000
"Thor_1000"
```
### redis-dump命令数据备份
&#8195;&#8195;使用`redis-dump`命令对刚才数据进行备份，将`172.26.9.154:6379`的所有库数据进行备份，到指定目录的指定文件中，示例如下：
```
[redis@centos82 redis-stable]$ redis-dump -u:123456@172.26.9.154:6379 > /tmp/redis-6379back-230805.json
[redis@centos82 redis-stable]$ cat /tmp/redis-6379back-230805.json
{"db":0,"key":"superhero_1275","ttl":-1,"type":"string","value":"batman_1275","size":11}
{"db":0,"key":"superhero_740","ttl":-1,"type":"string","value":"batman_740","size":10}
```
### redis-reload数据导入&恢复
&#8195;&#8195;可以使用`redis-reload`命令将`redis-dump`命令到处备份的数据进行恢复或导入。示例将上面导出到文件redis-6379back-230805.json的数据恢复到`172.26.9.154:6501`：
```
[redis@centos82 redis-stable]$ cat /tmp/redis-6379back-230805.json |redis-load -u :123456@172.26.9.154:6501
[redis@centos82 redis-stable]$ ./src/redis-cli -h 172.26.9.154 -p 6501 -a 123456 info keyspace
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
# Keyspace
db0:keys=2000,expires=0,avg_ttl=0
db1:keys=2000,expires=0,avg_ttl=0
db10:keys=2000,expires=0,avg_ttl=0
```
### redis-copy命令数据迁移
将`172.26.9.154:6379`里面1号库数据插入到`172.26.9.154:6501`里面10号库里面去：
```
[redis@centos82 redis-stable]$ ./src/redis-cli -h 172.26.9.154 -p 6379 -a 123456 -n 1 info keyspace
# Keyspace
db0:keys=2000,expires=0,avg_ttl=0
db1:keys=2000,expires=0,avg_ttl=0
[redis@centos82 redis-stable]$ ./src/redis-cli -h 172.26.9.154 -p 6501 -a 123456 info keyspace
# Keyspace
[redis@centos82 redis-stable]$ /soft/ruby/bin/redis-copy --no-prompt :123456@172.26.9.154:6379/1 :123456@172.26.9.154:6501/10
Source:      redis://172.26.9.154:6379/1
Destination: redis://172.26.9.154:6501/10 (empty)
Pattern:     *
Key Emitter: Scan
Strategy:    DumpRestore
PROGRESS: {:success=>1000, :attempt=>1000}
PROGRESS: {:success=>2000, :attempt=>2000}
DONE: {:success=>2000, :attempt=>2000}
[redis@centos82 redis-stable]$ ./src/redis-cli -h 172.26.9.154 -p 6501 -a 123456 info keyspace
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
# Keyspace
db10:keys=2000,expires=0,avg_ttl=0
```
注意事项：
- `/redis-copy`命令中前面的节点信息是源，后面的是目标节点

## 待补充
