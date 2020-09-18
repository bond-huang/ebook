# Shell-PowerHA相关脚本
PowerHA使用过程中写的一些脚本。
## 双节点PowerHA切换
使用smit菜单和命令行切换也比较方便快捷，但是有时候想用脚本进行批量切换。
### 脚本使用说明
脚本使用说明如下：
- 适用于双节点的PowerHA环境，多节点或多site不支持
- 适用于单资源组的PowerHA环境，多资源组不支持
- 适用于PowerHA7.2版本，老版本如HA6.1可能不支持（未测试过）
- 在任意节点运行脚本都行，脚本中有判断资源组所在的节点

### 脚本代码
脚本代码如下：
```sh
#!/bin/ksh
# Two nodes PowerHA resource group switch script.
Cls_dir="/usr/es/sbin/cluster/utilities"
Cls_State=`$Cls_dir/cldump |sed -n '/Cluster State/p' |awk '{print $3}'`
Cls_Substate=`$Cls_dir/cldump |sed -n '/Cluster Substate/p' |awk '{print $3}'`
echo "Check the Cluster state,please waiting..."
if [ $Cls_State = "UP" ] && [ $Cls_Substate = "STABLE" ]
then
    echo "The Cluster state is STABLE!"
else
    echo "The Cluster state is abnormal,please check the cluster!"
    exit 1
fi
####
echo "Check the Nodes state,please waiting..."
Node_State=$($Cls_dir/cldump |sed -n '/Node name/p'| awk '{print $5}')
for state in $Node_State
do 
    if [ $state != "UP" ]
    then 
        echo "Someone node state is abnormal,please check the cluster!"
        exit 1
    fi
done
####
RG=$($Cls_dir/clmgr list rg)
RG_Node=`$Cls_dir/cldump |tail -2 |awk '{if($2=="ONLINE"){print $1}}'`
Standy_Node=`$Cls_dir/cldump |tail -2 |awk '{if($2=="OFFLINE"){print $1}}'`
####
if [ -n $RG_Node  ] && [ -n $Standy_Node ]
then
    echo "Move the resource group form $RG_Node to $Standy_Node,please waiting..."
    $Cls_dir/clRGmove -s 'false' -m -i -g $RG -n $Standy_Node
else
    echo "RG state is abnormal on someone node,please check the cluster!"
    exit 1
fi
####
if [ $? -eq 0 ]
then 
    echo "Successfully moved the resource group form $RG_Node to $Standy_Node!"
    echo "Please check the Cluster status and check the application!"
else
    echo "Failed to move the resource group form $RG_Node to $Standy_Node!"
    echo "Please check the PowerHA log:/var/hacmp/log/hacmp.out!"
fi
```
### 运行示例
&#8195;&#8195;在AIX系统7.1.3.7及HA版本7.2.1.2版本中运行示例（发现最后写的判断有点多余了，clRGmove会有成功与否的输出）：
```
# sh HAswtich.sh
Check the Cluster state,please waiting...
Move the resource group form HQ_test1 to HQ_test2,please waiting...
Attempting to move resource group HQrg to node HQ_test2.

Waiting for the cluster to process the resource group movement request....

Whit for the cluster to stabilize

Resource group movement successful.
Resource group HQrg is online on node HQ_test2.

Cluster Name: HQ_cls
Resource Group name:HQrg
Node                                                            Group State
-----------------------------------------------------------------------------
HQ_test2                                                        ONLINE
HQ_test1                                                        OFFLINE
Successfully moved the resource group form $RG_Node to $Standy_Node!
Please check the Cluster status and check the application!
```
## PowerHA检查
&#8195;&#8195;PowerHA系统需要定期进行切换演练，有些重要行业例如一些银行放在人行窗口进行切换，在切换前要检查系统，如果需要检查的系统比较多，用脚本就比较方便了。
### 脚本使用说明
说明如下：
- 检查了Cluster进程状态，只要有一个不是active就不通过了
- 检查了HA集群状态，node状态等等
- 检查了hosts表和rhosts
- HA6.1和HA7.1(7.2)版本rhosts位置不一样，并且HA6.1里面一般配置IP，在HA7.1中一般配置Boot IP Lable（一般IP Lable和node配置成一致）在脚本中有判断系统版本，根部版本检查不同路径的rhosts
- 脚本中写了一个同步，并且询问了是否进行同步，根据需求选择yes或no

### 脚本代码
