# Cisco-常用命令
## 日常检查命令
查看交换机日志，例如查看最近50条日志信息：
```
# show log last 50
```
查看交换机板卡状态：
```
# show module
```
查看交换机背板板卡状态：
```
# show module  xbar
```
查看交换机电源状态：
```
# show environment  power
```
查看交换机风扇状态：
```
# show environment  fan
```
查看交换机温度情况：
```
# show environment temperature
```
查看交换机HA状态：
```
# show system  redundancy status
```
查看交换机端口状态：
```
# show interface brice
```
状态说明：
- 端口状态有up(正常)，down（关闭），trunking（级联口），init（初始化），linkfailure（物理链路有问题），notConnect（没有连线）等

查看交换机端口WWN：
```
# show flogi database vsan 71 
```
查看交换机实时流量：
```
# show interface counters brief
```
查看交换机端口丢包率：
```
# show int fc x/x counters
```
查看交换机端口错误计数：
```
# show int fc2/36 counters detail
```
查看交换机收发光功率：
```
# show int fc x/x tracsceiver detail
```
## 待补充