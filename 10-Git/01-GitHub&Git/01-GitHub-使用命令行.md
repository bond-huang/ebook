# GitHub使用命令行
刚接触的一般会使用GitHub的web界面对自己的Repositories进行操作，简单方便。但是有些操作在web界面是无法完成的，例如就遇到想改文件夹名字，发现没找到，只能用命令行进行；在大批量的修改上传代码的时候，用命令行操作也能体现优势。
### 安装Git
官方下载链接:[https://git-scm.com/downloads](https://git-scm.com/downloads)
Windows下直接运行按照即可。
Debian和Ubuntu中安装方法如下：
```shell
apt-get install git
```
其它Linux或者Unix参考官方链接：[https://git-scm.com/download/linux](https://git-scm.com/download/linux)
### 配置Git
我使用的是Windows，安装成功后点击开始菜单，找到Git Bash点击打开命令行
依次输入如下命令：
```shell
#初始化
git init
#生成密钥，输入GitHub注册邮箱
ssh–keygen –t rsa –C <email add>
```
生成的密钥在目录<User directory>/.ssh下面，打开.pub的文件，复制里面的所有内容。
在GitHub WEB界面依次进行如下操作：
- 点击右上角头像，选择选项“setting”
- 点击左边导航栏“SSH and GPG keys"
- 点击选项“New SSH key”
- “Title”自定义，“Kek”栏里把刚才复制的内容粘贴上去
- 点击选项“Add SSH key”

添加成功后回到Git Bash，输入如下命令进行验证：
```shell
ssh -T git@github.com
```
提示成功即可以了。
### 克隆仓库
我是预先再GitHub已经有Repositories了，所以不需要新建，直接clone过来即可。
再GitHub WEB界面依次进行如下操作：
- 进入对应需要clone的Repositories界面
- 点击绿色“Code”选项
- 如果显示“Clone with HTTPS”，则点击右边“Use SSH"
- 显示为“Clone with SSH”时，在链接右边有复制按钮，点击复制链接

进入到Git，输入如下命令进行克隆：
```shell
git clone git@github.com:bond-huang/huang.git
```
等待片刻，即克隆到本地了，输入ls命令也查看目录，有Repositories名字的目录，CD进入目录，就可以对远程Repositories进行相应操作。
