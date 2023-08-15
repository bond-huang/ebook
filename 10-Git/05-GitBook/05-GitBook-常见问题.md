# GitBook-常见问题
## 安装报错
安装GitBook报错：
```
[root@centos82 ~]# npm install -g gitbook
npm WARN deprecated graceful-fs@3.0.5: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
npm WARN deprecated chokidar@1.0.6: Chokidar 2 will break on node v14+. Upgrade to chokidar 3 with 15x less dependencies.
npm WARN deprecated nunjucks@2.2.0: potential XSS vulnerability in autoescape mode, and with escape filter was fixed in v2.4.3
npm WARN deprecated request@2.51.0: request has been deprecated, see https://github.com/request/request/issues/3142
```
安装完成gitbook-cli后，运行`gitbook -V`自动安装：
```
[root@centos82 ~]# gitbook -V
CLI version: 2.3.2
Installing GitBook 3.2.3
gitbook@3.2.3 ../tmp/tmp-8300BiiqfilAM1im/node_modules/gitbook
......
└── npm@3.9.2
GitBook version: 3.2.3
```
## 构建报错
构建时候报错示例：
```
[gitbook@centos82 ebook]$ gitbook build .
info: 7 plugins are installed
info: 12 explicitly listed

Error: Couldn't locate plugins "advanced-emoji, toggle-chapters, splitter, github, search-plus, anchor-navigation-ex-toc, editlink, copy-code-button, theme-comscore", Run 'gitbook install' to install plugins from registry.
```
构建前记得安装插件，安装内容在当前目录`book.json`文件中：
```
[gitbook@centos82 ebook]$ gitbook install
```
构建过程中有语法报错，示例
```
error: error while generating page "05-IBM_Operating_System/07-RHEL-Ansible学习笔记/05-Ansible-文件部署到受管主机.md":

Template render error: (/home/gitbook/ebook/05-IBM_Operating_System/07-RHEL-Ansible学习笔记/05-Ansible-文件部署到受管主机.md) [Line 263, Column 14]
  unknown block tag: EXPR
```
&#8195;&#8195;此md文件中是jinja2相关说明，涉及到了&#123;&#37; &#37;&#125;等符号，在GitBook中也代表特殊意义，需要用实体编号代替。具体编号参考Markdown相关说明。

## 待补充