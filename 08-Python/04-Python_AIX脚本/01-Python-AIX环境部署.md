# Python-AIX环境部署
AIX系统中安装Python不复杂，不过相对Windows和Linux步骤还是麻烦点，需要安装依赖包以及安装顺序也有要求。简单记录下免得忘记了。
### AIX系统中安装Python
AIX 系统版本：7100-04-03-1642；Python版本:Python 3.7。
##### 安装包准备
Python安装包：python3-3.7.6-1.aix6.1.ppc.rpm

除了Python的安装包，还需要准备依赖包，也行版本不一样要求不一样，不知道需要什么依赖包的话，可以直接安装Python安装包，会有提示需要哪些包，如下所示：
```shell
error: failed dependencies:
        bzip2 >= 1.0.8 is needed by python3-3.7.6-1
        gdbm >= 1.18.1 is needed by python3-3.7.6-1
        libgdbm.a(libgdbm.so.6)   is needed by python3-3.7.6-1
        libreadline.a(libreadline.so.8)   is needed by python3-3.7.6-1
        libsqlite3.so   is needed by python3-3.7.6-1
        readline >= 8.0-2 is needed by python3-3.7.6-1
        sqlite >= 3.28.0 is needed by python3-3.7.6-1
```
所以还需要准备包：
- bzip2-1.0.8-2.aix6.1.ppc.rpm          
- gdbm-1.18.1-1.aix6.1.ppc.rpm       
- readline-8.0-2.aix6.1.ppc.rpm
- sqlite-3.28.0-1.aix6.1.ppc.rpm 

开发工具包(根据需求装)：
- 附带开发工具：python3-tools-3.7.6-1.aix6.1.ppc.rpm
- 开发需要的libraries和header文件：python3-devel-3.7.6-1.aix6.1.ppc.rpm

这些rpm包下载地址：[AIX Toolbox for Linux Applications](https://www.ibm.com/support/pages/node/883796?mhsrc=ibmsearch_a&mhq=AIX%20Toolbox%20for%20Linux%C2%AE%20Applications)

##### 安装Python
安装顺序就是在直接安装Python时候提示的依赖包需求顺序，依次执行：
```shell
rpm -ivh bzip2-1.0.8-2.aix6.1.ppc.rpm          
rpm -ivh gdbm-1.18.1-1.aix6.1.ppc.rpm       
rpm -ivh readline-8.0-2.aix6.1.ppc.rpm
rpm -ivh sqlite-3.28.0-1.aix6.1.ppc.rpm
rpm -ivh python3-3.7.6-1.aix6.1.ppc.rpm
rpm -ivh python3-tools-3.7.6-1.aix6.1.ppc.rpm
rpm -ivh python3-devel-3.7.6-1.aix6.1.ppc.rpm
```
查看是否安装,输入命令`rpm -qa|grep python`，输入`Python3`进入到IDLE：
```shell
Python 3.7.6 (default, Feb 28 2020, 04:49:11) 
[GCC 8.3.0] on aix6
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

