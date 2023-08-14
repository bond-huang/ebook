# Nginx-Win基础操作
## Nginx下载安装
官方网站：[http://nginx.org/](http://nginx.org/)    
下载：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)    
下载Windows-1.24.0版本下载后解压即可。

### 示例简单WEB服务
conf目录下nginx.conf文件修改配置，端口号改为81，配置如下：
```ini
server {
        listen     81;
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
在D盘新建个文件夹Download，放入部分文件。

&#8195;&#8195;修改Windows系统host表，将big1000.com解析到本地localhost，host文件路径：`C:\Windows\System32\drivers\etc\hosts`，添加条目如下：
```
127.0.0.1   big1000.com
```
&#8195;&#8195;在Nginx安装目录的绝对路径输入框种输入CMD进入命令行，执行命令`nginx.exe`或`start nginx`即启动nginx（双击exe文件也可以）：

```
D:\软件\nginx-1.24.0>start nginx
```
查看进程：
```
C:\Users\admin>tasklist |findstr "nginx.exe"
nginx.exe                    28052 Console                    1      8,220 K
nginx.exe                     1084 Console                    1      8,508 K
nginx.exe                    21404 Console                    1      8,260 K
nginx.exe                     8744 Console                    1      8,564 K
```
验证WEB服务器是否启动：
- 浏览器输入：big1000.com:81可以访问Nginx主页：Welcome to nginx!
- 浏览器输入：big1000.com/download可以进去D盘Download目录，点击文件可以下载文件

修改配置后重新加载Nginx：
```
D:\软件\nginx-1.24.0>nginx -s reload
```
关闭Nginx：
```
D:\软件\nginx-1.24.0>nginx -s stop
```
## 待补充