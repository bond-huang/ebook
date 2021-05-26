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

## 待补充
