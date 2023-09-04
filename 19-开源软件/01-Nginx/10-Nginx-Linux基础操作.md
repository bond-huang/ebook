# Nginx-Linux基础操作
## 安装部署
使用系统为阿里云的CentOS8.2版本：
安装Nginx：
```
[root@centos82 ~]# yum install nginx
......
Installed:
  gd-2.2.5-7.el8.x86_64
  jbigkit-libs-2.1-14.el8.x86_64
  libXpm-3.5.12-8.el8.x86_64
  libjpeg-turbo-1.5.3-12.el8.x86_64
  libtiff-4.0.9-20.el8.x86_64
  libwebp-1.0.0-5.el8.x86_64
  libxslt-1.1.32-6.el8.x86_64
  nginx-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
  nginx-all-modules-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch
  nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch
  nginx-mod-http-image-filter-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
  nginx-mod-http-perl-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
  nginx-mod-http-xslt-filter-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
  nginx-mod-mail-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
  nginx-mod-stream-1:1.14.1-9.module_el8.0.0+184+e34fea82.x86_64
```
默认配置文件位置：`/etc/nginx/nginx.conf`，启动服务：
```
[root@centos82 ~]# systemctl start nginx
[root@centos82 ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-08-14 21:20:17 CST; 2s ago
  Process: 398530 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 398528 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 398526 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 398531 (nginx)
    Tasks: 3 (limit: 11496)
   Memory: 5.3M
   CGroup: /system.slice/nginx.service
           ├─398531 nginx: master process /usr/sbin/nginx
           ├─398532 nginx: worker process
           └─398533 nginx: worker process

Aug 14 21:20:17 centos82 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Aug 14 21:20:17 centos82 nginx[398528]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 14 21:20:17 centos82 nginx[398528]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Aug 14 21:20:17 centos82 systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@centos82 ~]# ps -aux |grep nginx
root      398531  0.0  0.1 117796  2172 ?        Ss   21:20   0:00 nginx: master process /usr/sbin/nginx
nginx     398532  0.0  0.4 149304  8220 ?        S    21:20   0:00 nginx: worker process
nginx     398533  0.0  0.4 149304  8220 ?        S    21:20   0:00 nginx: worker process
root      398546  0.0  0.0  12132  1148 pts/3    S+   21:20   0:00 grep --color=auto nginx
```
WEB登录到服务器地址，即可访问Nginx主页：Welcome to nginx on Red Hat Enterprise Linux!

停止Nginx：
```
[root@centos82 ~]# systemctl stop nginx
[root@centos82 ~]# ps -aux |grep nginx
root      398796  0.0  0.0  12132  1156 pts/4    S+   21:27   0:00 grep --color=auto nginx
```
## 服务使用
### 使用https
&#8195;&#8195;一般云平台服务器都可以免费下载证书。我在阿里云下载证书，一共两个文件`.pem`和`.key`文件，上传到服务器默认`/etc/nginx`目录。Nginx配置示例：
```ini
    # HTTPS server
    #
    server {
       listen       443 ssl;
        server_name  localhost;

        ssl_certificate      big1000.com.pem;
        ssl_certificate_key  big1000.com.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   /home/gitbook/ebook/_book;
            index  index.html;
        }
		location /status {
                stub_status;
                access_log off;
                allow 127.0.0.1;
                allow ::1;
                deny all;
		    }
    }
```
重启nginx服务即可。
## 常用命令
### 常用查看命令
查看当前nginx使用配置文件：
```
[root@centos82 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
## 待补充