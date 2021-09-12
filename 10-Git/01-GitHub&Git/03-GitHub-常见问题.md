# GitHub-常见问题
记录使用过程中遇到的问题。
## Git Bash问题
### 提交问题
使用`git add .`时候报错如下：
```
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in src/plugins/element.js.
```
原因是我一直在本地写代码，后面才在GitHub创建的仓库，执行下面命令：
```
$ git rm -r --cached .
rm '.gitignore'
rm 'README.md'
rm 'babel.config.js'
rm 'package-lock.json'
rm 'package.json'
...
$ git config core.autocrlf false
```
然后查看状态：
```
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    .gitignore
        deleted:    README.md
        deleted:    babel.config.js
...
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        README.md
        babel.config.js
        package-lock.json
        package.json
        public/
        src/
```
根据提示添加file并查看状态：
```
$ git add .
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   package-lock.json
        modified:   package.json
        modified:   src/App.vue
        new file:   src/api/demo.js
        new file:   src/api/login.js
        new file:   src/api/menu.js
...
```
提交到GitHub：
```
$ git commit -m "First commit"
[master 9e1f8ee] First commit
 41 files changed, 2111 insertions(+), 235 deletions(-)
 rewrite src/App.vue (94%)
 create mode 100644 src/api/demo.js
 create mode 100644 src/api/login.js
...
```
去GitHub上看发现并没有提交，shm是我的仓库名称：
```
$ git push shm
fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream shm master
```
使用命令`git remote -v`查看指向的repository，是空的：
```
$ git remote -v
```
原因是没有将本地与远程仓库关联，使用下面命令关联：
```
$ git remote add origin git@github.com:bond-huang/shm.git
```
再次push：
```
$ git push
fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin master

$ git push --set-upstream origin master
Enumerating objects: 79, done.
Counting objects: 100% (79/79), done.
Delta compression using up to 4 threads
Compressing objects: 100% (72/72), done.
Writing objects: 100% (79/79), 260.74 KiB | 953.00 KiB/s, done.
Total 79 (delta 6), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (6/6), done.
remote:
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/bond-huang/shm/pull/new/master
remote:
To github.com:bond-huang/shm.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
打开GitHub，会有如下提示：
```
master had recent pushes 2 minutes ago 
```
查看状态：
```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```
&#8195;&#8195;进入GitHub，代码就在master分支下面。如果master不是默认的分支，在图形界面切换一下即可`Switch default branch to another branch`。

### ssl问题
&#8195;&#8195;最近换了新电脑，从github上克隆下来的仓库，配置好keygen后，运行命令`git commit`后提示需要设置用户名和邮箱，设置即可。运行命令`git push`后弹出对话框要输入用户密码，选择网页认证成功后，报错如下：
```
$ git push
fatal: 发送请求时出错。
fatal: 无法连接到远程服务器
fatal: 由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。 13.229.188.59:443
fatal: unable to access 'https://github.com/bond-huang/ebook.git/': OpenSSL SSL_read: Connection was reset, errno 10054
```
网上查说要解除ssl认证，运行了如下命令：
```
$ git config --global http.sslVerify "false"
```
参考链接：[fatal: unable to access ‘xxx.git/‘: OpenSSL SSL_read: Connection was reset, errno 10054](https://joycez.blog.csdn.net/article/details/116523092)

然后再次尝试，在弹出对话框成功认证后，push成功：
```
warning: ----------------- SECURITY WARNING ----------------
warning: | TLS certificate verification has been disabled! |
warning: ---------------------------------------------------
warning: HTTPS connections may not be secure. See https://aka.ms/gcmcore-tlsverify for more information.
warning: ----------------- SECURITY WARNING ----------------
warning: | TLS certificate verification has been disabled! |
warning: ---------------------------------------------------
warning: HTTPS connections may not be secure. See https://aka.ms/gcmcore-tlsverify for more information.
Enumerating objects: 37, done.
Counting objects: 100% (37/37), done.
Delta compression using up to 12 threads
Compressing objects: 100% (20/20), done.
Writing objects: 100% (20/20), 9.90 KiB | 3.30 MiB/s, done.
Total 20 (delta 15), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (15/15), completed with 15 local objects.
To https://github.com/bond-huang/ebook.git
   3a23091..2a40d1a  master -> master
```
下次再push时候就不会弹出认证的对话框了：
```
$ git push
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 12 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 1.53 KiB | 1.53 MiB/s, done.
Total 6 (delta 5), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (5/5), completed with 5 local objects.
To https://github.com/bond-huang/ebook.git
   2a40d1a..bd14a23  master -> master
```
还查到一个方法,下次再遇到试试看，命令如下：
```
$ git config --global http.sslBackend "openssl"
```
## VScode中标记
&#8195;&#8195;将本地项目连接到远程GitHub仓库时候，使用VScode编辑代码后文件后面会有相应的字母信息，文件名称也会显示对应的颜色，代码中也有标记的代码行，说明如下：
- M modified
    - 在github中添加过该文件，然后对在VScode中对文件进行了修改，文件后标记M，并且文件名显示黄色
    - 在代码中，修改后的代码行前面有个蓝色标记，点开可以查看到修改前后的对比信息
- U untracked
    - 在本地新建了文件，还未提交到GitHub上，就会标记U
- D delete
    - 删除了的文件，VScode会记录下这个状态
- 3,M
    - 表示有3个错误，并且是modified的

参考博客：[vscode-git中的U,M和D文件标记含义](https://blog.csdn.net/qsj0606/article/details/114439860)
## 待补充
