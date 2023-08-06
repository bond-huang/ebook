# Redis-集群模式操作
## 集群切换
### 三主三从集群切换
使用之前配置的集群，信息如下：

系统名称|IP|系统版本|主从|服务端口|配置文件
:---:|:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6500|redis-134b.conf
redhat8|192.168.100.134|RHEL8.0|从|6501|redis-134c.conf
redhat9|192.168.100.133|RHEL9.0|主|6500|redis-133b.conf
redhat9|192.168.100.133|RHEL9.0|从|6501|redis-133c.conf
redhat9a|192.168.100.138|RHEL9.0|主|6500|redis-138b.conf
redhat9a|192.168.100.138|RHEL9.0|从|6501|redis-138c.conf

对三个从节点分别执行切换，任意系统节点执行，命令示例：
```shell
redis-cli -h 192.168.100.134 -p 6501 -a 123456 -c cluster failover
redis-cli -h 192.168.100.133 -p 6501 -a 123456 -c cluster failover
redis-cli -h 192.168.100.138 -p 6501 -a 123456 -c cluster failover
```
执行命令：
```
[redis@redhat9 ~]$ redis-cli -h 192.168.100.134 -p 6501 -a 123456 -c cluster failover
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
[redis@redhat9 ~]$ redis-cli -h 192.168.100.133 -p 6501 -a 123456 -c cluster failover
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
[redis@redhat9 ~]$ redis-cli -h 192.168.100.138 -p 6501 -a 123456 -c cluster failover
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
```
查看集群信息：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6501 (ae4d8172...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.138:6501 (b3d1d5d6...) -> 0 keys | 5462 slots | 1 slaves.
192.168.100.133:6501 (fbf68c6d...) -> 0 keys | 5461 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.100.134:6500)
S: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots: (0 slots) slave
   replicates fbf68c6df1d7223009b86f80b17be8b78fb98aa6
S: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots: (0 slots) slave
   replicates b3d1d5d6de952883a3fed212f943c16565a48b44
M: ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500
   slots: (0 slots) slave
   replicates ae4d817220f757abd4c9d76a71800920285bbcd9
M: fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
弄完切换回来。
## 集群节点增减
### 集群节点增加
将之前三主三从增加到四主四从，使用信息如下：

系统名称|IP|系统版本|主从|服务端口|配置文件
:---:|:---:|:---:|:---:|:---:|:---:
redhat8|192.168.100.134|RHEL8.0|主|6500|redis-134b.conf
redhat8|192.168.100.134|RHEL8.0|从|6501|redis-134c.conf
redhat9|192.168.100.133|RHEL9.0|主|6500|redis-133b.conf
redhat9|192.168.100.133|RHEL9.0|从|6501|redis-133c.conf
redhat9|192.168.100.133|RHEL9.0|主|6502|redis-133d.conf
redhat9a|192.168.100.138|RHEL9.0|主|6500|redis-138b.conf
redhat9a|192.168.100.138|RHEL9.0|从|6501|redis-138c.conf
redhat9a|192.168.100.138|RHEL9.0|从|6502|redis-138d.conf

&#8195;&#8195;在对应分区分别添加`redis-133d.conf`和`redis-138d.conf`配置文件，配置文件和c差不多修改相关文件名和端口即可。启动新加的节点，命令分别如下：
```shell
/soft/redis/bin/redis-server /soft/redis/conf/redis-133d.conf
/soft/redis/bin/redis-server /soft/redis/conf/redis-138d.conf
```
启动后添加节点，下面两种方式均可：
```shell
redis-cli -p 6500 -a 123456 --cluster add-node 192.168.100.133:6502 192.168.100.133:6500
redis-cli -a 123456 -p 6500 -h 192.168.100.134 cluster meet 192.168.100.138 6502
```
添加`192.168.100.133:6502`示例：
```
[redis@redhat8 ~]$ redis-cli -p 6500 -a 123456 --cluster add-node 192.168.100.133:6502 192.168.100.133:6500
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.100.133:6502 to cluster 192.168.100.133:6500
>>> Performing Cluster Check (using node 192.168.100.133:6500)
M: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501
   slots: (0 slots) slave
   replicates 6541e11615d81e677f6ebba0b4d4d3df0772ca81
M: e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501
   slots: (0 slots) slave
   replicates d26627581602fc00c2b5f249c31dc831cee01217
S: ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501
   slots: (0 slots) slave
   replicates e9451337dd0bdd3923adda404089953065fa6712
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Getting functions from cluster
>>> Send FUNCTION LIST to 192.168.100.133:6502 to verify there is no functions in it
>>> Send FUNCTION RESTORE to 192.168.100.133:6502
>>> Send CLUSTER MEET to node 192.168.100.133:6502 to make it join the cluster.
[OK] New node added correctly.
```
现在变成四主三从，继续添加`192.168.100.138:6502`：
```
[redis@redhat8 ~]$ redis-cli -a 123456 -p 6500 -h 192.168.100.134 cluster meet 192.168.100.138 6502
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
```
现在变成五主三从，查看示例：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.138:6502 (7f5337f4...) -> 0 keys | 0 slots | 0 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 5462 slots | 1 slaves.
192.168.100.133:6502 (adba90d8...) -> 0 keys | 0 slots | 0 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 5461 slots | 1 slaves.
[OK] 0 keys in 5 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.100.134:6500)
......
```
将`192.168.100.138:6502`改为`192.168.100.133:6502`的从：
```
[redis@redhat9a ~]$ redis-cli -p 6502 -h 192.168.100.138 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.138:6502> cluster replicate adba90d885b53bcb7b9e6216d1a2a2fd281c4aba
OK
```
注意事项：
- 进入打算作为从的节点
- `replicate`后面接的是作为主节点的ID

### 集群槽位移动
前面从三主三从增加到四主四从后，新的主节点``192.168.100.133:6502``没有槽位，如下所示：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 5462 slots | 1 slaves.
192.168.100.133:6502 (adba90d8...) -> 0 keys | 0 slots | 1 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 5461 slots | 1 slaves.
```
槽位移动建议在业务低峰期进行，不影响业务情况下。
#### 手动槽位移动
将主节点`192.168.100.134:6500`往新主节点`192.168.100.133:6502`分配1000个slot：
```shell
redis-cli -h 192.168.100.134 -p 6500 -a 123456 --cluster reshard 192.168.100.133:6500
```
执行示例：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 --cluster reshard 192.168.100.133:6500
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing Cluster Check (using node 192.168.100.133:6500)
......
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 1000
What is the receiving node ID? adba90d885b53bcb7b9e6216d1a2a2fd281c4aba
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: d26627581602fc00c2b5f249c31dc831cee01217
Source node #2: done

Ready to move 1000 slots.
  Source nodes:
    M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
  Destination node:
    M: adba90d885b53bcb7b9e6216d1a2a2fd281c4aba 192.168.100.133:6502
       slots: (0 slots) master
       1 additional replica(s)
  Resharding plan:
    Moving slot 0 from d26627581602fc00c2b5f249c31dc831cee01217
    ......
    Moving slot 999 from d26627581602fc00c2b5f249c31dc831cee01217
Do you want to proceed with the proposed reshard plan (yes/no)? yes
Moving slot 0 from 192.168.100.134:6500 to 192.168.100.133:6502:
......
Moving slot 999 from 192.168.100.134:6500 to 192.168.100.133:6502:
```
其它两个节点也分别移动1000个，完成后查看效果：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 4461 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 4462 slots | 1 slaves.
192.168.100.133:6502 (adba90d8...) -> 0 keys | 3000 slots | 1 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 4461 slots | 1 slaves.
[OK] 0 keys in 4 masters.
```
命令说明：
- `reshard`自定义模式，会进入菜单让自定义移动数量，移动源和目标
- `reshard`后面任意其它节点均可，能连上集群即可
- 移动槽位注意接收和源节点的ID
- 选择源节点时候输入`all`将从三个主节点平均往目标节点移动slot

#### 自动均衡槽位
&#8195;&#8195;如果目标节点是空槽位，加上`--cluster-use-empty-masters`让没有分配slot的主节点参与，默认是不会允许其参与：
```shell
redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
--cluster rebalance --cluster-use-empty-masters 192.168.100.133:6500
```
让有槽位的主节点自动进行均衡，：
```shell
redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
--cluster rebalance  192.168.100.133:6501
```
执行后查看槽位示例：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 4096 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 4096 slots | 1 slaves.
192.168.100.133:6502 (adba90d8...) -> 0 keys | 4096 slots | 1 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 4096 slots | 1 slaves.
[OK] 0 keys in 4 masters.
```
命令说明：
- 加上`--cluster-use-empty-masters`让没有分配slot的主节点参与均衡，不加不参与
- `rebalance`后面节点信息任意写集群中一个节点信息即可
#### 槽位清空
&#8195;&#8195;如果槽位中有数据，需要进行迁移slots，例如将主`192.168.100.133:6502`节点槽位清空迁移到其它节点。首先迁移到主`192.168.100.133:6500`节点，示例：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 4096 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 8192 slots | 2 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 4096 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.100.134:6500)
M: d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: 6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500
   slots:[0-1364],[5461-12287] (8192 slots) master
   2 additional replica(s)
S: adba90d885b53bcb7b9e6216d1a2a2fd281c4aba 192.168.100.133:6502
   slots: (0 slots) slave
```
迁移完成后`192.168.100.133:6502`节点变成从节点。
### 集群节点删除
&#8195;&#8195;将上面四主四从减少到三主三从。删除主`192.168.100.133:6502`节点和从`192.168.100.138:6502`节点，删除命令示例如下：
```shell
redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
--cluster del-node 192.168.100.138:6502 7f5337f446dab1543f290e87a9fdde4cfc874e1d
```
删除从`192.168.100.138:6502`节点执行示例：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
> --cluster del-node 192.168.100.138:6502 7f5337f446dab1543f290e87a9fdde4cfc874e1d
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node 7f5337f446dab1543f290e87a9fdde4cfc874e1d from cluster 192.168.100.138:6502
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```
成功删除，继续删除主`192.168.100.133:6502`节点：
```
redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
--cluster del-node 192.168.100.133:6502 adba90d885b53bcb7b9e6216d1a2a2fd281c4aba
```
执行删除失败，因为此主节点非空：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 \
> --cluster del-node 192.168.100.133:6502 adba90d885b53bcb7b9e6216d1a2a2fd281c4aba
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node adba90d885b53bcb7b9e6216d1a2a2fd281c4aba from cluster 192.168.100.133:6502
[ERR] Node 192.168.100.133:6502 is not empty! Reshard data away and try again.
```
&#8195;&#8195;安全保险的做法就是先迁移槽位，再删除从节点，再删除主节点，迁移完主`192.168.100.133:6502`节点的槽位后删除`192.168.100.133:6502`节点（已变成从节点），示例：
```
[redis@redhat8 ~]$ redis-cli -h 192.168.100.134 -p 6500 -a 123456 --cluster del-node 192.168.100.133:6502 adba90d885b53bcb7b9e6216d1a2a2fd281c4aba
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node adba90d885b53bcb7b9e6216d1a2a2fd281c4aba from cluster 192.168.100.133:6502
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```
变成三主三从，如有必要，可以进行槽位均衡，均衡后查看示例：
```
[redis@redhat9a ~]$ redis-cli --cluster check 192.168.100.134:6500 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.100.134:6500 (d2662758...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.133:6500 (6541e116...) -> 0 keys | 5461 slots | 1 slaves.
192.168.100.138:6500 (e9451337...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
```
但是槽位在每个节点上不一定是连续的，示例：
```
[redis@redhat9 ~]$ redis-cli -h 192.168.100.133 -p 6500 -a 123456 -c cluster nodes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
d26627581602fc00c2b5f249c31dc831cee01217 192.168.100.134:6500@16500 master - 0 1691203001000 19 connected 1365-5460 5462-6826
b3d1d5d6de952883a3fed212f943c16565a48b44 192.168.100.138:6501@16501 slave 6541e11615d81e677f6ebba0b4d4d3df0772ca81 0 1691202999804 17 connected
6541e11615d81e677f6ebba0b4d4d3df0772ca81 192.168.100.133:6500@16500 myself,master - 0 1691203000000 17 connected 6827-12287
e9451337dd0bdd3923adda404089953065fa6712 192.168.100.138:6500@16500 master - 0 1691203002849 18 connected 0-1364 5461 12288-16383
fbf68c6df1d7223009b86f80b17be8b78fb98aa6 192.168.100.133:6501@16501 slave d26627581602fc00c2b5f249c31dc831cee01217 0 1691202998000 19 connected
ae4d817220f757abd4c9d76a71800920285bbcd9 192.168.100.134:6501@16501 slave e9451337dd0bdd3923adda404089953065fa6712 0 1691203001834 18 connected
```
## 待补充