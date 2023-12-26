# AIX-HAXD常见问题
## PowerHAXD常用命令
### 常用查看命令
查看节点HA服务状态：
```sh
lssrc -g cluster
lssrc -ls clstrmgrES
```
查看domain中节点状态：
```sh
lsrpnode
```
查看domain状态：
```sh
lsrpdomain
```
查看caa配置：
```sh
/opt/rsct/bin/caa_config -s <domain name>
/usr/sbin/rsct/bin/caa_config -s <domain name>
```
查看集群资源组状态：
```sh
/usr/es/sbin/cluster/utilities/clRGinfo
/usr/es/sbin/cluster/utilities/clRGinfo -p
```
## PowerHAXD常见问题
### 本地和远程互相获取不到状态
由于某些原因，本地或者远程的site上节点HA停止了，例如状态：
- 本地site的node1和node2的PowerHA启动的
- 远程site的node3的PowerHA是停止的

本地site的节点两个运行`lsrpnode`看到状态:
```
Name        OpState     RSCTVersion
node1       Online      3.2.1.4
node2       Online      3.2.1.4
node3       Offline     3.2.1.4
```
远程site的节点运行`lsrpnode`看到状态:
```
Name        OpState     RSCTVersion
node1       Offline     3.2.1.4
node2       Offline     3.2.1.4
node3       Online      3.2.1.4
```
此时如果启动远程site的node3 HA时候，可能会发生脑裂：
- 远程site的node3挂载文件系统和服务IP
- 本地site的服务节点的服务IP无法释放，VG也无法停止，导致业务异常和IP冲突

解决方法：
- 如果没有启动远程site node3的HA，重启下node3操作系统，或者HA相关服务（clstrmgrES），直到互相都看到是online时候，再启动node3的PowerHA集群服务
- 如果已经启动node3导致脑裂了，三个节点都重启下操作系统，互相都看到是online时候，启动PowerHAXD集群，三个节点都启动

### CAA信息与配置不一致
由于某些原因，修改了site信息，例如原site名称为siteA和siteB：
- siteA里面有节点node1和node2，优先级高；将siteA改成了site_2，节点改成node3，优先级预期变成低
- siteB里面有节点node3，优先级低；将siteB改成了site_1，节点改成node1和node2，优先级预期变成高

&#8195;&#8195;但是caa里面配置却不是预期的，如果发生异常，很可能产生脑裂，生产两个节点被重启。使用`/opt/rsct/bin/caa_config -s <domain name>`命令查看CAA配置时候显示和预期不一致，如下：
- site名称还是siteB，里面有node1和node2，但是site_priority是2
- site名称还是siteA，里面有node3，但是site_priority是1

目前找到的一种修改方法：
- 停掉PowerHA集群
- 三个节点分别删除caavg_private，依次执行，步骤如下：
    - 停掉服务：`stopsrc -s IBM.ConfigRM`
    - 设置环境变量：`export CAA_FORCE_ENABLED=true`
    - 强行删除caavg_private：`rmcluster -r hdiskX`，提示successfully表示成功，`lspv`查看确认
    - unset环境变量：`unset CAA_FORCE_ENABLED`
    - 启动服务：`startsrc -s IBM.ConfigRM`，或者直接重启系统
- 三个节点caavg_private删除后，在主节点删除PowerHAXD配置里面的site配置，两个site都删除
- 重新添加两个site，例如site_1和site_2，添加对应的节点
- 需要重新添加心跳盘配置，`Add a Repository Disk`选项中进行操作，两个site都需要添加
- 如果site name发生了变化，需要在存储配置里面修改，例如`Change/Show an SVC Cluster`里面的`PowerHA SystemMirror site`的配置
- 在配置的主节点上发起同步，将配置同步到其他节点
- 同步完成后检查三个节点caavg_private是否正常，检查caa配置是否是预期的，`lsrpnode`检查节点状态
- 检查确认无误后，启动PowerHAXD集群，三个节点都启动
- 检查HA状态和资源组状态：`/usr/es/sbin/cluster/utilities/clRGinfo`

注意事项：
- 执行`rmcluster -r hdiskX`就重启系统并且删除不掉，可能是`IBM.ConfigRM`服务未停止，停止后重试

## 待补充