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
## web文件服务器
### 部署使用
配置文件示例（AI给的验证可用）：
```conf
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    # 大文件优化，非常关键，解决几十GB文件下载中断
    tcp_nopush     on;
    keepalive_timeout  65;
    # 关闭压缩，大文件不压缩节省CPU
    gzip  off;
    # 缓冲区优化，大文件下载
    proxy_buffer_size 4k;
    client_body_buffer_size 128k;

    server {
        listen       8080;
        # 监听0.0.0.0，局域网其他电脑可访问
        server_name  0.0.0.0;

        # 共享D盘全部文件
        location / {
            alias D:/;
            autoindex on;              # 开启目录浏览
            autoindex_exact_size off;  # 显示人性化文件大小
            autoindex_localtime on;    # 使用本地时间
			charset GBK;   # 解决windows中文文件名乱码(解决不了)

            # 账号密码认证
            auth_basic "login";
			auth_basic_user_file C:/Users/ZYAT/Desktop/ffilesoft/nginx-1.31.6/conf/.htpasswd;

            # 大文件下载优化
            sendfile on;
            max_ranges 1;
        }
    }
}
```
conf目录下创建`.htpasswd`文件，写入一下内容：
```
admin:$apr1$7ijp9YzY$jOlRH91UVsOMruWaDiBar.
```
说明：
- admin密码就是123456，我生成后直接粘贴的
- 可以使用htpasswd工具生成`.htpasswd`文件
- 中文乱码问题解决不了，回头解决了再补充

在htpasswd工具：[https://toolgen.cn/password/htpasswd](https://toolgen.cn/password/htpasswd)
### 常见问题
输入密码后进不去，报错示例：
```
2026/09/22 15:49:18 [crit] 25548#32588: *1 CreateFile() "C:\Users\ZYAT\Desktop\ffilesoft
ginx-1.31.6\conf\.htpasswd" failed (123: The filename, directory name, or volume label syntax is incorrect), client: 127.0.0.1, server: 0.0.0.0, request: "GET / HTTP/1.1", host: "127.0.0.1:8080"
```
原因：路径不能用`\`，要用`/`

## 待补充