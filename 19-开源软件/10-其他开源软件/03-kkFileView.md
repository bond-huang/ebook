# kkFileView

## 简介
&#8195;&#8195;kkFileView是一款‌基于Spring Boot开发的开源文件在线预览解决方案‌，用户无需下载文件即可在浏览器中直接查看Office文档、PDF、图片、CAD 等多种格式内容

官方源码托管于
- GitHub：[github.com/kekingcn/kkFileView](github.com/kekingcn/kkFileView)
- Gitee：[gitee.com/kekingcn/file-online-preview](gitee.com/kekingcn/file-online-preview)

官网地址为：[kkview.cn](kkview.cn)

下载地址：
- [https://github.com/kekingcn/kkFileView/releases](https://github.com/kekingcn/kkFileView/releases)
- [https://gitee.com/kekingcn/file-online-preview/releases#release-v4.4.0](https://gitee.com/kekingcn/file-online-preview/releases#release-v4.4.0)
‌‌
## 配置使用
### windows配置使用
使用的是windwos kkFileView-4.0.0版本，`bin`目录下运行`startup.bat`：
```
Using KKFILEVIEW_BIN_FOLDER C:\Users\ZYAT\Desktop\ffilesoft\kkFileView-4.0.0\bin
Starting kkFileView...
Please check log file in ../log/kkFileView.log for more information
You can get help in our official homesite: https://kkFileView.keking.cn
If this project is helpful to you, please star it on https://gitee.com/kekingcn/file-online-preview/stargazers
```
查看`kkFileView.log`提示成功了
```
2026-09-22 23:14:22.364  INFO 49200 --- [           main] o.s.b.web.embedded.jetty.JettyWebServer  : Jetty started on port(s) 8012 (http/1.1) with context path '/'
2026-09-22 23:14:22.386  INFO 49200 --- [           main] cn.keking.ServerMain                     : kkFileView 服务启动完成，耗时:12.9831524s，演示页请访问: http://127.0.0.1:8012 
```
打开页面测试成功了
## 待补充