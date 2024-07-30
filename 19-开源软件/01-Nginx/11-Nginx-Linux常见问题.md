# Nginx-Linux常见问题
## 启动问题
### 启动不了failed状态
查看nginx状态示例如下：
```
[root@huang nginx]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2024-07-30 19:13:55 CST; 6s ago
  Process: 13890 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 13888 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)

Jul 30 19:13:55 huang systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jul 30 19:13:55 huang nginx[13890]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jul 30 19:13:55 huang nginx[13890]: nginx: [emerg] open() "/etc/nginx/logs/access.log" failed (2: No such file or directory)
Jul 30 19:13:55 huang nginx[13890]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jul 30 19:13:55 huang systemd[1]: nginx.service: control process exited, code=exited status=1
Jul 30 19:13:55 huang systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Jul 30 19:13:55 huang systemd[1]: Unit nginx.service entered failed state.
Jul 30 19:13:55 huang systemd[1]: nginx.service failed.
Jul 30 19:13:57 huang systemd[1]: Unit nginx.service cannot be reloaded because it is inactive.
```
很明显下面那条说明了原因：
```
Jul 30 19:13:55 huang nginx[13890]: nginx: [emerg] open() "/etc/nginx/logs/access.log" failed (2: No such file or directory)
```
&#8195;&#8195;在启动的nginx的配置文件中定义了日志的位置，但是实际上没有，需要添加对应的日志目录及文件，并且注意目录和文件的权限。修改完成后重启nginx即可。
## 待补充