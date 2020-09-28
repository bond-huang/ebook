# TravisCI-基础操作
&#8195;&#8195;配置Travis-CI自动构建步骤不难，目前用到的知识也不多，以为后期不会再用到了，就没打算写，结果后来由于一些原因又搞了两次，还是记下来避免忘记。记录一些构建过程中的问题，以及记录我配置好后出现问题后（被GitHub撤销访问）重新配置，对于完整构建可以参考Lyon的分享：[GitHub Pages&Gitbook&Travis CI持续构建博客](https://lyonyang.github.io/blogs/09-Linux/Git/GitHub%20Pages&Gitbook&Travis%20CI%E6%8C%81%E7%BB%AD%E6%9E%84%E5%BB%BA%E5%8D%9A%E5%AE%A2.html)
### Travis-CI重新配置
近期收到GitHub的邮件，主要内容是：As a precautionary measure, we have revoked the OAuth token. A new token will need to be generated in order to continue using OAuth to authenticate to GitHub.

被GitHub撤销了访问，需要重新配置。
##### GitHub生成Personal access tokens
步骤如下：
- 登录到GitHub，点击右上角头像，选择选项"Settings"
- 选择选项"Developer settings"
- 选择选项"Personal access tokens"
- 点击选项"Generate new token",如果有之前的用不了的就"Delete"删除掉
- 在"Note"选项中输入自定义的名字,权限选择里面勾上"repo"所有
- 点击选项"Generate token"
- 复制生成的token，忘了复制就重新来一遍

##### Travis-CI配置
步骤如下：
- 登录到Travis-CI，进入到个人仓库页面
- 点击右边的选项"More options",选择选项"Settings"
- "General"里面我勾选了"Build pushed branches"和"Build pushed pull requests"
- "Auto Cancellation"里面我全勾选了，说实话作为小白对描述不是很理解
- 在"Environment Varialbes"中"NAME"选项里面写入名字，名字跟脚本deploy.sh里面写入的一样
- 在"Environment Varialbes"中"VALUE"选项里粘贴之前复制的token
- 然后点击右边的"Add"选项
- 添加成功后，点击右边的选项"More options",选择选项"Trigger build"开始构建

之前构建成功过的，这次一般都会成功的，因为出问题只是GitHub取消了访问。

### Travis-CI初始配置问题
在第一次用Travis-CI配置自动构建的时候，"Trigger build"后各种问题，调试了好久第32次才通过。忘了作详细记录，目前记得的简单记录下。

##### gitbook版本问题
在.travis.yml中，有安装gitbook版本，但是试了好几个版本在那一项都过不了，提示内容忘了截取，最后没有指定版本并且强制安装：
```
install:
  - "npm install -g gitbook"
  - "npm install -g gitbook-cli --force"
```
##### JS版本问题
改了上面后发现JS版本也不过，高低版本都不行好像，最后改成了8：
```
node_js:
  - "8"
```
### Travis-CI更新异常
问题描述：能运行但是不能自动添加新的内容      
报错及日志如下：
```
The command bash deploy.sh exited with 1.
log:
Switched to a new branch 'gitbook'
On branch gitbook
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   SUMMARY.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	node_modules/
	travis_wait_2044.log

no changes added to commit (use "git add" and/or "git commit -a")
```
查了下应该是分支冲突了，SUMMARY.md文件提交不了，修改几次脚本都不行，直接用绝招：

&#8195;&#8195;在GitHub里面`master-->View all branches-->`删掉所有由Travis创建的分支，travis-ci上的缓存也清理了，但是还是不行。后来把出错当天更新的文档删掉了，然后gitbook分支也删了，再重新运行就好了。当天可疑操作就是文件名后缀写重复了，不知道是不是这个原因：
```
02-Flask-项目构建笔记_1.md.md
``
