# TravisCI-基础操作
&#8195;&#8195;配置Travis-CI自动构建步骤不难，目前用到的知识也不多，以为后期不会再用到了，就没打算写，结果后来由于一些原因又搞了两次，还是记下来避免忘记。记录一些构建过程中的问题，以及记录我配置好后出现问题后（被GitHub撤销访问）重新配置，对于完整构建可以参考Lyon的分享：[GitHub Pages&Gitbook&Travis CI持续构建博客](https://lyonyang.github.io/blogs/09-Linux/Git/GitHub%20Pages&Gitbook&Travis%20CI%E6%8C%81%E7%BB%AD%E6%9E%84%E5%BB%BA%E5%8D%9A%E5%AE%A2.html)
## Travis-CI重新配置
近期收到GitHub的邮件，主要内容是：As a precautionary measure, we have revoked the OAuth token. A new token will need to be generated in order to continue using OAuth to authenticate to GitHub.

被GitHub撤销了访问，需要重新配置。
### GitHub生成Personal access tokens
步骤如下：
- 登录到GitHub，点击右上角头像，选择选项"Settings"
- 选择选项"Developer settings"
- 选择选项"Personal access tokens"
- 点击选项"Generate new token",如果有之前的用不了的就"Delete"删除掉
- 在"Note"选项中输入自定义的名字,权限选择里面勾上"repo"所有
- 点击选项"Generate token"
- 复制生成的token，忘了复制就重新来一遍

### Travis-CI配置
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

## Travis-CI初始配置问题
在第一次用Travis-CI配置自动构建的时候，"Trigger build"后各种问题，调试了好久第32次才通过。忘了作详细记录，目前记得的简单记录下。

### gitbook版本问题
在.travis.yml中，有安装gitbook版本，但是试了好几个版本在那一项都过不了，提示内容忘了截取，最后没有指定版本并且强制安装：
```
install:
  - "npm install -g gitbook"
  - "npm install -g gitbook-cli --force"
```
### JS版本问题
改了上面后发现JS版本也不过，高低版本都不行好像，最后改成了8：
```
node_js:
  - "8"
```
## Travis-CI更新异常
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

&#8195;&#8195;在GitHub里面`master-->View all branches-->`删掉所有由Travis创建的分支，travis-ci上的缓存也清理了，但是还是不行。后来把出错当天更新的文档删掉了，然后gitbook分支也删了，再重新运行就好了。当天可疑操作就是文件名后缀写重复了，当然不是这个原因；            
&#8195;&#8195;后来发现，那些SUMMARY.md提示无关紧要，`deploy.sh exited with 1`这个才是关键，检查脚本中有个条件语句判断上一条命令的退出状态码，执行失败脚本就exited with 1，也就是`gitbook build .`执行出现了问题，问题就在最新加的markdown中有html的代码，虽然我使用了代码注释但是估计gitbook还是识别有问题，GitHub识别没什么问题；    
&#8195;&#8195;最后弄明白了，代码块引用没什么问题，但是在说明描述中单反引号引用html相关代码时候不行，用对应的HTML ASCII码替代即可。

## 构建问题排查
官方常见构建问题说明：
- [Common Build Problems](https://docs.travis-ci.com/user/common-build-problems)
- [Build Config Validation](https://docs.travis-ci.com/user/build-config-validation)

### Branch not included per configuration
官方描述：
```
please make sure your branch is not explicitly excluded or not included in your .travis.yml file. 
```  
我遇到的报错：
```
Branch "main" not included per configuration
```
原因是在`.travis.yml`写错了：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
branches:
  only:
    - master
script:
  - travis_wait 10 bash start.sh
```
主分支应该是main，把master改成main即可。

### 10分钟超时
使用`arm64`和`ppc64le`构建我的ebook时候报错如下：
```
No output has been received in the last 10m0s, this potentially indicates a stalled build or something wrong with the build itself.

Check the details on how to adjust your build configuration on: https://docs.travis-ci.com/user/common-build-problems/#build-times-out-because-no-output-was-received

The build has been terminated
```
把`node.js`改成14版本后，不会超时，但是报错：
```
TypeError: cb.apply is not a function
    at /home/travis/.nvm/versions/node/v14.17.1/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (fs.js:193:5)
Installing GitBook 3.2.3
/home/travis/.nvm/versions/node/v14.17.1/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
```
参照方法进行处理报错没解决：[Gitbook-cli install error TypeError: cb.apply is not a function inside graceful-fs](https://stackoverflow.com/questions/64211386/gitbook-cli-install-error-typeerror-cb-apply-is-not-a-function-inside-graceful)   

把`node.js`改成10版本后，报错：
```
Error: shasum check failed for /tmp/npm-3441-2b05e04f/registry.npmjs.org/acorn/-/acorn-0.9.0.tgz
Error: Couldn't locate plugins "advanced-emoji, toggle-chapters, splitter, github, search-plus, anchor-navigation-ex-toc, editlink, copy-code-button, theme-comscore", Run 'gitbook install' to install plugins from registry.
/home/travis/.travis/functions: line 607:  3425 Terminated              travis_jigger "${!}" "${timeout}" "${cmd[@]}"
The command "travis_wait 100 bash deploy.sh" exited with 1.
```

网上有注释掉三行代码解决问题的：
- [Gitbook错误"cb.apply is not a function"的解决办法](https://zhuanlan.zhihu.com/p/367562636)
- [How I fixed a "cb.apply is not a function" error while using Gitbook](https://flaviocopes.com/cb-apply-not-a-function/?ref=aggregate.stitcher.io)

根据此方法，在`.travis.yml`和`deploy.sh`里面都试过了不行一样有报错，下面这个报错删掉前面bash即可：
```
$ bash sed -i 's/^fs.*stat = statFix(fs.*stat)/\/\/ &/g' /home/travis/.nvm/versions/node/v14.17.1/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js

/bin/sed: /bin/sed: cannot execute binary file
```
后来在`deploy.sh`里面cat了这个文件：
```
cat /home/travis/.nvm/versions/node/v14.17.1/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js
```
多此一举从行首开始匹配，去掉`^`符号，继续构建报错：
```
$ npm install -g gitbook
2.31s$ npm install -g gitbook-cli --force
npm WARN using --force I sure hope you know what you are doing.
npm ERR! cb() never called!
npm ERR! This is an error with npm itself. Please report this error at:
npm ERR!     <https://npm.community>
npm ERR! A complete log of this run can be found in:
npm ERR!     /home/travis/.npm/_logs/2021-06-22T16_31_00_790Z-debug.log
The command "npm install -g gitbook-cli --force" failed and exited with 1 during .
```
改成了12版本，接近成功但是还是有报错：
```
*** Please tell me who you are.
Run
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
to set your account's default identity.

Omit --global to set the identity only in this repository.
fatal: empty ident name (for <travis@travis-job-bond-huang-ebook-517957310.(none)>) not allowed
error: src refspec gh-pages does not match any.
error: failed to push some refs to 'https://[secure]@github.com/bond-huang/ebook.git'
/home/travis/.travis/functions: line 607:  3416 Terminated              travis_jigger "${!}" "${timeout}" "${cmd[@]}"
The command "travis_wait 100 bash deploy.sh" exited with 1.
cache.2
store build cache
```
根据提示要设置用户信息，将`deploy.sh`中对应代码修改成：
```sh
git config --global user.name "huang"
git config --global user.email "huang19891023@163.com"
```
不设置成全局的也可以在创建`gh-pages`分支前设置，修改后继续报错：
```
Error: Couldn't locate plugins "advanced-emoji, toggle-chapters, splitter, github, search-plus, anchor-navigation-ex-toc, editlink, copy-code-button, theme-comscore", Run 'gitbook install' to install plugins from registry.
/home/travis/.travis/functions: line 607:  3423 Terminated              travis_jigger "${!}" "${timeout}" "${cmd[@]}"
The command "travis_wait 100 bash deploy.sh" exited with 1.
cache.2
store build cache
```
在`.travis.yml`加入如下代码：
```js
before_install:
  - "npm cache clean --force"
```
然后构建，超时了几次，多尝试了几次构建成功了。

其它参考链接：
- [Gitbook build stopped to work in node 12.18.3 #110](https://github.com/GitbookIO/gitbook-cli/issues/110)
- [TypeError: cb.apply is not a function](https://www.mmbyte.com/article/152924.html)
- [node v12.18.3 doesn't work with npm v6.9.2 and below #34491](https://github.com/nodejs/node/issues/34491)
- [All my react-native projects shows error TypeError: cb.apply is not a function](https://stackoverflow.com/questions/63054545/all-my-react-native-projects-shows-error-typeerror-cb-apply-is-not-a-function/63054816#63054816)

## 迁移问题
&#8195;&#8195;2021年6约15号之后，`travis-ci.org`将停止服务，不能进行构建了，转向`travis-ci.com`，需要把仓库迁移到`travis-ci.com`，迁移比较简单，`travis-ci.org`上发起，邮箱确认后在`travis-ci.com`点击迁移即可。
### error while trying to fetch the log
#### 问题描述
迁移完成后，在travis-ci.com上构建，提示或报错如下：
```
Oh no! You tried to trigger a build for bond-huang/ebook but the request was rejected. 
There was an error while trying to fetch the log. 
```
&#8195;&#8195;原因是`travis-ci.com`上策略更改了一下，在主页中，有个`Plan`选项，第一个是`Free Plan`,其它都是付费的，当然我选择了`Free`，有10000积分，构建了一次花掉了60积分，看来以后得省着点用了。

#### 解决方案
&#8195;&#8195;合作伙伴队列解决方案是由Travis-CI的合作伙伴赞助的基础架构解决方案，可以完全免费使用（仅适用于开源软件仓库）。目前有：
- IBM CPU在IBM Cloud中构建（由 IBM 赞助）
- ARM64 CPU构建在Equinix Metal（前 Packet）基础架构中（由ARM赞助）

要使用Partner Queue Solution运行作业，在公共仓库中`.travis.yml`使用以下标签：
```
os: linux
arch:
  - arm64
  - ppc64le
  - s390x
```
尝试使用了IBM Power平台ppc64le构建了我的`navigator`,用s390x构建了我的`ebook`没有扣积分。
#### 参考链接
参考链接：
- [Oh no! You tried to trigger a build for orgName/project but the request was rejected](https://travis-ci.community/t/oh-no-you-tried-to-trigger-a-build-for-orgname-project-but-the-request-was-rejected/10657)
- [Travis-CI Billing Overview](https://docs.travis-ci.com/user/billing-overview/#usage---credits)
- [Building on Multiple CPU Architectures](https://docs.travis-ci.com/user/multi-cpu-architectures/)

## 用户密码问题
构建时候报错：
```
remote: Invalid username or password.
fatal: Authentication failed for 'https://[secure]@github.com/bond-huang/ebook.git/'
```
一般都是GitHub上的Personal access tokens没了，重新创建一个即可。

## 构建超时
默认构建超过10分钟没有输出，就会超时报错：
```
No output has been received in the last 10m0s, this potentially indicates a stalled build or something wrong with the build itself.

Check the details on how to adjust your build configuration on: https://docs.travis-ci.com/user/common-build-problems/#build-times-out-because-no-output-was-received

The build has been terminated
```
如果有超过10分钟不产生输出的命令，可以在它前面加上前缀，这travis_wait是构建环境导出的函数。例如：
```yaml
install: travis_wait mvn install
```
继续上面的示例，将等待时间延长至30分钟：
```yaml
install: travis_wait 30 mvn install
```
官方参考链接：[Common Build Problems-build-times-out-because-no-output-was-received](https://docs.travis-ci.com/user/common-build-problems/#build-times-out-because-no-output-was-received)

## 待补充