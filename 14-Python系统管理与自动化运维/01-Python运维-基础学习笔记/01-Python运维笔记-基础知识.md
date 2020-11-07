# Python运维笔记-基础知识

## Python内置小工具
### 下载服务器
&#8195;&#8195;Linux传输文件通常使用ftp等，用命令有时感觉不方便，Python有个内置web服务器，可以作为一个下载服务器，在Python3中，使用示例：
```
[root@redhat8 shell]# ls
basis  for  function  gawk  if-for  input  instance  output  regular  sed  sed_gawk  test
[root@redhat8 shell]# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
&#8195;&#8195;然后打开浏览器，输入地址`http://192.168.18.131:8000/`既可，可以看到类似FTP下载页面，使用笔记方便。如果目录下存在一个index.html的文件，默认显示该文件内容，如果没有，默认显示当前目录下的文件列表。

在Python2中的使用方法：
```
python -m SimpleHTTPServer
```
### 字符串转换为JSON
&#8195;&#8195;JSON是一种轻量级的数据交换格式，网上可以搜索到在线JSON格式化工具，当然可以在命令行的Python解析器来解析JSOS串。使用示例：
```
[root@redhat8 shell]# echo '{"job":"devops","name":"bond","sex":"male"}'|python3 -m json.t
ool{
    "job": "devops",
    "name": "bond",
    "sex": "male"
}
```
还可以自动对齐和格式化，示例：
```
[root@redhat8 shell]# echo '{"address":{"province":"guangdong","city":"shenzhen"},"name":"
bond","sex":"male"}'|python3 -m json.tool
{
    "address": {
        "province": "guangdong",
        "city": "shenzhen"
    },
    "name": "bond",
    "sex": "male"
}
```
### 检查第三方库
检查第三方库是否正确安装，只需要尝试import即可，不报错就没问题：
```
[root@redhat8 shell]# python3
Python 3.6.8 (default, Jan 11 2019, 02:17:16) 
[GCC 8.2.1 20180905 (Red Hat 8.2.1-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> improt paramiko
```
或者使用`-c`参数快读执行`import`语句:
```
[root@redhat8 python]# python3 -c "import paramiko"
```
## pip高级用法
Python生态主流的包管理工具是pip。
### pip介绍
