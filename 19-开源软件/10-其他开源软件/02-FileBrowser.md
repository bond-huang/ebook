# FileBrowser
## 安装下载
下载链接：[https://github.com/filebrowser/filebrowser/releases](https://github.com/filebrowser/filebrowser/releases)

Windows版本解压后点击exe就运行了。
## 配置部署
### 配置使用
&#8195;&#8195;在`filebrowser.exe`同目录新建文本文档，写入下面内容，例如共享路径是`D:\sharefile`：
```ini
@echo off
taskkill /f /im filebrowser.exe >nul
filebrowser.exe -a 0.0.0.0 -p 8080 -r "D:\sharefile"
```
或者CMD运行：
```sh
filebrowser.exe -a 0.0.0.0 -p 8080 -r "D:\sharefile" -b /files
```
参数说明：
- `-r "D:\sharefile"`： 指定共享根目录。如果直接用D盘作为根目录：写成`-r "D:/"`
- `-a 0.0.0.0`：允许局域网其他电脑访问；如果只本机访问写127.0.0.1

&#8195;&#8195;保存文件，重命名为`start.bat`，双击运行，然后浏览器刷新 `http://127.0.0.1:8080/files/`，就进入指定目录了。

## 常见问题
### 密码忘记
&#8195;&#8195;数据库文件`filebrowser.db`里面只保存密码哈希值，单向加密，无法反向解密出原始密码，只能重置新密码，Windows重置步骤如下：
- 先关闭当前运行的FileBrowser窗口
- CMD进入到`filebrowser.exe`所在文件夹
- 执行下面命令，把`123456`替换成新密码：
    ```
    .\filebrowser.exe users update admin --password A123456789abc
    ```
- 执行完成，再重新双击bat启动filebrowser

## FileBrowser Quantum
### FileBrowser Quantum & onlyoffice
相关链接：
- FileBrowser Quantum下载地址：[https://github.com/gtsteffaniak/filebrowser/releases/tag/v2.0.8-beta](https://github.com/gtsteffaniak/filebrowser/releases/tag/v2.0.8-beta)
- FileBrowser Quantum文档链接：[https://filebrowserquantum.com/en/docs/getting-started/windows/](https://filebrowserquantum.com/en/docs/getting-started/windows/)
- onlyoffice桌面版下载链接：[https://www.onlyoffice.com/desktop](https://www.onlyoffice.com/desktop)
- onlyoffice server下载链接：[https://www.onlyoffice.com/nl/download](https://www.onlyoffice.com/nl/download)

server版本不免费，换一种方案。找到社区版本继续。需要安装的软件：
- RabbitMQ：[https://www.rabbitmq.com/docs/install-windows](https://www.rabbitmq.com/docs/install-windows)
- Erlang/OTP：[https://www.erlang.org/downloads](https://www.erlang.org/downloads)
- PostgreSQL：[https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
- onlyoffice server下载链接：[https://github.com/ONLYOFFICE/DocumentServer/releases/download/v9.4.0/onlyoffice-documentserver.exe](https://github.com/ONLYOFFICE/DocumentServer/releases/download/v9.4.0/onlyoffice-documentserver.exe)

&#8195;&#8195;先安装Erlang/OTP和RabbitMQ，然后安装PostgreSQL，提示输入密码随便输入个，JWT提示点确定即可。安装完成后，JWT默认已启用，程序自动生成了随机密钥。密钥存放路径：
`%ProgramFiles%\ONLYOFFICE\DocumentServer\config\local.json`，打开文件，在`services.CoAuthoring.secret.inbox.string`这个参数里面。复制秘钥。

配置FileBrowser Quantum的config.yaml，填入onlyoffice地址和密钥：
```yml
integrations:
  office:
    url: "http://192.168.1.1"
    secret: "这里粘贴刚才复制的JWT密钥"
    viewOnly: true
```
&#8195;&#8195;如果FileBrowser和OnlyOffice在同一 Windows主机，url 填`http://127.0.0.1`；其他电脑访问预览，用本机局域网IP `http://192.168.1.1`。

OnlyOffice服务状态检查：
```
Running  DsConverterSvc      文档转换服务（已正常运行）
Running  DsDocServiceSvc     文档主服务（已正常运行）
Running  DsProxySvc          代理服务（已正常运行）
Stopped  DsExampleSvc        示例服务（**这个不用启动！**只是demo示例，不需要）
```
&#8195;&#8195;`http://127.0.0.1`或者本机局域网IP，例如 `http://192.168.x.x`能打开ONLYOFFICE欢迎页面，代表服务就绪。准备启动FileBrowser Quantum，启动报错，示例：
```
PS C:\Users\ZYAT\Desktop\ffilesoft> .\filebrowser.exe -c config.yaml
2026/09/23 10:59:23 [DEBUG] Default SQLite driver initialized
2026/09/23 10:59:23 [ERROR] unable to load config, waiting 5 seconds before exiting...
2026/09/23 10:59:28 [FATAL] error parsing YAML data: [231:1] mapping key "integrations" already defined at [183:1]
  228 |   disableRateLimit: false
  229 |
  230 |
> 231 | integrations:
        ^
  232 |   office:
  233 |     url: "http://192.168.1.1"
  234 |     secret: "xHQ1yr0Hn094rtrhZmr3qXAgfPEDJh"
```
&#8195;&#8195;写重复了，把贴的内容对应的183行去既可以。启动正常，使用`http://192.168.1.1:8080`（FileBrowser里面配置的8080端口）访问正常，可以预览PPTX，xlsx及老版本的doc。

### FileBrowser Quantum & LibreOffice
LibreOffice下载链接：[https://vec.libreoffice.org/](https://vec.libreoffice.org/)

没法直接接入FileBrowser Quantum，算了。

## 待补充