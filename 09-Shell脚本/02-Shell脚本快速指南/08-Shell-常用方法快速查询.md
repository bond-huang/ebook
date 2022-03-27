# Shell-常用方法快速查询
一些简单，但是常用的方法。
## 数据行处理方法
获取行数据：
```shell
# 取第一行
sed '1q'
head -n 1
awk 'FNR=1 {print}'
# 取前十行
sed '10q'
head -n 10
awk 'FNR>=1&&FNR<=10 {print}'
```
删除行：
```shell
# 删除第一行
sed '1d'
# 删除第2至5行
sed '2,5d'
```
