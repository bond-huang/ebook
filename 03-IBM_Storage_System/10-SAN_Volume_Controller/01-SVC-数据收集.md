# SVC-数据收集
SVC是IBM Spectrum Storage™产品家族成员，IBM Storwize Storage系列及IBM FlashSystem系列和SVC管理界面基本一致，收集数据方法也基本一致。

### 收集数据方式
在SVC中，收集故障诊断日志有四种方式：
- management GUI interface：用户管理界面
- service assistant  interface：一般是IBM工程师用
- command-line interface (CLI)：很少用
- service command-line interface (CLI) ：IBM专家用
- USB flash drive interface：极少数情况下IBM工程师用

本文主要介绍用Management GUI数据收集方法，详解各种数据适合的不同故障场景，可以让客户及时准确收集对应的数据，以便IBM快速准确的判断故障。

### Management GUI数据收集
整理摘自IBM官方支持中心，链接：[https://www.ibm.com/support/pages/node/690179](https://www.ibm.com/support/pages/node/690179)
##### 收集方法
在Management GUI中，选择Settings--> Support--> Download Support Package-->选择日志类型然后点击 "Download"。     
四种方式收集日志描述及大小估算：   

Option|Description|Size(1Group,30vol)|Size(4Groups,250Vol)
:---|:---|:---|:---
1|Standard logs|10 MB|340 MB
2|Standard logs plus one existing statesave|50 MB|520 MB
3|Standard logs plus most recent statesave from each node|90 MB|790 MB
4|Standard logs plus new statesaves|90 MB|790 MB

##### 四种选项解析
Option1：Standard logs   
大多数问题都可以通过此日志去诊断，例如简单硬件故障，包含内容如下：
- SVC/Storwize事件日志
- SVC/Storwize审核日志
- Linux日志，包括/var/log/messages

Option2：Standard logs plus one existing statesave     
此选项包含选项1以及系统中的当前配置节点最新statesave。在某些微码版本中，此选项实际上从当前配置节点收集最新的转储。

Option3：Standard logs plus most recent statesave from each node      
此选项包含选项1以及系统中每个节点最新statesave。

Option4：Standard logs plus new statesaves    
此选项生成livedump。livedump在系统中的每个节点或节点容器上生成，并且此时包含许多有关系统的详细日志记录信息，此详细的日志记录信息对于分析某些问题是必需的。       
收集此日志注意事项：
- livedump生成可能需要10到30分钟
- 进行livedump收集会对系统性能产生暂时的小影响

##### 根据故障类型选择日志
- 对于与主机或存储之间有关的问题，选择Option4
- 对于关键性能问题，先收集Option1，然后收集Option4
- 对于一般性能问题，收集Option4
- 对于2030、1196或1195错误，收集Option3，不要收集Option4
- 对于与压缩卷有关的问题，收集Option4
- 对于存储子系统的问题，收集Option4
- 对于与Metro或Global Mirror有关的问题，包括1920错误，从两个系统中收集Option4
- 对于所有其他问题，包括一般硬件问题，收集Option1

### Service Assistant Interface收集
客户很少用到，不作详述，一般是IBM工程师用的，在发生宕机故障时候，收集宕机时刻生成的dump日志。

### USB flash drive interface收集
节点启动不了时候收集，一般是IBM工程师用，IBM工程师也很少用到，不作详述。
### 其它信息收集
##### VPD信息收集
收集方法如下：
- 登录到Management GUI中
- 选择"Monitoring"，然后选择"System"
- 选中对应的节点，然后右键
- 选择"Actions"菜单里面的"download VPD"
