# Nginx-Win常见问题
## 问题排查思路
&#8195;&#8195;在Windows下运行Nginx时，如果浏览器无法访问localhost，可能有几个原因导致。以下是一些常见的排查步骤：
- Nginx服务是否启动： 确保Nginx已经成功启动并正在运行。可以在Nginx的安装目录中找到启动脚本，并双击运行或通过命令行运行`nginx.exe`来启动服务
- 检查端口冲突： Nginx默认监听80端口，如果该端口被其他应用程序占用，Nginx将无法正常工作。可以通过运行`netstat -an`命令查看端口占用情况
- 防火墙和安全软件： 确保防火墙或安全软件没有阻止Nginx的网络访问。在防火墙中允许Nginx的80端口可以解决这个问题
- 检查Nginx配置文件： 检查Nginx的配置文件是否正确，特别是监听的端口和站点根目录等信息。配置文件默认位于`nginx/conf/nginx.conf`。
- 查看Nginx错误日志： 如果Nginx启动有问题，查看Nginx的错误日志文件，通常位于`nginx/logs/error.log`，可以帮助你找到具体错误原因。
- 管理员权限运行： 确保以管理员权限运行Nginx。有时候，特别是在80端口上运行服务，需要管理员权限才能绑定端口

## 80端口占用问题
&#8195;&#8195;Windows系统下80端口大概率不能直接使用，使用其它端口访问后面需要加入端口号，使用80端口就不需要。如果被占用，使用80端口时候Nginx启动报错如下：
```
2023/07/20 01:08:29 [emerg] 13228#17568: bind() to 0.0.0.0:80 failed (10013: An attempt was made to access a socket in a way forbidden by its access permissions)
```
&#8195;&#8195;管理员账户进入PowerShell，查看80端口使用情况（如果没有第一条这种占用的，那就直接改Nginx端口到80启动Nginx即可）：
```
PS C:\Users\admin> netstat -ano |findstr :80
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       5304
  TCP    127.0.0.1:80           127.0.0.1:61900        ESTABLISHED     5304
  TCP    127.0.0.1:80           127.0.0.1:61985        ESTABLISHED     5304
```
&#8195;&#8195;查看端口5304可能是svchost.exe使用了，此时千万不用尝试干掉此进程，干掉就会蓝屏。修改下注册表：
- 以管理员身份运行regedit
- 打开键值: HKEY LOCAL MACHINE--SYSTEM--CurrentControlSet--services--HTTP
- 在右边找到Start这一项，将其改为0;
- 重启系统，System进程不会占用80端口

重启系统后再次查看：
```
PS C:\windows\system32> netstat -ano |findstr :80
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       5080
```
查看5080端口
```
PS C:\windows\system32> tasklist |findstr "5080"
dhcpsrv.exe                   5080 Services                   0      7,964 K
svchost.exe                  15080 Services                   0      8,740 K
```
干掉5080那个dhcpsrv.exe，如果是还是svchost.exe占用的话，干掉电脑就会蓝屏：
```
PS C:\windows\system32> taskkill /f /t /im "dhcpsrv.exe"
成功: 已终止 PID 5080 (属于 PID 856 子进程)的进程。
```
再次查看就没有进程使用80端口了。还需要防火墙开放80端口的访问：
```
PS C:\windows\system32> netsh advfirewall firewall add rule name="允许80端口访问" dir=in action=allow protocol=TCP localport=80
确定。

PS C:\windows\system32>
```
注意：上面命令放开端口后，重启后会失效，不要重启系统。     
conf目录下nginx.conf修改配置，listen端口号改为80:
```ini
server {
        listen       80;
		autoindex on;
        server_name  big1000.com;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
        # 配置处理根目录请求的相关设置
		}
        location /download		{
		# 允许所有访问
        allow all;
		# 禁止访问时返回403错误
        deny all;
        alias D:\Download;
        }
}
```
启动Nginx：
```
D:\软件\nginx-1.24.0>start nginx

D:\软件\nginx-1.24.0>tasklist |findstr "nginx.exe"
nginx.exe                    12208 Console                    1      8,180 K
nginx.exe                    11720 Console                    1      8,540 K
```
再次查看端口使用情况：
```
PS C:\windows\system32> netstat -ano |findstr :80
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       11720
PS C:\windows\system32> tasklist |findstr "11720"
nginx.exe                    11720 Console                    1      8,540 K
```
Nginx可以使用80端口了。清除浏览器的缓存：
- 浏览器输入：big1000.com可以访问Nginx主页：Welcome to nginx!
- 浏览器输入：big1000.com/download可以进去D盘Download目录，点击文件可以下载文件

## 文件夹访问问题
### 文件访问受限
配置文件如下：
```ini
server {
        listen       80;
        server_name  big1000.com;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
        # 配置处理根目录请求的相关设置
		}
        location /download		{
		# 允许所有访问
        allow all;
		# 禁止访问时返回403错误
        deny all;
        alias D:\Download;
        }
}
```
浏览器输入`big1000.com`正常进入Nginx主页，访问`big1000.com/download`报错403，查看日志报错如下：
```
directory index of "D:\Download/" is forbidden
```
添加`autoindex on;`：

```ini
    server {
        listen       80;
		autoindex on;
        server_name  servicecenter.xiv.ibm.com;
        ...
    }
```
&#8195;&#8195;如果在Nginx的配置中添加了`autoindex on;`，那么在没有指定默认索引文件的情况下，Nginx会自动列出目录的内容。这通常用于在没有提供默认首页文件的情况下，浏览目录中的文件列表。重新加载Nginx：
```
nginx -s reload
```
访问`big1000.com/download`即可以到指定目录。
## 待补充