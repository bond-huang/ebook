# HMC中shell脚本使用
### 简介
HMC底层是Redhat，经过修改和限制后，还是相对比较封闭；一般登录是采用hscroot这个用户，此用户权限一般，很多命令没有，此用户也无法ftp。
官方有提供一些小工具去抓取一些数据，例如HMC Scanner，简单的可以直接在命令行进行输入，例如用for循环批量创建分区和分区资源的时候。
### HMC Scanner
HMC Scanner可以用于HMC配置信息收集，主要是Lpar的一些重要配置信息，会自动生成一个配置表格。
此工具有shell也有java，跑的时候还是比较慢，耐心等待，但是数据确实很全面的。
官方链接：
[HMC Scanner](http://www.ibm.com/developerworks/wikis/display/WikiPtype/HMC+Scanner)
由于HMC的一些局限性，我觉得此工具提供了一个很好的接口，在上面进行修改或者添加，得到自己定制的内容，也是很不错的。
### for循环中运算问题
近期在HMC中用hscroot用户跑一个for循环去对分区进行批量资源创建的时候，发现运算还是有点不一样。
在V9R1版本和V7R7.9版本中都进行了如下运算测试：
```shell
hscroot@hmc:~> for i in {2..5}
> do
> echo Lpar$(echo "$i*10+1"|bc)
> done
```
运行后都是会报错：bash: bc: command not found
说明HMC的hscroot用户没bc这个命令，用下面运算方法替代即可：
```shell
hscroot@hmc:~> for i in {2..5}
> do
> echo Lpar$(($i*10+1))
> done
```
运行后结果如下：
```shell
Lpar21
Lpar31
Lpar41
Lpar51
```
