# Docker-安装配置
安装配置笔记，参考教程链接：
- [Docker runoob网站教程](https://www.runoob.com/docker/centos-docker-install.html)

## CentOS安装
系统环境：
- Centos 7.5

### 配置YUM源
查看配置文件：
```
[root@VM-0-6-centos ~]# ll /etc/yum.repos.d/
total 8
-rw-r--r-- 1 root root 614 Mar 15 20:04 CentOS-Base.repo
-rw-r--r-- 1 root root 230 Mar 15 20:04 CentOS-Epel.repo
```
腾讯云服务器，看了下配置基本都是TX的，直接安装找不到包，说明没配置：
```
[root@VM-0-6-centos ~]# yum install docker-ce docker-ce-cli containerd.io
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
No package docker-ce available.
No package docker-ce-cli available.
No package containerd.io available.
Error: Nothing to do
```
#### 设置仓库
&#8195;&#8195;`yum-utils`提供了`yum-config-manager`，并且`device mapper`存储驱动程序需要`device-mapper-persistent-data`和`lvm2`，安装如下所示：
```
# yum install -y yum-utils device-mapper-persistent-data lvm2
```
配置官方源地址:
```
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/
docker-ce.reporepo saved to /etc/yum.repos.d/docker-ce.repo
```
或者使用阿里云和清华大学地址：
```
# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```
### 安装Docker Engine-Community
安装最新版本的Docker Engine-Community和containerd：
```
[root@VM-0-6-centos ~]# yum install docker-ce docker-ce-cli containerd.io
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
docker-ce-stable                                                   | 3.5 kB  00:00:00     
(1/2): docker-ce-stable/7/x86_64/updateinfo                        |   55 B  00:00:00     
(2/2): docker-ce-stable/7/x86_64/primary_db                        |  58 kB  00:00:00 
...
Install  3 Packages (+12 Dependent packages)
Total download size: 104 M
Installed size: 430 M
Is this ok [y/d/N]: y
...
Retrieving key from https://download.docker.com/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
...
Installed:
  containerd.io.x86_64 0:1.4.4-3.1.el7          docker-ce.x86_64 3:20.10.5-3.el7         
  docker-ce-cli.x86_64 1:20.10.5-3.el7         

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.5-4.el7                                                  
  checkpolicy.x86_64 0:2.5-8.el7                                                          
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8                                      
  docker-ce-rootless-extras.x86_64 0:20.10.5-3.el7                                        
  fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                                                   
  fuse3-libs.x86_64 0:3.6.1-4.el7                                                         
  libcgroup.x86_64 0:0.41-21.el7                                                          
  libsemanage-python.x86_64 0:2.5-14.el7                                                  
  policycoreutils-python.x86_64 0:2.5-34.el7                                              
  python-IPy.noarch 0:0.75-6.el7                                                          
  setools-libs.x86_64 0:3.3.8-4.el7                                                       
  slirp4netns.x86_64 0:0.4.3-4.el7_8                                                      

Complete!
```
#### 指定版本安装
如果需要指定版本，先查看版本：
```
[root@VM-0-6-centos ~]# yum list docker-ce --showduplicates | sort -r
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror, langpacks
Installed Packages
docker-ce.x86_64            3:20.10.5-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.5-3.el7                    @docker-ce-stable
docker-ce.x86_64            3:20.10.4-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.3-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.2-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.1-3.el7                    docker-ce-stable 
...
```
命令示例：
```
# yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
说明：
- &#60;VERSION_STRING>是完整软件包名称，格式是：软件包名称（docker-ce）加上版本字符串（第二列，从第一个冒号（:）一直到第一个连字符），并用连字符（-）分隔
- 例如示例中第一条记录完整版本号就是：docker-ce-20.10.5

### 启动测试
启动Docker：
```
[root@VM-0-6-centos ~]# systemctl start docker
```
运行hello-world映像：
```
[root@VM-0-6-centos ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete 
Digest: sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24
Status: Downloaded newer image for hello-world:lates
...
```
找不到，但是自动下载了，然后执行了，可以再来一次：
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
说明安装成功。

## 待补充
