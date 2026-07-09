# NeoKylin-软件安装

## 常用软件安装
### squid安装
准备yum源：中标麒麟系统V10SP3-2403安装光盘

准备安装包：
```
squid-4.9-25.ky10.aarch64.rpm
perl-Digest-SHA-6.02-7.ky10.aarch64.rpm
libecap-1.0.1-4.ky10.aarch64.rpm
```
&#8195;&#8195;直接安装squid-4.9-25.ky10.aarch64.rpm提示缺少perl和libecap依赖，这两个在系统光盘里面没有。可以去麒麟官方下载，下载地址：
[https://update.cs2c.com.cn/NS/V10/V10SP3-2403/os/adv/lic/base/aarch64/Packages/](https://update.cs2c.com.cn/NS/V10/V10SP3-2403/os/adv/lic/base/aarch64/Packages/)

安装过程：
- 首先安装`libecap-1.0.1-4.ky10.aarch64.rpm`，如果提示有依赖在yum源进行安装；
- 然后安装`perl-Digest-SHA-6.02-7.ky10.aarch64.rpm`，我安装提示很多perl的依赖，直接yum源安装即可：`yum install -y perl`
- 最后安装`squid-4.9-25.ky10.aarch64.rpm`即可

## 待补充
